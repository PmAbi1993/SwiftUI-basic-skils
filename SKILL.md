---
name: swiftui-app-craft
description: Build, review, and refactor iOS SwiftUI apps using a deep combined reference pack for modern APIs, Observation, SwiftData, navigation, layout, lists, scrolling, animation, performance, image/text handling, and Swift 6.2 hygiene.
metadata:
  short-description: Deep iOS SwiftUI app craft reference pack
---

# SwiftUI App Craft

Use this skill for iOS SwiftUI app implementation, review, modernization, and refactoring. It combines the practical review strictness of SwiftUI Pro with the broader technical coverage of SwiftUI Expert, narrowed to iOS project work.

The `references/` folder is the main product. Load only the files needed for the task, but prefer the detailed references over improvising from memory.

## Defaults

- Target iOS 26+ and Swift 6.2+ for new code unless the project declares a lower deployment target.
- Prefer SwiftUI-native APIs and Swift Concurrency.
- Avoid UIKit unless SwiftUI has no suitable API, the project already depends on it, or the user asks.
- Do not introduce third-party frameworks without asking.
- Keep behavior changes explicit and avoid architecture rewrites during focused tasks.
- Preserve project conventions when they are reasonable.
- Report only genuine review findings: correctness, maintainability, project quality, performance, modern API usage, or data-flow problems.

## First Reference to Load

Start with `references/index.md`, then load the relevant topic files:

| Task | Required references |
|---|---|
| Build a feature or screen | `api-modernization.md`, `state-data-flow.md`, then UI topic files |
| Review SwiftUI code | `review-template.md`, `api-modernization.md`, `view-composition.md`, `state-data-flow.md`, `performance.md` |
| Modernize deprecated code | `api-modernization.md`, plus affected topic files |
| Design or review MVVM structure | `mvvm.md`, `state-data-flow.md`, `view-composition.md` |
| Fix state or data flow | `state-data-flow.md`, `performance.md` |
| SwiftData storage, fetching, or migrations | `swiftdata-migrations.md`, `state-data-flow.md`, `mvvm.md` |
| Navigation, sheets, alerts, dialogs | `navigation-presentation.md` |
| Lists, tables, scrolling, empty states | `lists-scroll.md`, `performance.md` |
| High-performance vertical or horizontal listing | `performantSwiftUIListing.md`, `lists-scroll.md`, `performance.md` |
| Layout and visual system consistency | `layout-design.md`, `view-composition.md` |
| Animation or transitions | `animations.md`, `performance.md` |
| Focus, keyboard, search, form input | `focus-input.md`, `api-modernization.md` |
| Images, search, text, formatting | `images-text.md`, `performance.md` |
| Swift language, concurrency, repo hygiene | `swift-hygiene.md` |

## Build Workflow

1. Inspect the existing project target, folder structure, naming, state style, and dependencies.
2. Identify owned state, injected read-only data, injected mutable data, environment values, and async work before editing.
3. Choose native SwiftUI controls and APIs from the references.
4. Keep view bodies simple: structure in the view, behavior in focused methods or models.
5. Add tests for non-trivial logic if the project already has tests or the change introduces meaningful logic.

## Review Workflow

1. Load `review-template.md` for the output structure.
2. Check modern API usage first.
3. Check state ownership, bindings, observable models, and async work.
4. Check navigation and presentation consistency.
5. Check view body size, extracted subviews, identity, list IDs, repeated transforms, and hot paths.
6. Check Swift hygiene and project safety issues.
7. Organize findings by severity and file. For each finding, include impact, location, and a concrete fix.

## Refactor Workflow

1. Preserve behavior unless the user asks for behavior changes.
2. Make the smallest structural improvement that removes the real problem.
3. Prefer direct SwiftUI modernization over adding new abstractions.
4. Keep each extracted view or model purpose-specific.
5. Re-run project checks available in the repo when practical.

## Core Hard Rules

- Use `@Observable` for new shared mutable state.
- Use `@State` to own observable instances in views.
- Mark view-owned `@State` and legacy `@StateObject` as `private`.
- Never declare parent-provided values as `@State` or `@StateObject`.
- Use `@Binding` only when the child writes parent state.
- Use `@Bindable` when an injected observable model needs bindings.
- Use `NavigationStack` or iPad-appropriate `NavigationSplitView`, not `NavigationView`.
- Use `.sheet(item:)` for optional model-driven sheets.
- Use `Button` for commands unless tap count or tap location is required.
- Use stable identity in `ForEach`; never use `.indices` for dynamic content.
- Keep `body` pure and cheap.
- Use `.task` or `.task(id:)` for async work tied to view lifetime.
- Use `.animation(_:value:)`, not broad `.animation(_:)`.
- Gate iOS 26 APIs with `#available` if the app supports lower versions.
