# Lists and Scroll Views

List and scroll code should preserve identity, avoid repeated heavy transforms, and keep row dependencies narrow.

## ForEach Identity

Prefer `Identifiable` models:

```swift
struct Todo: Identifiable {
    let id: UUID
    var title: String
}

ForEach(todos) { todo in
    TodoRow(todo: todo)
}
```

Never use `.indices` for dynamic collections:

```swift
ForEach(todos.indices, id: \.self) { index in
    TodoRow(todo: todos[index])
}
```

Use stable IDs that do not change when content changes. `\.self` is fine only for immutable unique scalar values.

For enumerated content, do not convert to an array just for `ForEach`:

```swift
ForEach(items.enumerated(), id: \.element.id) { offset, item in
    Row(number: offset + 1, item: item)
}
```

## Filtering and Sorting

Avoid repeated inline transforms in `List` or `ForEach`:

```swift
List(filteredItems) { item in
    ItemRow(item: item)
}
```

Derive transformed data from the model or from a focused computed value. Cache only when invalidation is explicit.

## Lazy Containers

Use `LazyVStack` or `LazyHStack` for large scrollable custom layouts:

```swift
ScrollView {
    LazyVStack {
        ForEach(items) { item in
            ItemRow(item: item)
        }
    }
}
```

Do not replace `List` with custom scroll views unless the design needs custom behavior that `List` cannot provide.

## Empty States

Use `ContentUnavailableView` for empty data and search results:

```swift
ContentUnavailableView("No Items", systemImage: "tray")
```

For searchable content, use the system search empty state when it fits.

## Scroll APIs

- Use `.scrollIndicators(.hidden)` rather than old initializer parameters.
- Use `ScrollViewReader` with stable IDs for programmatic scrolling.
- Use `scrollPosition` for modern state-driven scroll position when targeting newer iOS versions.
- Gate frequent geometry-driven state updates by thresholds to avoid redraw storms.

## Row Rules

- Keep rows small and pure.
- Avoid `AnyView` in rows.
- Avoid broad environment model reads in every row when passing a specific value is enough.
- Keep the number of child views per row stable when practical.

