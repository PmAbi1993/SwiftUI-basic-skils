# iOS SwiftUI API Modernization

Load this first for reviews and refactors. It combines current iOS SwiftUI API replacements, common LLM mistakes, and project-level migration judgment.

## Modernization Principles

- Prefer the newest stable API available for the project deployment target.
- For new iOS 26+ code, use current SwiftUI syntax directly.
- When supporting lower versions, wrap new APIs with `#available` and provide a sensible fallback.
- Do not rewrite entire screens just because one old API appears; make focused changes unless the surrounding pattern is fragile.
- Avoid UIKit substitutes when SwiftUI has a native API.

## Always Use These Replacements

| Older API or pattern | Preferred API or pattern | Notes |
|---|---|---|
| `navigationBarTitle(_:)` | `navigationTitle(_:)` | Direct replacement. |
| `navigationBarItems(...)` | `toolbar { ToolbarItem(...) }` | Use semantic placements. |
| `navigationBarHidden(_:)` | `toolbarVisibility(.hidden, for: .navigationBar)` | Modern toolbar control. |
| `statusBar(hidden:)` | `statusBarHidden(_:)` | Direct replacement. |
| `edgesIgnoringSafeArea(_:)` | `ignoresSafeArea(_:edges:)` | More precise naming. |
| `colorScheme(_:)` | `preferredColorScheme(_:)` | Describes intent. |
| `foregroundColor(_:)` | `foregroundStyle(_:)` | Supports hierarchical and material-aware styles. |
| `cornerRadius(_:)` | `clipShape(.rect(cornerRadius:))` | More explicit clipping. |
| `actionSheet(...)` | `confirmationDialog(...)` | Modern action choice UI. |
| `alert(isPresented:content:)` | `alert(_:isPresented:actions:message:)` | Modern alert builder. |
| `autocapitalization(_:)` | `textInputAutocapitalization(_:)` | `.never` replaces old `.none` usage. |
| `disableAutocorrection(_:)` | `autocorrectionDisabled(_:)` | Modern text input API. |
| `accentColor(_:)` | `tint(_:)` | Current tint API. |
| `TextField` commit callbacks | `onSubmit(of:_:)` | Pair with focus state where needed. |
| `NavigationView` | `NavigationStack` or `NavigationSplitView` | Use split view for iPad-style multi-column layouts. |
| Destination-style navigation for model routes | `NavigationLink(value:)` plus `navigationDestination(for:)` | Type-safe and scalable. |
| `ObservableObject` for new shared state | `@Observable` | Use legacy wrappers only for existing or integration code. |
| `onChange(of:perform:)` one-value closure | `onChange(of:) { }` or `{ old, new in }` | The single-parameter form is stale. |
| `UIImpactFeedbackGenerator` in SwiftUI views | `sensoryFeedback(_:trigger:)` | Declarative and state-driven. |
| `MagnificationGesture` | `MagnifyGesture` | Modern gesture value shape. |
| `RotationGesture` | `RotateGesture` | Modern gesture value shape. |
| `.coordinateSpace(name:)` | `.coordinateSpace(.named(...))` | Modern coordinate-space syntax. |
| `.tabItem { ... }` | `Tab("Title", systemImage:) { ... }` | Required for newer tab capabilities. |
| `PreviewProvider` | `#Preview` | Current preview syntax. |
| Manual custom environment key boilerplate | `@Entry` in `EnvironmentValues` | Also works for focus, transaction, and container values. |
| `showsIndicators: false` initializers | `.scrollIndicators(.hidden)` | Modern scroll indicator control. |
| `UIGraphicsImageRenderer` for SwiftUI views | `ImageRenderer` | Native SwiftUI rendering. |

## Toolbar Placement

Use current toolbar placements:

```swift
.toolbar {
    ToolbarItem(placement: .topBarLeading) {
        Button("Cancel", action: cancel)
    }

    ToolbarItem(placement: .topBarTrailing) {
        Button("Save", action: save)
    }
}
```

Avoid stale placements such as `.navigationBarLeading` and `.navigationBarTrailing`.

## Overlay and Shape Styling

Prefer closure-based overlay APIs:

```swift
Text("Draft")
    .padding(8)
    .overlay(alignment: .topTrailing) {
        Image(systemName: "pencil")
    }
```

