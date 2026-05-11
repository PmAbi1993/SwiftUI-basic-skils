# SwiftUI App Craft Reference Index

This folder is the copy-pasteable iOS SwiftUI reference pack. It intentionally focuses on iOS app implementation and review.

## How to Use This Pack

- For any SwiftUI task, start with the project target and existing conventions.
- Read only the topic files that match the task.
- Use the rules as practical defaults, not as a reason to rewrite working architecture.
- Prefer concrete before/after fixes in reviews.
- Treat performance advice as risk-based: apply hard rules immediately, suggest advanced optimization when the code path is hot or repeated.

## Reference Map

| File | Use for |
|---|---|
| `api-modernization.md` | Deprecated APIs, iOS version transitions, modern SwiftUI replacements, iOS 26 APIs |
| `mvvm.md` | Practical SwiftUI MVVM boundaries, view model design, dependency injection, testing, anti-patterns |
| `state-data-flow.md` | Observation, property wrappers, bindings, environment models, and SwiftData state rules |
| `swiftdata-migrations.md` | SwiftData usage with MVVM, `@Query`, `ModelContext`, repositories, versioned schemas, migrations |
| `view-composition.md` | View body structure, extraction, identity, containers, styling, UIKit bridges |
| `navigation-presentation.md` | NavigationStack, NavigationSplitView for iPad, paths, sheets, alerts, dialogs, popovers |
| `lists-scroll.md` | ForEach identity, List, Table, lazy stacks, empty states, scroll APIs, row dependencies |
| `performantSwiftUIListing.md` | Deep iOS 17+ guide for performant vertical and horizontal lists, row identity, data shaping, and listing anti-patterns |
| `layout-design.md` | Relative layout, system controls, forms, labels, spacing, styling, design consistency |
| `performance.md` | Redraw control, hot paths, lazy loading, view dependencies, body cost, advanced diffing |
| `animations.md` | Implicit and explicit animation, transitions, matched geometry, transactions, phase/keyframes |
| `focus-input.md` | Focus state, keyboard submit flows, search focus, forms, text input behavior |
| `images-text.md` | AsyncImage, downsampling, SF Symbols, FormatStyle, localized strings, rich text |
| `swift-hygiene.md` | Swift 6.2, concurrency, error handling, Foundation APIs, secrets, tests, lint cleanliness |

## Combined Source Roles

- The broad expert source contributes deep references for state management, view composition, layout, lists, scrolling, animations, images, input handling, and performance.
- The pro reviewer source contributes sharper project defaults: iOS 26+, Swift 6.2+, no third-party frameworks without asking, avoid UIKit unless needed, one substantial type per file, and findings that focus on real problems.

## Review Severity Guide

- **High**: Can cause incorrect UI, stale data, state loss, broken navigation, crashes, severe performance issues, or unsafe data handling.
- **Medium**: Deprecated APIs, fragile state ownership, unnecessary invalidation, maintainability problems in active code paths, or code likely to fail under project growth.
- **Low**: Local cleanup that improves clarity but is not behaviorally risky.

## Do Not Treat as a Full Rewrite Mandate

If the existing project uses legacy wrappers, destination-style navigation, or UIKit bridges, modernize only when the change is local, safe, and aligned with the request. Otherwise, report the issue and propose a staged migration.
