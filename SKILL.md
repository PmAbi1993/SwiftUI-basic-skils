---
name: swiftui-app-craft
description: Build, review, and refactor iOS SwiftUI apps with modern SwiftUI APIs, Swift 6.2 defaults, Observation, state/data flow, navigation, layout, animations, Liquid Glass, SwiftData, performance, and Swift hygiene.
metadata:
  short-description: iOS SwiftUI app build and review coach
---

# SwiftUI App Craft

Use this skill when building, reviewing, or refactoring iOS SwiftUI app code. Be a decisive project coach: favor modern native SwiftUI, preserve existing project conventions, and report only issues that matter for correctness, maintainability, product quality, or performance.

## Operating Rules

- Assume iOS 26+ and Swift 6.2+ for new code unless the project declares a lower target.
- Prefer SwiftUI-native APIs. Use UIKit only when the project already uses it, SwiftUI has no suitable API, or the user asks for it.
- Do not add third-party frameworks without asking.
- Keep business logic out of view bodies; use testable models, services, or focused helpers according to the project style.
- Split substantial new Swift types into separate files when creating production code.
- For reviews, flag genuine problems only. Skip stylistic preferences unless they affect maintenance or app behavior.
- Load only the references relevant to the task.

## Workflow

### Build New UI or Features

1. Inspect the project target, file structure, data models, and existing style.
2. Choose state ownership before writing views: owned state, injected read-only data, injected mutable data, environment values, and async work.
3. Load `references/api-modernization.md`, `references/state-data-flow.md`, and whichever UI references match the feature.
4. Implement with native controls, type-safe navigation, small views, clean actions, and predictable data flow.
5. Add or update tests for non-trivial logic when the project has a test target.

### Review Existing Code

1. Load `references/api-modernization.md` first.
2. Route each concern through the Topic Router below.
3. Group findings by file and severity. Include line references when available.
4. For each issue, explain the rule and show a short before/after fix.
5. End with the highest-impact fixes first.

### Refactor or Modernize

1. Preserve behavior and public interfaces unless the user asks for larger changes.
2. Replace deprecated APIs with current iOS equivalents.
3. Reduce broad state dependencies, long bodies, repeated transformations, and fragile navigation.
4. Prefer small direct fixes over introducing new architecture.

## Topic Router

| Task area | Read |
|---|---|
| Deprecated or modern APIs | `references/api-modernization.md` |
| State, bindings, shared models, SwiftData | `references/state-data-flow.md` |
| View extraction, body hygiene, file shape | `references/view-composition.md` |
| Navigation, sheets, alerts, dialogs | `references/navigation-presentation.md` |
| Lists, `ForEach`, scroll views | `references/lists-scroll.md` |
| Layout, visual design, system controls | `references/layout-design.md` |
| Performance and hot paths | `references/performance.md` |
| Animations and transitions | `references/animations.md` |
| iOS 26 Liquid Glass | `references/liquid-glass.md` |
| Images, text, formatting, string handling | `references/images-text.md` |
| Swift 6.2, concurrency, code hygiene | `references/swift-hygiene.md` |

## Core Checklist

- [ ] Use `@Observable` for new shared state and `@State` to own observable instances.
- [ ] Mark view-owned `@State` as `private`.
- [ ] Never declare parent-provided values as `@State` or `@StateObject`.
- [ ] Use `@Binding` only when the child writes to parent state.
- [ ] Use `@Bindable` for injected observable models that need bindings.
- [ ] Use `NavigationStack` or iPad-appropriate `NavigationSplitView`, not `NavigationView`.
- [ ] Use value-based navigation with `navigationDestination(for:)` where practical.
- [ ] Use `.sheet(item:)` for optional model-driven sheets.
- [ ] Use `Button` for tappable commands unless tap count or location is required.
- [ ] Use stable identity in `ForEach`; never use `.indices` for dynamic content.
- [ ] Keep `body` pure and light: no heavy sorting, decoding, networking, or object creation.
- [ ] Use `.task` or `.task(id:)` for async work tied to view lifetime.
- [ ] Use `.animation(_:value:)`; never use broad `.animation(_:)`.
- [ ] Gate iOS 26 APIs with `#available` when supporting lower targets.

## Review Output

For code reviews, use this compact shape:

- `### FileName.swift`
- `**High: Problem title**`
- One short paragraph explaining why it matters.
- A small before/after Swift snippet.
- `### Priority` with the highest-impact fixes first.
