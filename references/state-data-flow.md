# State, Data Flow, and SwiftData

SwiftUI data flow should make ownership obvious. Most bugs in SwiftUI screens come from using the right-looking wrapper in the wrong ownership position.

## Ownership Decision Table

| Situation | Preferred choice |
|---|---|
| View owns simple mutable UI state | `@State private var value` |
| View owns an observable model | `@State private var model = Model()` |
| Child needs to mutate parent state | `@Binding var value` |
| Child only reads parent value | `let value` |
| Child reads parent value and reacts to changes | `var value` plus `.onChange()` |
| Child receives observable model and needs bindings | `@Bindable var model: Model` |
| Many descendants need shared model | inject observable model with `.environment(model)` and read with `@Environment(Model.self)` |
| Legacy owned object | `@StateObject private var model` |
| Legacy injected object | `@ObservedObject var model` |

## Core Rules

- New shared mutable state should use `@Observable`.
- `@Observable` UI models should be main-actor isolated unless the project uses Main Actor default isolation.
- Use `@State`, not `@StateObject`, to own an `@Observable` instance.
- View-owned `@State` and legacy `@StateObject` should be `private`.
- Do not declare parent-provided values as `@State` or `@StateObject`; they capture only the initial value.
- Use `@Binding` only when the child writes the value.
- Use `@Bindable` only when the child receives an observable model and needs bindings to properties.
- Avoid storing derived state unless you control invalidation.

## Observable Model Pattern

```swift
@Observable
@MainActor
final class AccountEditorModel {
    var displayName = ""
    var isSaving = false

    func save() async throws {
        isSaving = true
        defer { isSaving = false }
        // Save through service.
    }
}

struct AccountEditorScreen: View {
    @State private var model = AccountEditorModel()

    var body: some View {
        AccountEditorForm(model: model)
    }
}

struct AccountEditorForm: View {
    @Bindable var model: AccountEditorModel

    var body: some View {
        TextField("Display Name", text: $model.displayName)
    }
}
```

Why `@State` owns the model: SwiftUI recreates view values often. `@State` preserves the model instance across redraws.

## Passed Values Are Not State

Wrong:

```swift
struct ProfileHeader: View {
    @State var user: User

    var body: some View {
        Text(user.name)
    }
}
```

Correct:

```swift
struct ProfileHeader: View {
    let user: User

    var body: some View {
        Text(user.name)
    }
}
```

If the child edits the value, pass a binding. If it edits properties on an observable model, pass the model and use `@Bindable`.

## Bindings

Avoid manual `Binding(get:set:)` in view bodies when a normal binding plus change reaction works:

```swift
TextField("Username", text: $model.username)
    .onChange(of: model.username) {
        model.saveDraft()
    }
```

Manual bindings are acceptable at API boundaries, but they should not become a habit for basic form plumbing.

Numeric entry should bind to numeric values:

```swift
TextField("Score", value: $score, format: .number)
    .keyboardType(.numberPad)
```

The keyboard modifier alone does not make a text binding numeric.

## Let vs Var for Inputs

Use `let` for read-only display:

```swift
struct BookRow: View {
    let book: Book

    var body: some View {
        Text(book.title)
    }
}
```

Use `var` when the view reacts to parent-provided changes:

```swift
struct SearchResults: View {
    var query: String
    @State private var resultCount = 0

    var body: some View {
        ResultsList(query: query)
            .onChange(of: query) { _, newQuery in
                resultCount = calculateCount(for: newQuery)
            }
    }
}
```

## Environment Models

Use environment models for state intentionally shared across a feature or app subtree:

```swift
@Observable
@MainActor
final class SessionModel {
    var currentUser: User?
}

RootView()
    .environment(SessionModel())
```

Read it where needed:

```swift
struct AccountMenu: View {
    @Environment(SessionModel.self) private var session

    var body: some View {
        Text(session.currentUser?.name ?? "Guest")
    }
}
```

Avoid putting highly volatile values in environment state. Even when a specific key is not read everywhere, broad environment dependency checking can become expensive in large repeated trees.

## Property Wrappers Inside Observable Models

Property wrappers such as `@AppStorage`, `@SceneStorage`, and `@Query` transform storage. `@Observable` also transforms storage. If existing code uses a wrapper inside an observable model, isolate it with `@ObservationIgnored` and expose a plain observed property for UI state when needed.

Avoid using `@AppStorage` as the source of truth inside observable models. In particular, do not use it for secrets or sensitive user data.

## Legacy ObservableObject

Legacy wrappers are not automatically wrong. They are acceptable when:

- The project already uses Combine-based models.
- A dependency still exposes `ObservableObject`.
- Rewriting would expand the scope beyond the task.

Rules for legacy code:

- Use `@StateObject` only when the view creates and owns the object.
- Use `@ObservedObject` only when the object is injected.
- Never create an injected object inline with `@ObservedObject`.
- If a custom initializer creates `@StateObject`, pass the expression directly to `StateObject(wrappedValue:)` so creation stays deferred.
- If `ObservableObject` is required for Combine publishers, ensure `import Combine` exists where needed.

## Nested Models

Nested legacy `ObservableObject` changes are not automatically observed through the parent. Pass nested objects directly to child views or migrate to Observation when practical.

Observation handles nested observable objects more naturally, but broad dependencies still matter. Passing narrow values can be better than passing a whole model.

## SwiftData

Use `@Query` in views when live model updates should drive UI.

Use `ModelContext.fetchCount()` when only a count is needed and live updates are not required from that call alone. If the UI must refresh automatically, pair the count with another state or query that invalidates the view.

Prefer model types that conform to `Identifiable` over scattering `id:` key paths through view code.

## SwiftData With CloudKit

When a SwiftData model syncs through CloudKit:

- Do not use `@Attribute(.unique)`.
- Every model property needs a default value or optional type.
- Relationships need optional types.
- Validate migrations carefully because CloudKit-backed model changes have less room for casual schema churn.

## Data Flow Review Checklist

- [ ] Ownership is clear from wrappers.
- [ ] View-owned state is private.
- [ ] Parent-provided values are not stored as state.
- [ ] Bindings imply real mutation.
- [ ] Observable UI models are isolated to the main actor or covered by project default isolation.
- [ ] Manual bindings are limited to boundary cases.
- [ ] Derived state is not stale.
- [ ] Environment models are intentional and not overused in repeated rows.
- [ ] SwiftData live vs non-live data needs are explicit.

