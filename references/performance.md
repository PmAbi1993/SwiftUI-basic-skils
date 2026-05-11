# Performance

SwiftUI performance is usually about invalidation scope, identity stability, and keeping repeated work out of view builders. Do not optimize blindly, but treat hot paths and repeated rows with care.

## Main Principles

- Narrow dependencies: pass what a view needs, not a whole app model.
- Preserve identity in repeated content.
- Keep `body` cheap and side-effect-free.
- Avoid redundant state writes.
- Prefer lazy containers for large custom scroll content.
- Treat environment values that change frequently as expensive in large trees.
- Use advanced diffing tools such as `Equatable` only when the view is actually expensive and equality is correct.

## Redundant State Updates

SwiftUI invalidates on assignment; it does not automatically skip equal values in your custom state flow.

Bad:

```swift
.onReceive(timer) { date in
    currentMinute = calendar.component(.minute, from: date)
}
```

Better:

```swift
.onReceive(timer) { date in
    let nextMinute = calendar.component(.minute, from: date)
    if currentMinute != nextMinute {
        currentMinute = nextMinute
    }
}
```

This matters in timers, scroll handlers, drag gestures, sensor updates, and geometry observation.

## Hot Path State Gates

Avoid updating state for every tiny scroll or geometry change:

```swift
.onGeometryChange(for: Bool.self) { proxy in
    proxy.frame(in: .scrollView).minY < -32
} action: { shouldShowTitle in
    if showsTitle != shouldShowTitle {
        showsTitle = shouldShowTitle
    }
}
```

Convert noisy continuous values into meaningful state transitions.

## Pass Narrow Inputs

Bad:

```swift
TaskRow(task: task, appModel: appModel)
```

Better:

```swift
TaskRow(
    task: task,
    tint: appModel.theme.tint,
    isEditing: appModel.isEditing
)
```

Even with Observation’s property tracking, narrow inputs make code easier to reason about and reduce accidental dependencies.

## Observable Granularity

If changing one list item causes every row to update, consider per-item observable state:

```swift
@Observable
final class TaskRowModel {
    var isFavorite = false
    var isExpanded = false
}

struct TaskRow: View {
    let task: Task
    let model: TaskRowModel

    var body: some View {
        HStack {
            Text(task.title)
            if model.isFavorite {
                Image(systemName: "star.fill")
            }
        }
    }
}
```

Do this when row state is independently mutable or the list is large. Do not create a row model factory for every simple list.

## Body Cost

Avoid in `body`:

- Sorting or filtering large arrays.
- Creating formatters.
- Decoding images or data.
- Constructing service objects.
- Synchronous file reads.
- Network calls.
- Saving data.
- Launching unmanaged tasks.

Prefer model-owned or explicitly derived values:

```swift
var filteredBooks: [Book] {
    guard !query.isEmpty else { return books }
    return books.filter { $0.title.localizedStandardContains(query) }
}
```

If filtering is expensive, cache with explicit invalidation rather than recalculating during every redraw.

## View Initializers

Keep view initializers simple. Avoid doing work in `init` that can be done in `.task`, in a model, or before constructing the view.

Bad:

```swift
init(items: [Item]) {
    self.sortedItems = items.sorted { $0.name < $1.name }
}
```

Better:

```swift
.task(id: items) {
    sortedItems = items.sorted { $0.name < $1.name }
}
```

Or move sorting into the model that owns the data.

## Async Work

Prefer `.task` and `.task(id:)` for async work tied to a view lifecycle:

```swift
.task(id: bookID) {
    await model.load(bookID: bookID)
}
```

Use `.onAppear` for synchronous view-appearance side effects only when `.task` is not the right semantic fit.

## Lazy Loading

Use lazy stacks for large custom scroll content:

```swift
ScrollView {
    LazyVStack {
        ForEach(items) { item in
            ItemRow(item: item)
        }
    }
}
```

`List` already virtualizes many standard row scenarios. Do not replace `List` with a custom scroll layout unless the design requires it.

## AnyView

Avoid `AnyView`, especially in repeated rows. Type erasure hides structure from SwiftUI and can increase diffing cost.

Prefer:

- `@ViewBuilder`
- Concrete extracted views
- `Group`
- Generic container types
- Enum switches inside builders

## Equatable Views

For expensive views with stable inputs, `Equatable` can help:

```swift
struct PriceChartSummary: View, Equatable {
    let points: [Point]
    let selectedRange: DateInterval

    static func == (lhs: Self, rhs: Self) -> Bool {
        lhs.points == rhs.points &&
        lhs.selectedRange == rhs.selectedRange
    }

    var body: some View {
        ChartSummary(points: points, range: selectedRange)
    }
}
```

Only do this when equality is cheaper than recomputing the body and when all important inputs are included.

## POD Wrapper Pattern

Views containing only simple stored values can be cheap to diff. In very hot paths, a small wrapper view can isolate expensive internal state:

```swift
struct ScoreTile: View {
    let score: Int

    var body: some View {
        ScoreTileInternal(score: score)
    }
}

private struct ScoreTileInternal: View {
    let score: Int
    @State private var isExpanded = false

    var body: some View {
        Text(score, format: .number)
    }
}
```

This is advanced and should be used sparingly.

## Environment Cost

Avoid environment state for frequently changing values such as scrolling, timers, or per-row status in large lists. Passing a concrete value into the subview is often cheaper and clearer.

## Closures That May Run Away From Main Actor State

Some SwiftUI closures may be evaluated away from main-actor-isolated state for performance. Capture values instead of reading isolated state directly:

```swift
.visualEffect { [isHighlighted] content, geometry in
    content.opacity(isHighlighted ? 1 : 0.7)
}
```

Watch for this in:

- `Shape.path(in:)`
- `visualEffect`
- custom `Layout` methods
- `onGeometryChange` transform closures

## Scroll Rendering

If a scroll view has a static, opaque, solid background, prefer leaving content background visible when it helps scroll-edge rendering efficiency. Use hidden scroll content backgrounds when the design needs a custom background.

## Debugging Redraws

During local development, use `Self._printChanges()` or `Self._logChanges()` inside a view body guarded for debug builds to understand invalidation. Remove or guard these calls before shipping.

```swift
#if DEBUG
let _ = Self._logChanges()
#endif
```

## Review Checklist

- [ ] Hot-path state writes check meaningful changes first.
- [ ] Large lists use stable identity and lazy behavior.
- [ ] Rows receive narrow inputs.
- [ ] Expensive transforms are outside repeated view builders.
- [ ] Async work uses `.task` where appropriate.
- [ ] No object creation in `body`.
- [ ] No unnecessary `AnyView`.
- [ ] Environment state is not used for high-frequency per-row changes.
- [ ] Advanced equality or wrapper optimizations are justified by cost.

