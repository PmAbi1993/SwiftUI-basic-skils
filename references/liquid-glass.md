# Liquid Glass

Liquid Glass is an iOS 26+ design material and interaction system. Use it for bars, controls, floating surfaces, and source-based transitions where the app’s visual direction calls for it. Do not make every container glass.

## Availability

Guard Liquid Glass APIs when supporting earlier iOS versions:

```swift
if #available(iOS 26, *) {
    content
        .glassEffect(.regular, in: .rect(cornerRadius: 18))
} else {
    content
        .background(.ultraThinMaterial, in: .rect(cornerRadius: 18))
}
```

## Basic Effect

```swift
Text("Continue")
    .padding(.horizontal, 20)
    .padding(.vertical, 12)
    .glassEffect(.regular, in: .capsule)
```

Modifier order matters. Apply size, padding, and layout before glass so the glass shape matches the intended surface.

## Interactive Glass

Use interactive glass only for controls:

```swift
Button("Save", systemImage: "checkmark", action: save)
    .buttonStyle(.glassProminent)
```

For a custom control:

```swift
content
    .padding()
    .glassEffect(.regular.interactive(), in: .rect(cornerRadius: 16))
```

Do not use interactive style for passive decoration.

## GlassEffectContainer

Wrap related glass elements so SwiftUI can resolve them as a group:

```swift
GlassEffectContainer(spacing: 16) {
    HStack(spacing: 16) {
        Button("Add", systemImage: "plus", action: add)
        Button("Edit", systemImage: "pencil", action: edit)
        Button("Delete", systemImage: "trash", role: .destructive, action: delete)
    }
    .buttonStyle(.glass)
}
```

Use container spacing to match visual separation.

## Glass Button Styles

Use system button styles first:

```swift
Button("Done", systemImage: "checkmark", action: done)
    .buttonStyle(.glassProminent)

Button("More", systemImage: "ellipsis", action: showMore)
    .buttonStyle(.glass)
```

Use prominent glass for primary actions only. Too many prominent glass buttons make the hierarchy noisy.

## Toolbar Grouping

Use `ToolbarSpacer` for related toolbar actions:

```swift
.toolbar {
    ToolbarItem(placement: .topBarTrailing) {
        Button("Previous", systemImage: "chevron.up", action: previous)
    }

    ToolbarItem(placement: .topBarTrailing) {
        Button("Next", systemImage: "chevron.down", action: next)
    }

    ToolbarSpacer(.fixed)

    ToolbarItem(placement: .topBarTrailing) {
        Button("Settings", systemImage: "gear", action: openSettings)
    }
}
```

Use `sharedBackgroundVisibility(.hidden)` only when an item should visually separate from the toolbar group.

## Tab Bar Behavior

Use `tabBarMinimizeBehavior(_:)` when scrolling should minimize the tab bar:

```swift
TabView {
    Tab("Today", systemImage: "calendar") {
        TodayView()
    }
}
.tabBarMinimizeBehavior(.onScrollDown)
```

Use `tabViewBottomAccessory` for persistent controls:

```swift
TabView {
    AppTabs()
}
.tabViewBottomAccessory {
    MiniPlayer()
}
```

Read accessory placement from the environment when the accessory needs to adapt between expanded and compact positions.

## Search Tab

Use `Tab(role: .search)` for a dedicated search surface:

```swift
TabView {
    Tab("Library", systemImage: "books.vertical") {
        LibraryView()
    }

    Tab(role: .search) {
        SearchView()
    }
}
```

Do not mix old tab item syntax with new role-based tabs.

## Morphing Presentation

Use source and destination IDs for source-based transitions:

```swift
@Namespace private var namespace

Button("Add", systemImage: "plus") {
    showsAdd = true
}
.navigationTransitionSource(id: "add", namespace: namespace)
.sheet(isPresented: $showsAdd) {
    AddScreen()
        .navigationTransitionDestination(id: "add", namespace: namespace)
}
```

IDs must match and remain stable for the transition pair.

## Background Extension

Use `backgroundExtensionEffect()` when artwork behind a glass surface should visually extend beyond safe-area edges:

```swift
Image(.hero)
    .resizable()
    .scaledToFill()
    .backgroundExtensionEffect()
```

Use this for image-rich designs, not generic plain backgrounds.

## Complete Pattern: Floating Action Cluster

```swift
struct FloatingActions: View {
    let add: () -> Void
    let filter: () -> Void

    var body: some View {
        if #available(iOS 26, *) {
            GlassEffectContainer(spacing: 12) {
                HStack(spacing: 12) {
                    Button("Filter", systemImage: "line.3.horizontal.decrease", action: filter)
                    Button("Add", systemImage: "plus", action: add)
                        .buttonStyle(.glassProminent)
                }
                .buttonStyle(.glass)
            }
        } else {
            HStack(spacing: 12) {
                Button("Filter", systemImage: "line.3.horizontal.decrease", action: filter)
                Button("Add", systemImage: "plus", action: add)
                    .buttonStyle(.borderedProminent)
            }
            .padding(10)
            .background(.ultraThinMaterial, in: .capsule)
        }
    }
}
```

## Design Guidance

- Use glass where content behind it helps the material read.
- Keep glass surfaces few and purposeful.
- Prefer system glass styles before custom styling.
- Do not stack glass on glass.
- Avoid placing tiny low-contrast text on busy glass.
- Use fallbacks that preserve layout even if visual material changes.

## Review Checklist

- [ ] iOS 26 APIs are guarded when needed.
- [ ] Glass is applied after layout modifiers.
- [ ] Interactive glass is only used for interactive controls.
- [ ] Related glass controls are grouped in `GlassEffectContainer`.
- [ ] Prominent glass is reserved for primary actions.
- [ ] Fallbacks preserve size and hierarchy.

