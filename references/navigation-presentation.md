# Navigation and Presentation

Navigation should be type-safe, local enough to reason about, and consistent within a hierarchy.

## NavigationStack

Use `NavigationStack` for push navigation:

```swift
NavigationStack {
    List(items) { item in
        NavigationLink(value: item) {
            ItemRow(item: item)
        }
    }
    .navigationDestination(for: Item.self) { item in
        ItemDetailView(item: item)
    }
}
```

Prefer `NavigationLink(value:)` with `navigationDestination(for:)` over destination closures for model-driven navigation.

Do not mix destination-style `NavigationLink(destination:)` and value-based destinations in the same hierarchy.

Register one destination per data type in a hierarchy. Duplicate registrations make behavior hard to predict.

## iPad Layout

Use `NavigationSplitView` when the app benefits from sidebar/content/detail navigation on larger iPad sizes. Keep compact behavior in mind and test selection state carefully.

## Programmatic Paths

Use `NavigationPath` or a typed path when navigation needs to be driven from state:

```swift
@State private var path: [Route] = []

NavigationStack(path: $path) {
    ContentView()
        .navigationDestination(for: Route.self) { route in
            route.destination
        }
}
```

Keep route types small, hashable, and stable.

## Sheets

Use `.sheet(item:)` for optional model-driven presentations:

```swift
@State private var selectedItem: Item?

.sheet(item: $selectedItem, content: EditItemView.init)
```

Prefer sheet content that owns its save/cancel actions and calls `dismiss()` internally when appropriate.

Use enum-backed sheet state when one screen can present several sheet kinds.

## Alerts and Dialogs

- Use the modern `alert(_:isPresented:actions:message:)` API.
- Omit an alert's single no-op OK button; SwiftUI provides dismissal.
- Use `confirmationDialog` for action choices, especially destructive choices.
- Attach dialogs to the UI that triggers them so transitions originate from the right place.

## Popovers and Covers

Use `popover` for contextual, lightweight choices and `fullScreenCover` for immersive tasks that should block the underlying screen. Keep dismissal paths obvious and state-driven.

