# Performance

Performance work starts with reducing unnecessary invalidation, expensive body work, and unstable identity. Apply advanced optimization only when there is a real bottleneck or clear risk.

## Reduce Redraw Fan-Out

- Pass only the values a child needs.
- Avoid large configuration or app models in frequently repeated rows.
- Avoid frequently changing values in environment state.
- Prefer granular observable models when one changed item should not redraw a whole list.

```swift
// Better: narrow dependency
ItemRow(item: item, tint: theme.primaryTint)
```

## Guard State Updates

SwiftUI does not skip a state assignment just because the value is equal. Check hot-path updates:

```swift
.onChange(of: offset) { _, newOffset in
    let shouldShow = newOffset < -32
    if shouldShow != showsTitle {
        showsTitle = shouldShow
    }
}
```

This matters in scroll, gesture, animation, timer, and geometry handlers.

## Keep Body Cheap

Avoid in `body`:

- Sorting and filtering large collections
- Date or number formatter creation
- Image decoding
- Network calls
- Object creation that should persist
- Synchronous file reads

Use `.task` for async loading tied to view lifetime:

```swift
.task(id: userID) {
    profile = await service.profile(for: userID)
}
```

## Lists and Rows

- Use stable `ForEach` identity.
- Use lazy stacks for large custom scroll layouts.
- Avoid `AnyView` in rows.
- Avoid inline filters in `ForEach`.
- Keep row inputs narrow and row bodies simple.

## Expensive Views

Use `Equatable` views only when the body is expensive and equality is correct:

```swift
struct SummaryView: View, Equatable {
    let summary: Summary

    static func == (lhs: Self, rhs: Self) -> Bool {
        lhs.summary.id == rhs.summary.id
    }

    var body: some View {
        ComplexSummary(summary: summary)
    }
}
```

Remember to update equality when inputs change.

## Debugging Redraws

During local debugging, `Self._printChanges()` or `Self._logChanges()` can show which properties are invalidating a view. Remove or guard debugging calls before shipping.

## Common Anti-Patterns

- Creating `DateFormatter` or `NumberFormatter` in `body`.
- Using `onAppear` for async work that should cancel automatically.
- Storing derived state without invalidation rules.
- Passing a full app model into every row.
- Triggering state writes on every pixel of scrolling.

