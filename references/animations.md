# Animations

Animation should be state-driven, scoped, and cheap. Prefer transforms over layout changes when animating frequently.

## Implicit Animations

Always use the value-scoped API:

```swift
Circle()
    .scaleEffect(isSelected ? 1.2 : 1)
    .animation(.bouncy, value: isSelected)
```

Never use broad `.animation(_:)`; it can animate unrelated changes.

## Explicit Animations

Use `withAnimation` for event-driven state changes:

```swift
Button("Expand") {
    withAnimation(.snappy) {
        isExpanded.toggle()
    }
}
```

For chained animations, use completion instead of delayed nested tasks:

```swift
withAnimation {
    scale = 2
} completion: {
    withAnimation {
        scale = 1
    }
}
```

## Transitions

Transitions animate insertion and removal. The state change that inserts or removes the view must happen inside an animation context:

```swift
if isShowing {
    DetailPanel()
        .transition(.move(edge: .bottom).combined(with: .opacity))
}
```

```swift
withAnimation {
    isShowing.toggle()
}
```

## Performance

- Prefer `opacity`, `scaleEffect`, `offset`, and `rotationEffect` over animating layout-heavy frame changes.
- Avoid doing work inside animation state changes.
- Keep animated views narrow; broad parent invalidation can make animations stutter.

## Phase and Keyframe Animation

Use `phaseAnimator` for simple multi-step state sequences and `keyframeAnimator` for precise timing.

Use enum phases for clarity:

```swift
enum PulsePhase: CaseIterable {
    case small
    case large
}
```

## Custom Animatable Values

For iOS 26+, prefer `@Animatable` over manual `animatableData` when all animated properties are supported:

```swift
@Animatable
struct Wedge: Shape {
    var startAngle: Angle
    var endAngle: Angle
    @AnimatableIgnored var clockwise: Bool

    func path(in rect: CGRect) -> Path {
        Path()
    }
}
```

Use manual `Animatable` conformance only when the macro cannot express the behavior clearly.

