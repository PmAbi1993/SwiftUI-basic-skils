# Swift Hygiene

Use Swift 6.2-era language and concurrency defaults. Keep code safe, direct, and consistent with the project.

## Concurrency

- Prefer `async`/`await` APIs over callback APIs when both exist.
- Prefer structured concurrency over Grand Central Dispatch for new code.
- Use `.task` and `.task(id:)` for view-lifetime async work.
- Use `Task.sleep(for:)`, not `Task.sleep(nanoseconds:)`.
- Treat `Task.detached()` as suspicious; use it only with a clear isolation reason.
- Protect mutable shared state with actors or main-actor isolation.
- Check whether the project uses Main Actor default isolation before adding redundant `MainActor.run`.
- Watch `@Sendable` closures and captured mutable state under strict concurrency.

## Main Actor UI Models

UI-facing observable models should generally be main-actor isolated:

```swift
@Observable
@MainActor
final class SettingsModel {
    var isSaving = false
}
```

If the project has default main-actor isolation, avoid redundant annotations when the project style omits them.

## Error Handling

- Avoid force unwraps and force `try`.
- Use `guard let`, `if let`, nil coalescing, `try?`, or `do/catch`.
- If a crash is intentional, use `fatalError("Clear reason")`.
- Do not swallow user-action errors with only `print`.
- Surface actionable failures through project-standard UI or state.

Bad:

```swift
Button("Save") {
    Task {
        try? await model.save()
    }
}
```

Better:

```swift
Button("Save") {
    Task {
        await model.save()
    }
}
```

Let the model publish an error state or throw to a caller that presents the failure.

## Modern Swift Style

- Prefer `if let value { ... }` shorthand.
- Omit `return` in single-expression computed properties and functions when it improves clarity.
- Prefer `Date.now` over `Date()`.
- Prefer `Double` over `CGFloat` unless an API, optional, or `inout` case requires `CGFloat`.
- Prefer `count(where:)` over `filter { ... }.count`.
- Prefer static member lookup where it improves SwiftUI readability, such as `.borderedProminent`.

Example:

```swift
var tileColor: Color {
    if isCorrect {
        .green
    } else {
        .red
    }
}
```

## Foundation APIs

- Prefer `URL.documentsDirectory` over manual directory lookup for common app directories.
- Prefer `appending(path:)` for URL path components.
- Prefer modern string replacement when available: `replacing("a", with: "b")`.
- Prefer modern date parsing strategies, such as `Date(input, strategy: .iso8601)`.
- Avoid manual date format strings for user-facing text when `FormatStyle` works.
- If manual user-facing date formatting is unavoidable, use year symbol `y` rather than `yyyy`.

## Imports

When a SwiftUI file already imports SwiftUI, avoid adding UIKit just to use common platform image/color types that SwiftUI already exposes through the platform overlay. Add UIKit only when directly using UIKit APIs.

If legacy `ObservableObject` and Combine publishers are required, ensure `import Combine` is explicit where needed.

## Comparable and Sorting

If a type is repeatedly sorted with the same closure, centralize ordering:

```swift
struct Book: Comparable {
    static func < (lhs: Book, rhs: Book) -> Bool {
        lhs.title.localizedStandardCompare(rhs.title) == .orderedAscending
    }
}
```

Do this when one natural ordering exists. Do not force `Comparable` when a type has many equally valid sort modes.

## Secrets and Storage

- Do not commit API keys, tokens, or credentials.
- Do not store sensitive values in `@AppStorage`.
- Use the project’s secure storage layer for secrets.
- Avoid printing sensitive values in logs.

## Tests and Lint

- Unit-test core logic such as parsing, validation, sorting, filtering, and service behavior.
- UI tests are useful for flows that cannot be covered at the model level.
- If SwiftLint is configured, leave it with no warnings or errors.
- Do not add comments that restate obvious code; add comments for non-obvious decisions or constraints.

## Project Structure

- Prefer one substantial type per Swift file.
- Organize by feature when the project already does so.
- Avoid broad architecture changes during local fixes.
- Keep new helpers internal or private unless they are meant to be shared.
- Do not introduce third-party frameworks without asking.

## Review Checklist

- [ ] Async work uses modern concurrency.
- [ ] No unnecessary dispatch queues in new code.
- [ ] Shared mutable state has clear isolation.
- [ ] Errors from user actions are not silently swallowed.
- [ ] Force unwraps are justified or removed.
- [ ] Formatting and parsing use modern APIs.
- [ ] Secrets are not stored insecurely.
- [ ] Tests cover meaningful non-UI logic when present.