For shapes, fill and stroke can be chained in modern iOS:

```swift
RoundedRectangle(cornerRadius: 16)
    .fill(.background)
    .stroke(.quaternary, lineWidth: 1)
```

Do not add an overlay solely to draw a stroke when the shape can stroke itself.

## Tabs

Use `Tab` values for iOS 18+ style tabs:

```swift
enum AppTab: Hashable {
    case today
    case library
    case settings
}

@State private var selectedTab: AppTab = .today

TabView(selection: $selectedTab) {
    Tab("Today", systemImage: "calendar", value: .today) {
        TodayView()
    }

    Tab("Library", systemImage: "books.vertical", value: .library) {
        LibraryView()
    }

    Tab("Settings", systemImage: "gear", value: .settings) {
        SettingsView()
    }
}
```

Avoid integer or string tab selections in new code. Enums make tab state self-documenting and harder to mismatch.

## iOS 26 SwiftUI APIs

Use these when the project target allows it or when guarded:

- `glassEffect(_:in:)`, `GlassEffectContainer`, `.buttonStyle(.glass)`, and `.buttonStyle(.glassProminent)` for Liquid Glass surfaces.
- `tabBarMinimizeBehavior(_:)` for tab bars that minimize during scroll.
- `tabViewBottomAccessory` for persistent controls above the tab bar.
- `Tab(role: .search)` for a dedicated search tab that can integrate with system search behavior.
- `ToolbarSpacer` to visually group toolbar actions.
- `sharedBackgroundVisibility(_:)` when a toolbar item should opt out of a shared glass background.
- `badge(_:)` on toolbar item content for indicators.
- `searchToolbarBehavior(.minimizable)` for compact search presentation.
- `navigationZoomTransition`, `navigationTransitionSource`, and `navigationTransitionDestination` for source-based presentation transitions.
- `controlSize(.extraLarge)` for large prominent actions.
- Concentric corner style for controls that need to match container corners.
- Slider tick marks and neutral values.
- `TextEditor` with an `AttributedString` binding for rich text.
- SwiftUI `WebView` with `WebPage` for richer web interaction.
- `dragContainer` and related drag session APIs for multi-item drag.
- `@Animatable` and `@AnimatableIgnored` for animatable custom shapes and values.

Example guarded glass:

```swift
if #available(iOS 26, *) {
    content
        .padding()
        .glassEffect(.regular, in: .rect(cornerRadius: 18))
} else {
    content
        .padding()
        .background(.ultraThinMaterial, in: .rect(cornerRadius: 18))
}
```

## GeometryReader Alternatives

`GeometryReader` is still useful for real measurement-driven layout, but flag it when a modern API better expresses the intent:

- Use `containerRelativeFrame()` for sizing relative to a container.
- Use `visualEffect { content, geometry in ... }` for position-based visual effects.
- Use `onGeometryChange(for:of:action:)` to observe measured values.
- Use the `Layout` protocol for reusable custom layout behavior.

Use capture lists in closures that may run away from main-actor state:

```swift
.visualEffect { [isHighlighted] content, geometry in
    content.opacity(isHighlighted ? 1 : 0.6)
}
```

## Text and Formatting API Flags

- Never use `String(format:)` for user-facing numeric display when `FormatStyle` works.
- Prefer `Text(value, format: ...)` for numbers, dates, percentages, and currency.
- Use `localizedStandardContains()` for user-entered search.
- Do not concatenate `Text` with `+`; use interpolation with styled `Text` values.
- Prefer generated string catalog and asset symbols when the project is configured for them.
- Use automatic grammar agreement where the copy benefits from inflection:

```swift
Text("^[\(people.count) person](inflect: true)")
```

## Review Checklist

- [ ] No stale navigation APIs.
- [ ] No stale presentation APIs.
- [ ] No stale tab syntax in new code.
- [ ] No one-parameter `onChange` closures.
- [ ] No UIKit haptic generators in SwiftUI views when `sensoryFeedback` works.
- [ ] No unnecessary broad `GeometryReader`.
- [ ] No C-style formatting for user-facing values.
- [ ] No string-based assets where generated symbols are available.
- [ ] iOS 26 APIs are guarded when the project supports lower versions.

