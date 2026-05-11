# View Composition

SwiftUI views should be small, direct descriptions of state. Keep view bodies light enough that the reader can understand structure without wading through business rules.

## File and Type Shape

- Prefer one substantial Swift type per file for production code.
- Keep tiny private helper types in the same file only when they are tightly scoped and unlikely to be reused.
- Place repeated styling in `ViewModifier`, custom `ButtonStyle`, or small reusable views.
- Use static member lookup for discoverable styles where it improves call sites.

## Body Hygiene

Avoid work in `body`:

- Networking
- Decoding
- Sorting or filtering large collections
- Creating formatters or heavyweight objects
- Writing state
- Starting tasks manually

Move action logic into methods:

```swift
Button("Save", systemImage: "checkmark", action: save)
```

## Extract Subviews

Prefer extracted `View` structs over large computed properties or long `@ViewBuilder` methods. Extract when:

- A section has its own state or actions.
- A repeated section has inputs worth naming.
- The body is hard to scan.
- A subsection redraws for different reasons than the parent.

Small private `some View` helpers are acceptable for brief structural readability, but they should not become mini screens.

## Preserve Identity

Prefer modifiers over conditional branches when toggling appearance:

```swift
Text(title)
    .foregroundStyle(isSelected ? .primary : .secondary)
```

Use conditionals when the actual presence of a view changes behavior or layout.

Avoid conditional modifier helpers that return different view types under the hood; they can create identity churn.

## Container Views

For generic containers, store built content rather than escaping closures when possible:

```swift
struct Card<Content: View>: View {
    @ViewBuilder let content: Content

    var body: some View {
        VStack(alignment: .leading) {
            content
        }
        .padding()
        .background(.thinMaterial)
        .clipShape(.rect(cornerRadius: 12))
    }
}
```

## UIKit Bridges

Use `UIViewRepresentable` only when SwiftUI lacks a native option or the project has an existing UIKit component. Keep bridge types small, isolate delegate work in a coordinator, and expose a SwiftUI-shaped API to the rest of the app.

