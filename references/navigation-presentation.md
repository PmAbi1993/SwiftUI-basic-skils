# Navigation and Presentation

iOS navigation should be type-safe, state-driven where needed, and consistent within a hierarchy. Presentation state should describe what is shown, not just whether something is shown.

## NavigationStack

Use `NavigationStack` for push-style navigation:

```swift
NavigationStack {
    List(books) { book in
        NavigationLink(value: book) {
            BookRow(book: book)
        }
    }
    .navigationTitle("Books")
    .navigationDestination(for: Book.self) { book in
        BookDetailScreen(book: book)
    }
}
```

Prefer value-based navigation for model destinations. It scales better than destination closures and keeps routes centralized.

## Navigation Rules

- Do not use `NavigationView` in new code.
- Do not mix `NavigationLink(destination:)` with value-based destinations in the same navigation hierarchy.
- Register a destination type once per hierarchy.
- Keep route values stable and hashable.
- Avoid placing navigation registrations deep inside repeated rows.

## Programmatic Navigation

Use a typed path when routes are known:

```swift
enum AppRoute: Hashable {
    case book(Book.ID)
    case author(Author.ID)
}

@State private var path: [AppRoute] = []

NavigationStack(path: $path) {
    LibraryScreen(openBook: { id in
        path.append(.book(id))
    })
    .navigationDestination(for: AppRoute.self) { route in
        switch route {
        case .book(let id):
            BookDetailScreen(bookID: id)
        case .author(let id):
            AuthorScreen(authorID: id)
        }
    }
}
```

Use `NavigationPath` when routes are truly heterogeneous or plugin-like. Prefer typed paths for app-owned flows.

## NavigationSplitView for iPad

Use `NavigationSplitView` when an iPad layout benefits from sidebar/content/detail structure:

```swift
NavigationSplitView {
    Sidebar(selection: $selection)
} detail: {
    if let selection {
        DetailScreen(selection: selection)
    } else {
        ContentUnavailableView("Select an Item", systemImage: "sidebar.left")
    }
}
```

Keep compact behavior in mind. Selection state should still make sense when the split collapses.

## Sheet State

Use `.sheet(item:)` for optional data:

```swift
@State private var editedBook: Book?

.sheet(item: $editedBook, content: EditBookScreen.init)
```

This unwraps the item safely and removes the need for parallel `Bool` plus optional state.

## Enum-Based Sheets

Use enum sheet state when one screen presents multiple sheet types:

```swift
enum ActiveSheet: Identifiable {
    case edit(Book)
    case share(Book)

    var id: String {
        switch self {
        case .edit(let book): "edit-\(book.id)"
        case .share(let book): "share-\(book.id)"
        }
    }
}

@State private var activeSheet: ActiveSheet?

.sheet(item: $activeSheet) { sheet in
    switch sheet {
    case .edit(let book):
        EditBookScreen(book: book)
    case .share(let book):
        ShareBookScreen(book: book)
    }
}
```

Avoid several boolean sheet flags on the same view.

## Sheet Ownership

Sheet content should usually own its actions:

```swift
struct EditBookScreen: View {
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        Button("Done") {
            save()
            dismiss()
        }
    }
}
```

This keeps the presenting view focused on choosing what to present.

## Alerts

Use modern alert builders:

```swift
.alert("Delete Book?", isPresented: $showsDeleteAlert) {
    Button("Delete", role: .destructive, action: deleteBook)
    Button("Cancel", role: .cancel) { }
} message: {
    Text("This cannot be undone.")
}
```

If an alert has only a single no-op dismissal button, omit the button builder.

## Confirmation Dialogs

Use confirmation dialogs for action choices:

```swift
.confirmationDialog("Change Status", isPresented: $showsStatusDialog) {
    Button("Mark Finished", action: markFinished)
    Button("Archive", role: .destructive, action: archive)
}
```

Attach the dialog to the UI that triggers it. This matters for source-based presentation transitions and visual continuity.

## Popovers and Full-Screen Covers

- Use `popover` for contextual controls, small inspectors, and lightweight choices.
- Use `fullScreenCover` only for flows that should fully take over the screen.
- Keep dismissal state-driven and avoid hidden global presentation flags.

## Source-Based Presentation Transitions

For iOS 26+ presentation morphing, use source and destination IDs with a namespace:

```swift
@Namespace private var namespace

Button("Add", systemImage: "plus") {
    showsAdd = true
}
.navigationTransitionSource(id: "add", namespace: namespace)
.sheet(isPresented: $showsAdd) {
    AddItemScreen()
        .navigationTransitionDestination(id: "add", namespace: namespace)
}
```

Guard iOS 26-only transition APIs if supporting earlier versions.

## Presentation Review Checklist

- [ ] Navigation uses `NavigationStack` or iPad-appropriate split navigation.
- [ ] Value-based destinations are used for model navigation where practical.
- [ ] Route registrations are not duplicated.
- [ ] Optional model sheets use `.sheet(item:)`.
- [ ] Multiple sheet types use enum state rather than several booleans.
- [ ] Alerts and dialogs use modern builder APIs.
- [ ] Dismissal behavior is local and obvious.

