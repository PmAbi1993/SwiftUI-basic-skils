# SwiftUI MVVM

MVVM is useful in SwiftUI when it clarifies feature behavior, isolates testable logic, and keeps views declarative. It is not useful when every simple view gets an empty view model just to satisfy a pattern.

## Practical Definition

| Layer | Owns |
|---|---|
| Model | Domain data, persistence entities, value types, service DTOs, validation rules that are independent of UI |
| View | SwiftUI layout, bindings to UI controls, presentation modifiers, local visual state |
| ViewModel | Screen or feature behavior: loading, user actions, validation flow, derived display state, service coordination |

In modern SwiftUI, a view model is usually an `@Observable` type, not an `ObservableObject`.

## When to Use a ViewModel

Use a view model when a screen has:

- async loading or saving
- non-trivial validation
- multiple user actions that mutate state
- derived display state from several inputs
- dependency injection needs
- navigation/presentation decisions tied to business rules
- unit-test-worthy behavior

Do not create a view model for a static row, a tiny styling wrapper, or a view that only displays a value.

## Recommended Shape

```swift
@Observable
@MainActor
final class ProfileEditorViewModel {
    var name = ""
    var isSaving = false
    var errorMessage: String?

    private let service: ProfileServicing

    init(service: ProfileServicing) {
        self.service = service
    }

    var canSave: Bool {
        !name.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty && !isSaving
    }

    func load(userID: User.ID) async {
        do {
            let profile = try await service.profile(for: userID)
            name = profile.name
        } catch {
            errorMessage = "Could not load profile."
        }
    }

    func save(userID: User.ID) async {
        guard canSave else { return }
        isSaving = true
        defer { isSaving = false }

        do {
            try await service.updateProfile(userID: userID, name: name)
        } catch {
            errorMessage = "Could not save profile."
        }
    }
}
```

SwiftUI ownership:

```swift
struct ProfileEditorScreen: View {
    @State private var viewModel: ProfileEditorViewModel

    let userID: User.ID

    init(userID: User.ID, service: ProfileServicing) {
        self.userID = userID
        _viewModel = State(initialValue: ProfileEditorViewModel(service: service))
    }

    var body: some View {
        Form {
            TextField("Name", text: $viewModel.name)

            Button("Save") {
                Task {
                    await viewModel.save(userID: userID)
                }
            }
            .disabled(!viewModel.canSave)
        }
        .task(id: userID) {
            await viewModel.load(userID: userID)
        }
    }
}
```

Use `@State` to preserve an owned observable view model across redraws.

## View Responsibilities

Views should:

- declare layout and modifiers
- own small local visual state, such as expanded/collapsed UI
- bind controls to model/view model state
- call view model methods from user actions
- attach `.task`, `.sheet`, `.alert`, navigation, and toolbar modifiers
- keep presentation state local when it is purely UI

Views should not:

- fetch network or database data inline
- contain large validation branches
- build request payloads in button closures
- sort/filter large collections repeatedly in `body`
- know service implementations

## ViewModel Responsibilities

View models should:

- expose state the view needs
- expose computed display state
- coordinate use cases/services/repositories
- perform validation and state transitions
- convert thrown errors into user-facing state according to project conventions
- remain easy to unit test

View models should not:

- import SwiftUI unless they truly need SwiftUI types
- create SwiftUI views
- know view layout
- hold `Binding` properties
- become a dumping ground for unrelated feature logic
- store parent-provided values as private copies unless they are editing drafts

## Local View State vs ViewModel State

Keep state in the view when it is visual and disposable:

- sheet visibility
- current text field focus
- a local disclosure toggle
- selected tab inside a small local component
- animation triggers

Move state to the view model when it affects feature behavior:

- current load/save state
- validation result
- selected entity for a business workflow
- server response state
- filter/sort options that affect data loading
- error state that must be tested

## Inputs and Outputs

Prefer method calls over exposing implementation details:

```swift
Button("Archive") {
    Task {
        await viewModel.archive()
    }
}
```

For parent communication, prefer closures at the screen boundary:

```swift
struct ProfileEditorScreen: View {
    let onSaved: () -> Void
}
```

The view model can return an outcome:

```swift
enum SaveResult {
    case saved
    case failed(String)
}

func save() async -> SaveResult {
    // ...
}
```

This keeps navigation and presentation in the view while preserving testable business behavior.

## Dependency Injection

Inject dependencies through initializers:

```swift
protocol ProfileServicing {
    func profile(for id: User.ID) async throws -> Profile
    func updateProfile(userID: User.ID, name: String) async throws
}
```

Avoid singletons for feature logic unless the project already standardizes on them.

For previews and tests, pass fakes:

```swift
ProfileEditorScreen(
    userID: .preview,
    service: PreviewProfileService()
)
```

## Derived Display State

Use computed properties for cheap derived state:

```swift
var saveButtonTitle: String {
    isSaving ? "Saving..." : "Save"
}

var trimmedName: String {
    name.trimmingCharacters(in: .whitespacesAndNewlines)
}
```

Do not store derived values as mutable state unless they are expensive and explicitly invalidated.

## Async Actions

Use structured async methods on the view model:

```swift
func refresh() async {
    guard !isLoading else { return }
    isLoading = true
    defer { isLoading = false }

    do {
        items = try await service.items()
    } catch {
        errorMessage = "Could not refresh."
    }
}
```

Call from the view with `.task` or a `Task` inside a user action:

```swift
.task {
    await viewModel.refresh()
}
```

Avoid starting untracked long-running tasks inside the view model unless cancellation ownership is explicit.

## Navigation and Presentation

Keep pure presentation mechanics in the view. Keep business decisions in the view model.

Good split:

```swift
if await viewModel.save() == .saved {
    dismiss()
}
```

Avoid making a view model directly call `dismiss()` or construct destination views.

For complex flows, expose route-like state:

```swift
enum ProfileRoute: Hashable {
    case avatarPicker
    case deleteConfirmation
}

var route: ProfileRoute?
```

The view decides how that route maps to a sheet, dialog, or navigation destination.

## Editing Drafts

For edit screens, do not mutate persisted or parent-owned data until save unless live editing is intended.

Use a draft:

```swift
struct ProfileDraft: Equatable {
    var name: String
    var bio: String

    init(profile: Profile) {
        name = profile.name
        bio = profile.bio
    }
}
```

The view model edits the draft, validates it, then commits through a service/repository.

## Testing MVVM

View models should be testable without rendering SwiftUI:

- inject fake services
- call async methods directly
- assert state transitions
- test validation
- test error mapping
- test derived state

Example test shape:

```swift
@Test
func saveDisablesButtonWhileSaving() async {
    let service = SlowProfileService()
    let model = ProfileEditorViewModel(service: service)
    model.name = "Ava"

    await model.save(userID: .preview)

    #expect(model.errorMessage == nil)
}
```

## Anti-Patterns

- Creating a view model for every tiny view.
- Making the view model own SwiftUI layout state.
- Storing `Binding` inside a view model.
- Passing an entire app model into every view model.
- Letting view models create views.
- Hiding all actions behind one generic `send(_:)` method unless the project uses a reducer-style architecture.
- Copying parent inputs into view model state and then wondering why updates do not arrive.
- Performing database saves from inline button closures instead of a named method.

## Review Checklist

- [ ] The view model has a clear screen or feature responsibility.
- [ ] Owned observable view models are stored with `@State`.
- [ ] The view remains mostly layout and presentation.
- [ ] Business logic is testable without rendering SwiftUI.
- [ ] Dependencies are injected.
- [ ] Async methods guard duplicate work and expose loading/error state.
- [ ] Draft editing is used when persisted data should not mutate before save.
- [ ] Navigation and presentation mechanics stay in the view unless the project intentionally centralizes routing.

