# iOS SwiftUI API Modernization

Use this reference first when reviewing or updating SwiftUI code. Prefer the newest stable iOS API that matches the project deployment target.

## Always Prefer

| Older API or pattern | Preferred API or pattern |
|---|---|
| `navigationBarTitle(_:)` | `navigationTitle(_:)` |
| `navigationBarItems(...)` | `toolbar { ToolbarItem(...) }` |
| `navigationBarHidden(_:)` | `toolbarVisibility(.hidden, for: .navigationBar)` |
| `statusBar(hidden:)` | `statusBarHidden(_:)` |
| `edgesIgnoringSafeArea(_:)` | `ignoresSafeArea(_:edges:)` |
| `colorScheme(_:)` | `preferredColorScheme(_:)` |
| `foregroundColor(_:)` | `foregroundStyle(_:)` |
| `cornerRadius(_:)` | `clipShape(.rect(cornerRadius:))` |
| `actionSheet(...)` | `confirmationDialog(...)` |
| `alert(isPresented:content:)` | `alert(_:isPresented:actions:message:)` |
| `autocapitalization(_:)` | `textInputAutocapitalization(_:)` |
| `TextField` commit callbacks | `onSubmit(of:_:)` and `@FocusState` |
| `accentColor(_:)` | `tint(_:)` |
| `disableAutocorrection(_:)` | `autocorrectionDisabled(_:)` |
| `NavigationView` | `NavigationStack` or `NavigationSplitView` |
| `ObservableObject` for new shared state | `@Observable` |
| `onChange(of:perform:)` | `onChange(of:) { }` or `onChange(of:) { old, new in }` |
| `UIImpactFeedbackGenerator` in views | `sensoryFeedback(_:trigger:)` |
| `MagnificationGesture` | `MagnifyGesture` |
| `RotationGesture` | `RotateGesture` |
| `.coordinateSpace(name:)` | `.coordinateSpace(.named(...))` |
| `.tabItem { ... }` | `Tab("Title", systemImage:) { ... }` |
| `PreviewProvider` | `#Preview` |
| Manual `EnvironmentKey` boilerplate | `@Entry` on `EnvironmentValues` |

## iOS 18+ Tabs

Use `Tab` inside `TabView`:

```swift
TabView {
    Tab("Home", systemImage: "house") {
        HomeView()
    }

    Tab("Search", systemImage: "magnifyingglass") {
        SearchView()
    }
}
```

When using `TabView(selection:)`, prefer an enum selection:

```swift
enum AppTab: Hashable {
    case home
    case search
}

@State private var selectedTab: AppTab = .home

TabView(selection: $selectedTab) {
    Tab("Home", systemImage: "house", value: .home) {
        HomeView()
    }

    Tab("Search", systemImage: "magnifyingglass", value: .search) {
        SearchView()
    }
}
```

## iOS 26+ APIs

- Use `glassEffect(_:in:)`, `GlassEffectContainer`, and glass button styles for Liquid Glass when requested by the UI direction.
- Use `tabBarMinimizeBehavior(_:)` for scroll-aware tab bars.
- Use `tabViewBottomAccessory` for persistent controls above the tab bar.
- Use `Tab(role: .search)` for a dedicated search tab.
- Use `ToolbarSpacer` to group toolbar items.
- Use `searchToolbarBehavior(.minimizable)` when search should collapse in compact spaces.
- Use `@Animatable` instead of hand-writing `animatableData` when the shape fits the macro.
- Use `TextEditor(text:)` with `AttributedString` for rich text editing.
- Use SwiftUI `WebView` for web content; import `WebKit` when needed.

Gate these APIs when the project supports lower iOS versions:

```swift
if #available(iOS 26, *) {
    content.glassEffect()
} else {
    content.background(.ultraThinMaterial)
}
```

## Common Review Flags

- Deprecated modifiers remain in new code.
- UIKit haptic APIs are fired manually from SwiftUI views.
- `GeometryReader` is used where `containerRelativeFrame()`, `visualEffect()`, `onGeometryChange`, or a custom `Layout` would be simpler.
- Text is concatenated with `+`; prefer interpolation with styled `Text` values.
- Asset catalog images use string names when generated symbols are available.

