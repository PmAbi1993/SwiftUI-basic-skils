# Animations

SwiftUI animations should be state-driven, scoped to the value that changes, and designed to avoid unnecessary layout churn.

## Core Concepts

- Property animations animate changes to modifiers such as opacity, scale, offset, rotation, and color.
- Transitions animate insertion and removal.
- Explicit animations wrap state changes.
- Implicit animations attach to view hierarchy and watch specific values.
- Later animation modifiers closer to the changing view can override outer animation choices.

## Implicit Animations

Always include a value:

```swift
Circle()
    .scaleEffect(isSelected ? 1.2 : 1)
    .animation(.bouncy, value: isSelected)
```

Avoid:

```swift
Circle()
    .scaleEffect(isSelected ? 1.2 : 1)
    .animation(.bouncy)
```

The broad form can animate unrelated changes and is stale.

## Explicit Animations

Use `withAnimation` for event-driven changes:

```swift
Button("Expand") {
    withAnimation(.snappy) {
        isExpanded.toggle()
    }
}
```

Use explicit animations for button taps, drag completion, reorder changes, and route-like UI state changes.

## Chained Animations

Use completion closures rather than sleeping or delaying tasks:

```swift
withAnimation(.bouncy) {
    scale = 1.2
} completion: {
    withAnimation(.smooth) {
        scale = 1
    }
}
```

This makes the chain depend on animation completion, not guessed timing.

## Animation Placement

Attach implicit animation as close as practical to the values it should animate:

```swift
Text(title)
    .opacity(isVisible ? 1 : 0)
    .animation(.easeInOut, value: isVisible)
```

Avoid placing a broad animation high in a screen unless every child change should animate.

## Disabling Animations

Use transactions when a subtree should not animate a particular update:

```swift
content
    .transaction { transaction in
        transaction.animation = nil
    }
```

Use this for counters, fast-updating values, or UI that should update instantly inside an animated parent.

## Transitions

Transitions need an animation context around the insertion or removal:

```swift
if showsDetails {
    DetailsPanel()
        .transition(.move(edge: .bottom).combined(with: .opacity))
}
```

```swift
withAnimation(.snappy) {
    showsDetails.toggle()
}
```

Common transitions:

- `.opacity`
- `.scale`
- `.move(edge:)`
- `.slide`
- `.push(from:)`
- `.blurReplace` on modern iOS targets

## Asymmetric Transitions

Use asymmetric transitions when insertion and removal should differ:

```swift
.transition(.asymmetric(
    insertion: .move(edge: .bottom).combined(with: .opacity),
    removal: .opacity
))
```

## Matched Geometry

Use matched geometry for two views that represent the same conceptual element across layouts:

```swift
@Namespace private var namespace

if isExpanded {
    LargeArtwork()
        .matchedGeometryEffect(id: "artwork", in: namespace)
} else {
    SmallArtwork()
        .matchedGeometryEffect(id: "artwork", in: namespace)
}
```

Rules:

- IDs must be stable and unique in the namespace.
- Both source and destination need to exist in the same update.
- Keep the animated element’s layout predictable.
- Do not use matched geometry as a substitute for simpler transitions.

## Performance

Prefer transforms over layout changes:

- Prefer `offset` over changing padding for movement.
- Prefer `scaleEffect` over changing frame for size emphasis.
- Prefer `opacity` for simple appearance.
- Avoid expensive work during animation-triggering state changes.

Keep animated subtrees small. A parent with many changing dependencies can make a simple animation stutter.

## Phase Animator

Use `phaseAnimator` for simple multi-step sequences:

```swift
enum PulsePhase: CaseIterable {
    case small
    case large

    var scale: Double {
        switch self {
        case .small: 1
        case .large: 1.08
        }
    }
}

content
    .phaseAnimator(PulsePhase.allCases, trigger: isPulsing) { content, phase in
        content.scaleEffect(phase.scale)
    }
```

Enums make phases readable and extensible.

## Keyframe Animator

Use keyframes for precise multi-track timing:

```swift
content
    .keyframeAnimator(initialValue: AnimationValues(), trigger: trigger) { content, value in
        content
            .scaleEffect(value.scale)
            .rotationEffect(value.rotation)
    } keyframes: { _ in
        KeyframeTrack(\.scale) {
            CubicKeyframe(1.2, duration: 0.2)
            SpringKeyframe(1.0, duration: 0.4)
        }
    }
```

Use keyframes when timing is part of the design, not for simple show/hide.

## Transactions

Transactions carry animation context through updates. Use them to override or inspect animation behavior in a subtree:

```swift
content
    .transaction { transaction in
        transaction.disablesAnimations = shouldDisable
    }
```

Custom transaction keys can pass animation-related context through a feature, but use them sparingly.

## Animatable and Custom Shapes

Use `@Animatable` for iOS 26+ custom animatable values when it expresses the shape clearly:

```swift
@Animatable
struct RingSegment: Shape {
    var startAngle: Angle
    var endAngle: Angle
    @AnimatableIgnored var isClockwise: Bool

    func path(in rect: CGRect) -> Path {
        Path()
    }
}
```

Use manual `Animatable` conformance when you need precise control or support older targets:

```swift
struct Wedge: Shape {
    var progress: Double

    var animatableData: Double {
        get { progress }
        set { progress = newValue }
    }

    func path(in rect: CGRect) -> Path {
        Path()
    }
}
```

## Review Checklist

- [ ] Implicit animations include `value`.
- [ ] Explicit animations wrap the state change.
- [ ] Transitions are paired with an animation context.
- [ ] Layout-heavy animation is avoided where transforms work.
- [ ] Chained animations use completion, not arbitrary sleeps.
- [ ] Matched geometry IDs are stable.
- [ ] Custom animatable values include all animating properties.

