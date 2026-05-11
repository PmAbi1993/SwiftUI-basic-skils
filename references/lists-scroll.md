# Lists, Tables, and Scroll Views

Repeated content is where SwiftUI identity and dependency mistakes become expensive. Stable identity, narrow row inputs, and careful transforms matter.

## Stable Identity

Prefer `Identifiable` data:

```swift
struct Task: Identifiable {
    let id: UUID
    var title: String
    var isComplete: Bool
}

ForEach(tasks) { task in
    TaskRow(task: task)
}
```

Never use indices for dynamic content:

```swift
ForEach(tasks.indices, id: \.self) { index in
    TaskRow(task: tasks[index])
}
```

Index identity breaks when items move, insert, or delete.

## Truly Unique IDs

IDs must be unique and stable. Avoid using:

- Display names.
- Array offsets.
- Dates that can collide.
- `\.self` for mutable values.
- Generated UUIDs inside `body`.

If the data lacks a stable ID, add one at the model boundary.

## Enumerated Content

For numbered rows, use `enumerated()` directly:

```swift
ForEach(items.enumerated(), id: \.element.id) { offset, item in
    NumberedRow(number: offset + 1, item: item)
}
```

Do not convert to `Array(items.enumerated())` just to satisfy `ForEach`.

## Filtering and Sorting

Avoid repeated inline transforms:

```swift
ForEach(items.filter(\.isVisible)) { item in
    ItemRow(item: item)
}
```

Prefer a named derived value:

```swift
var visibleItems: [Item] {
    items.filter(\.isVisible)
}
```

For large or expensive transforms, move work into a model or update a cached value with explicit invalidation.

## List

Use `List` for standard iOS rows, selection, swipe actions, edit mode, sectioning, and platform-consistent behavior:

```swift
List {
    Section("Favorites") {
        ForEach(favorites) { item in
            FavoriteRow(item: item)
        }
    }
}
```

Use `.refreshable` for pull-to-refresh:

```swift
List(items) { item in
    ItemRow(item: item)
}
.refreshable {
    await model.reload()
}
```

## Empty States

Use `ContentUnavailableView`:

```swift
ContentUnavailableView("No Results", systemImage: "magnifyingglass")
```

For search screens, use the built-in search unavailable view when it fits the project.

## Custom List Backgrounds

Use modern scroll background controls:

```swift
List(items) { item in
    ItemRow(item: item)
}
.scrollContentBackground(.hidden)
.background(.background)
```

If a scroll view has an opaque, static, solid background, keeping `scrollContentBackground(.visible)` can be more efficient for scroll edge rendering.

## Table on iPad

Use `Table` when an iPad screen needs dense, multi-column data:

```swift
Table(books) {
    TableColumn("Title", value: \.title)
    TableColumn("Author", value: \.authorName)
}
```

For compact size classes, provide a row-style fallback if the table becomes too dense.

Selection:

```swift
@State private var selectedBook: Book.ID?

Table(books, selection: $selectedBook) {
    TableColumn("Title", value: \.title)
    TableColumn("Year") { book in
        Text(book.year, format: .number.grouping(.never))
    }
}
```

Keep table column values lightweight. Avoid expensive formatting closures in large datasets.

## ScrollView and Lazy Stacks

Use lazy stacks for large custom scroll layouts:

```swift
ScrollView {
    LazyVStack(spacing: 12) {
        ForEach(items) { item in
            ItemCard(item: item)
        }
    }
    .padding()
}
```

Do not use eager `VStack` or `HStack` for large repeated scroll content.

## Programmatic Scrolling

Use `ScrollViewReader` with stable IDs:

```swift
ScrollViewReader { proxy in
    List(messages) { message in
        MessageRow(message: message)
            .id(message.id)
    }
    .onChange(of: focusedMessageID) { _, id in
        if let id {
            proxy.scrollTo(id, anchor: .center)
        }
    }
}
```

Modern `scrollPosition` can be a better fit for state-driven scroll position in newer iOS targets.

## Scroll Effects

- Use `.scrollIndicators(.hidden)` to hide indicators.
- Use `scrollTargetBehavior` for paging or view-aligned scrolling.
- Use `scrollTransition` for item transitions tied to scroll position.
- Use `containerRelativeFrame()` for page-like layouts.
- Gate state writes from geometry or offset changes by thresholds.

## Row Dependency Rules

Rows should receive narrow inputs:

```swift
TaskRow(task: task, accent: theme.accent)
```

Avoid making every row read a large model when it only needs one value.

Avoid `AnyView` in rows. Use builders or concrete row types.

## Review Checklist

- [ ] Dynamic `ForEach` uses stable IDs.
- [ ] No index identity in mutable collections.
- [ ] IDs are unique and stable.
- [ ] Large custom scroll content uses lazy stacks.
- [ ] Filtering and sorting are not repeated inline in hot paths.
- [ ] Rows have narrow inputs.
- [ ] Empty states use system components where appropriate.
- [ ] Scroll state updates are throttled by meaning, not every tiny movement.

