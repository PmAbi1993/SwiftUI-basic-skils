# View Composition

SwiftUI performance and maintainability are both strongly tied to view shape. Keep bodies small, preserve identity, and extract views when the extracted piece has a real responsibility.

## Composition Principles

- A view body should describe UI structure, not perform work.
- Extract large or state-sensitive sections into separate `View` structs.
- Prefer project-local conventions over imposing a new architecture.
- Keep business logic in methods, models, services, or focused helpers.
- Use native SwiftUI controls and modifiers before custom containers.
- Prefer one substantial Swift type per file in production code.

## Recommended View File Shape

A practical order for readability:

```swift
struct EditBookScreen: View {
    @Environment(\.dismiss) private var dismiss
    @State private var model = EditBookModel()

    let bookID: Book.ID

    var body: some View {
        Form {
            titleSection
            detailsSection
        }
        .navigationTitle("Edit Book")
        .toolbar {
            ToolbarItem(placement: .topBarTrailing) {
                Button("Save", action: save)
            }
        }
    }

    private func save() {
        model.save(bookID: bookID)
        dismiss()
    }
}
```

This is a readability preference, not a reason to churn existing code. Apply it when creating new files or cleaning a file that is already being refactored.

## Body Anti-Patterns

Flag these in body code:

- Sorting or filtering large collections repeatedly.
- Creating formatters, contexts, services, or model objects.
- Starting unmanaged tasks.
- Writing state during body evaluation.
- Complex branching that hides the screen structure.
- Long inline button actions.
- Inline save, validation, parsing, networking, or persistence logic.

Move command behavior into methods:

```swift
Button("Archive", systemImage: "archivebox", action: archive)
```

## Extract Subviews, Not Large Computed Views

Small computed `some View` properties are acceptable for local readability. They become a problem when they contain their own complex branching, state-driven behavior, or repeated heavy work.

Prefer:

```swift
struct BookRow: View {
    let book: Book
    let isSelected: Bool

    var body: some View {
        HStack {
            Text(book.title)
            Spacer()
            if isSelected {
                Image(systemName: "checkmark")
            }
        }
    }
}
```

Over a large `@ViewBuilder` method inside a parent view.

Why this matters:

- Extracted view inputs are explicit.
- SwiftUI can reason about invalidation more narrowly.
- The compiler has smaller expressions to type-check.
- The code becomes easier to preview, test through models, and review.

## When to Extract

Extract when any of these are true:

- The section has its own state or actions.
- The section is reused.
- The parent body is difficult to scan.
- The section depends on a narrower set of values than the parent.
- The compiler struggles to type-check the body.
- A row or card appears inside a repeated collection.

Do not create tiny wrapper views that only hide one line of obvious SwiftUI.

## Conditional Views and Identity

Prefer modifiers for simple appearance changes:

```swift
Text(book.title)
    .foregroundStyle(isSelected ? .primary : .secondary)
    .opacity(isEnabled ? 1 : 0.5)
```

Use conditional views when the element truly appears or disappears:

```swift
if model.isShowingUpgradePrompt {
    UpgradePrompt()
        .transition(.move(edge: .bottom).combined(with: .opacity))
}
```

Avoid generic conditional modifier helpers that return different underlying types and obscure identity changes.

## ViewBuilder Guidance

Use `@ViewBuilder` for small branching or generic content. Avoid using it as a way to keep a screen in one file after it has become large.

For generic containers, store built content instead of an escaping closure when possible:

```swift
struct Card<Content: View>: View {
    @ViewBuilder let content: Content

    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            content
        }
        .padding()
        .background(.background, in: .rect(cornerRadius: 16))
    }
}
```

Avoid:

```swift
struct Card<Content: View>: View {
    let content: () -> Content

    var body: some View {
        content()
    }
}
```

The closure form is sometimes needed, but the stored built-view form is simpler and avoids repeated closure calls.

## Container Patterns

Reusable container views should:

- Accept only the data and content they need.
- Avoid owning unrelated state.
- Keep styling consistent and discoverable.
- Avoid creating a nested card-inside-card layout unless the design truly calls for it.

For shared styles, consider a `ViewModifier`:

```swift
struct PanelStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(.background, in: .rect(cornerRadius: 16))
    }
}

extension View {
    func panelStyle() -> some View {
        modifier(PanelStyle())
    }
}
```

## Overlay, Background, and ZStack

Use `overlay` and `background` when decorating a primary view:

```swift
Avatar(image: image)
    .overlay(alignment: .bottomTrailing) {
        StatusDot(status: status)
    }
```

Use `ZStack` when multiple peers share the same space and no single view is the obvious base.

This distinction helps preserve layout intent and avoid accidental expansion.

## Compositing Before Clipping

If a group of overlapping elements should clip as one rendered unit, use `compositingGroup()` before clipping:

```swift
content
    .compositingGroup()
    .clipShape(.rect(cornerRadius: 16))
```

This is useful for shadows, opacity, and overlapping subviews that should be treated as a single visual layer.

## Splitting State-Driven Parts

When one part of a screen changes frequently, isolate it:

```swift
struct PlayerScreen: View {
    let track: Track
    @State private var playback = PlaybackModel()

    var body: some View {
        VStack {
            TrackArtwork(track: track)
            PlaybackControls(model: playback)
        }
    }
}
```

Do not make a static artwork view depend on a frequently changing playback model unless it needs that data.

## Skeleton and Loading States

Use `redacted(reason:)` for skeleton loading when the real layout can be reused:

```swift
ProfileSummary(profile: placeholderProfile)
    .redacted(reason: model.isLoading ? .placeholder : [])
```

Avoid building an entirely separate placeholder hierarchy if the final layout already communicates the loading shape.

## AnyView

Avoid `AnyView` in normal SwiftUI code, especially in lists and rows. Prefer:

- `@ViewBuilder`
- Generic view types
- `Group`
- Enum-driven branching in a builder

Use `AnyView` only at boundaries where type erasure is genuinely necessary.

## UIKit Bridges

Use `UIViewRepresentable` only when SwiftUI lacks a suitable control or the project already owns a UIKit component.

Bridge rules:

- Keep the bridge narrow.
- Expose SwiftUI-shaped inputs and bindings.
- Put delegate work in a coordinator.
- Avoid leaking UIKit concepts through the rest of the SwiftUI app.
- Prefer SwiftUI replacements when modern APIs exist, such as SwiftUI `WebView` for web content on iOS 26+.

## Type-Checking Problems

When the compiler cannot type-check a body quickly:

- Extract subviews.
- Break large expressions into named values outside the builder.
- Simplify nested generics.
- Reduce chained conditionals.
- Move data transformation out of the view builder.

This is usually a design signal, not just a compiler annoyance.

## Review Checklist

- [ ] Body is readable and mostly structural.
- [ ] Heavy work is outside body.
- [ ] Large sections are extracted into real views.
- [ ] Repeated rows/cards are separate views with narrow inputs.
- [ ] Button actions are methods when non-trivial.
- [ ] Appearance toggles use modifiers when identity should be preserved.
- [ ] `AnyView` is absent unless justified.
- [ ] UIKit bridges are narrow and necessary.

