# Swift Hygiene

Use modern Swift that compiles cleanly under Swift 6.2-era expectations. Keep changes aligned with the existing project style.

## Swift 6.2 Defaults

- Prefer strict concurrency-safe code.
- Check whether the project uses Main Actor default isolation before adding redundant `MainActor.run`.
- Protect mutable shared state with actors or main-actor isolation.
- Treat `Task.detached()` as suspicious; use it only with a clear reason.
- Prefer `async`/`await` APIs over closure-based APIs when both exist.
- Avoid Grand Central Dispatch for new async work; use structured concurrency.
- Use `Task.sleep(for:)`, not `Task.sleep(nanoseconds:)`.

## Swift Style That Affects Quality

- Avoid force unwraps and force `try` unless failure is impossible and documented.
- Prefer `guard let`, `if let`, nil coalescing, or `do/catch`.
- Use `fatalError("Clear reason")` instead of a mysterious forced crash when a crash is intentional.
- Prefer `if let value { ... }` shorthand.
- Omit `return` in single-expression functions where it improves clarity.
- Prefer `Double` over `CGFloat` unless an API, optional, or `inout` situation requires `CGFloat`.

## Foundation and Standard Library

- Use `URL.documentsDirectory` and `appending(path:)` for common file URLs.
- Use `replacing("a", with: "b")` instead of older Foundation string replacement when possible.
- Use `count(where:)` instead of `filter { ... }.count`.
- If a type is repeatedly sorted with the same closure, consider centralizing ordering with `Comparable`.
- Prefer modern parsing and formatting strategies over manual format strings.

## Project Hygiene

- Do not commit secrets such as API keys.
- Add short comments only where logic is not self-evident.
- Keep core business logic testable outside SwiftUI views.
- Unit tests should cover important model, service, parsing, and validation logic when the project has tests.
- If SwiftLint is configured, leave it clean.
- Avoid introducing broad architecture changes during focused fixes.

