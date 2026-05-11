# State, Data Flow, and SwiftData

Design data flow before view structure. The main question is ownership: who creates the value, who reads it, and who writes it?

## Property Wrapper Guide

| Wrapper | Use when |
|---|---|
| `@State` | The view owns simple local state or owns an `@Observable` instance. Mark it `private`. |
| `@Binding` | A child view must mutate parent-owned state. |
| `@Bindable` | A view receives an `@Observable` model and needs bindings to its properties. |
| `@Environment` | A value or observable model is intentionally shared across a subtree. |
| `let` | A child receives read-only data. |
| `var` | A child receives read-only data and reacts with `.onChange()`. |

Legacy wrappers:

| Wrapper | Use when |
|---|---|
| `@StateObject` | The view owns a legacy `ObservableObject`. Prefer `@Observable` for new code. |
| `@ObservedObject` | The view receives a legacy `ObservableObject`. |
| `@EnvironmentObject` | Existing legacy environment object usage. Prefer observable models for new code. |

## Observation Defaults

Prefer `@Observable` classes for shared mutable state:

```swift
@Observable
@MainActor
final class ProfileModel {
    var name = ""
    var isSaving = false
}

struct ProfileView: View {
    @State private var model = ProfileModel()

    var body: some View {
        EditProfileView(model: model)
    }
}

struct EditProfileView: View {
    @Bindable var model: ProfileModel

    var body: some View {
        TextField("Name", text: $model.name)
    }
}
```

If the project enables Main Actor default isolation, explicit `@MainActor` on UI models may be redundant. Otherwise, shared UI models should be main-actor isolated.

## State Rules

- View-owned state is `private`.
- Parent-provided values are never `@State`; use `let`, `var`, `@Binding`, or `@Bindable`.
- A child should not receive a binding just to read a value.
- Avoid creating observable objects inline in `body`.
- If a view owns an observable class, store it in `@State` so redraws preserve the instance.
- Do not store derived values in `@State` unless you also own explicit invalidation.

## Bindings

Avoid `Binding(get:set:)` inside view bodies when a simpler source binding plus `.onChange()` works:

```swift
TextField("Username", text: $model.username)
    .onChange(of: model.username) {
        model.saveDraft()
    }
```

For numeric entry, bind to the numeric value and use a format initializer:

```swift
TextField("Score", value: $score, format: .number)
    .keyboardType(.numberPad)
```

## Environment

Use observable environment models for app-wide state only when broad sharing is intended:

```swift
@Observable
@MainActor
final class SessionModel {
    var user: User?
}

RootView()
    .environment(SessionModel())
```

Avoid storing frequently changing values in the environment because broad readers add update cost.

## Property Wrappers Inside Observable Models

Avoid property wrappers such as `@AppStorage`, `@SceneStorage`, and `@Query` directly inside `@Observable` models. If legacy code requires one, mark it `@ObservationIgnored` and expose plain observed properties for UI updates.

Never store usernames, passwords, tokens, or other secrets in `@AppStorage`.

## SwiftData Notes

- Use `@Query` in views when live updates are needed.
- Use `ModelContext.fetchCount()` only when a non-live count is acceptable or another update source refreshes the UI.
- Prefer identifiable model data over ad hoc IDs in view code.
- For SwiftData with CloudKit:
  - Do not use `@Attribute(.unique)`.
  - Model properties need default values or optional types.
  - Relationships need optional types.

