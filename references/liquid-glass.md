# Liquid Glass

Use Liquid Glass for iOS 26+ design direction when it fits the app surface. Do not force it into every card or background.

## Basic Effect

```swift
Text("Continue")
    .padding(.horizontal, 20)
    .padding(.vertical, 12)
    .glassEffect(.regular, in: .capsule)
```

SwiftUI anchors glass to the view bounds, so apply padding before the effect when the glass should include the padded area.

## Interactive Elements

Use interactive glass only for tappable or focusable controls:

```swift
Button("Save", systemImage: "checkmark", action: save)
    .buttonStyle(.glassProminent)
```

For custom controls:

```swift
content
    .padding()
    .glassEffect(.regular.interactive(), in: .rect(cornerRadius: 16))
```

## Grouped Glass

Use `GlassEffectContainer` when multiple glass shapes should be resolved together:

```swift
GlassEffectContainer(spacing: 16) {
    HStack(spacing: 16) {
        Button("Add", systemImage: "plus", action: add)
        Button("Edit", systemImage: "pencil", action: edit)
    }
    .buttonStyle(.glass)
}
```

## Modifier Order

Apply layout and visual sizing before glass:

```swift
content
    .padding()
    .frame(maxWidth: .infinity)
    .glassEffect(.regular, in: .rect(cornerRadius: 20))
```

Avoid clipping after glass unless you intentionally want to clip the rendered effect.

## Fallbacks

When supporting lower iOS versions, provide a material fallback:

```swift
if #available(iOS 26, *) {
    content
        .padding()
        .glassEffect(.regular, in: .rect(cornerRadius: 16))
} else {
    content
        .padding()
        .background(.ultraThinMaterial, in: .rect(cornerRadius: 16))
}
```

## Best Practices

- Use glass for controls, bars, overlays, and floating surfaces.
- Keep backgrounds clear enough for glass to make visual sense.
- Avoid stacking many glass layers.
- Prefer system glass button styles before custom effects.
- Keep high-contrast action hierarchy; not every action should be prominent.

