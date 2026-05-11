# Layout and Visual Design

Build iOS SwiftUI layouts that adapt naturally, use system controls, and keep project design decisions consistent. Avoid custom layout work when a native component already solves the problem.

## Layout Defaults

- Avoid `UIScreen.main.bounds`.
- Prefer flexible frames over fixed dimensions.
- Prefer relative layout over hard-coded screen math.
- Use `containerRelativeFrame()` for container-based sizing.
- Use `visualEffect` for position-based visual effects.
- Use `onGeometryChange` when a measured value needs to drive state.
- Use `GeometryReader` only when layout genuinely depends on geometry.
- Use the `Layout` protocol for reusable custom layout algorithms.

## Flexible Frames

Fixed frames often fail when content grows or the device changes. Prefer constraints:

```swift
Text(title)
    .frame(maxWidth: .infinity, alignment: .leading)
```

For controls:

```swift
Button("Continue", action: continueFlow)
    .buttonStyle(.borderedProminent)
    .frame(maxWidth: .infinity)
```

## Own Your Container

A reusable child view should not assume it is full-screen. Let the parent decide placement:

```swift
struct ProductCard: View {
    let product: Product

    var body: some View {
        VStack(alignment: .leading) {
            ProductImage(product: product)
            Text(product.name)
        }
    }
}
```

The parent can choose grid, list, padding, and width.

## System Components

Prefer native components:

- `Label` for icon plus text.
- `ContentUnavailableView` for empty or missing content.
- `Form` for settings and data entry.
- `LabeledContent` for title/value rows.
- `Picker` for bounded choices.
- `Toggle` for booleans.
- `Slider` or `Stepper` for numeric adjustment.
- `Menu` for secondary option sets.
- `ControlGroup` for related compact controls.

Example:

```swift
LabeledContent("Storage") {
    Text(usedStorage, format: .byteCount(style: .file))
}
```

## Design Constants

When an app has repeated spacing, radii, animation timing, or colors, centralize them:

```swift
enum AppStyle {
    static let cornerRadius: CGFloat = 16
    static let rowSpacing: CGFloat = 12
    static let screenPadding: CGFloat = 20
}
```

Do not invent a design-token system for a tiny local fix, but avoid scattering magic numbers across a feature.

## Forms and Inputs

Use `Form` when the UI is settings-like or data-entry-heavy:

```swift
Form {
    Section("Profile") {
        TextField("Name", text: $model.name)
        Picker("Role", selection: $model.role) {
            ForEach(Role.allCases) { role in
                Text(role.title).tag(role)
            }
        }
    }
}
```

Use `LabeledContent` for controls that need aligned labels:

```swift
LabeledContent("Volume") {
    Slider(value: $volume)
}
```

## Multi-Line Text Entry

Use vertical `TextField` when placeholder behavior and lightweight editing matter:

```swift
TextField("Notes", text: $notes, axis: .vertical)
    .lineLimit(4...)
```

Use `TextEditor` for long-form or rich text editing.

## Typography

- Prefer semantic font styles such as `.body`, `.headline`, `.title3`, and `.caption`.
- Use `bold()` instead of `fontWeight(.bold)`.
- Use custom weights only when they support a clear design hierarchy.
- Avoid tiny text unless the surrounding design makes it readable.
- Prefer `foregroundStyle(.secondary)` over manual opacity for secondary text.

## Shapes, Clipping, and Backgrounds

Use modern shape APIs:

```swift
content
    .padding()
    .background(.background, in: .rect(cornerRadius: 16))
```

Clip explicitly:

```swift
image
    .scaledToFill()
    .clipShape(.rect(cornerRadius: 12))
```

When using `RoundedRectangle`, the default corner style is already modern. Specify style only when a specific visual result is required.

## Touch Targets

Interactive controls should be comfortably tappable. Do not make custom buttons visually tiny unless the hit area is expanded:

```swift
Button(action: toggleFavorite) {
    Image(systemName: "heart")
        .frame(width: 44, height: 44)
}
```

## Layout Performance

Avoid:

- Deeply nested stacks when a simpler grid or layout would work.
- Preference-key chains that update constantly.
- Geometry-driven state updates on every pixel of movement.
- Rebuilding large sections when only a small label changes.

Gate layout state changes:

```swift
.onGeometryChange(for: Bool.self) { proxy in
    proxy.frame(in: .scrollView).minY < -40
} action: { shouldCollapse in
    if isCollapsed != shouldCollapse {
        isCollapsed = shouldCollapse
    }
}
```

## Review Checklist

- [ ] No screen-size assumptions.
- [ ] Fixed frames are justified.
- [ ] Native controls are used where they fit.
- [ ] Repeated design values are centralized when useful.
- [ ] Layout measurement is narrow and purposeful.
- [ ] Text hierarchy uses semantic styles.
- [ ] Custom controls have comfortable hit areas.

