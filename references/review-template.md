# SwiftUI Code Review Template

Use this template to organize SwiftUI code reviews consistently. The goal is to make reviews easy to scan, actionable, and focused on issues that affect correctness, maintainability, performance, or project quality.

## Review Principles

- Report genuine issues only.
- Be specific: name the file, line, behavior, and fix.
- Prefer one clear recommendation over several vague alternatives.
- Separate blocking issues from improvements.
- Do not mix personal taste with correctness.
- Include examples only when they clarify the fix.
- If no issue is found in an area, say so briefly in the checklist rather than inventing a finding.

## Full Review Format

~~~markdown
# SwiftUI Review

## Summary

Short 2-4 sentence overview of the review result.

- Scope reviewed: files, feature, PR, or module.
- Overall risk: Low / Medium / High.
- Main themes: e.g. state ownership, list identity, navigation, performance.

## Blocking Issues

Issues that should be fixed before merge because they can cause incorrect behavior, crashes, data loss, broken navigation, severe performance problems, or unsafe persistence.

### 1. [High] Short Finding Title

- File: `Path/To/File.swift`
- Line: `123`
- Area: State / Navigation / List / SwiftData / Performance / API / MVVM / Layout / Animation / Hygiene
- Rule: Name the violated rule.
- Impact: Explain what can go wrong.
- Recommendation: Say exactly what to change.

```swift
// Current
...

// Recommended
...
```

## Important Improvements

Issues that are not immediate blockers but should be addressed because they create fragility, stale UI, unnecessary redraws, migration risk, or maintainability cost.

### 2. [Medium] Short Finding Title

- File: `Path/To/File.swift`
- Line: `45`
- Area: Performance
- Rule: Keep expensive transforms out of `body`.
- Impact: This work can repeat during normal view updates.
- Recommendation: Move the transform to the view model or a cached derived value with explicit invalidation.

## Minor Cleanup

Small improvements that are safe and local. Keep this section short.

### 3. [Low] Short Finding Title

- File: `Path/To/File.swift`
- Line: `88`
- Recommendation: Short direct recommendation.

## Positive Notes

- Mention good existing choices that should be preserved.
- Keep this brief and technical.

## Review Checklist

- Modern SwiftUI APIs: Pass / Issues found / Not reviewed
- State ownership and bindings: Pass / Issues found / Not reviewed
- MVVM boundaries: Pass / Issues found / Not reviewed
- SwiftData and migrations: Pass / Issues found / Not reviewed
- Navigation and presentation: Pass / Issues found / Not reviewed
- Lists and scrolling: Pass / Issues found / Not reviewed
- View composition: Pass / Issues found / Not reviewed
- Performance: Pass / Issues found / Not reviewed
- Images and text formatting: Pass / Issues found / Not reviewed
- Swift hygiene and concurrency: Pass / Issues found / Not reviewed

## Suggested Fix Order

1. Fix the highest-risk behavior first.
2. Fix follow-on issues that depend on it.
3. Apply cleanup after behavior is stable.

## Sign-Off

Review status: Blocked / Changes recommended / Looks good with notes / Looks good
~~~

## Short Review Format

Use this when the review is small:

~~~markdown
# SwiftUI Review

## Findings

### [High] Finding title

- File: `Path/To/File.swift`
- Line: `123`
- Impact: What can go wrong.
- Fix: What to change.

## Checklist

- State/data flow: Pass
- Modern APIs: Pass
- Performance: Issues found

## Status

Changes recommended.
~~~

## Finding Title Style

Use direct technical titles:

- `[High] Row identity uses array indices`
- `[High] Edit screen mutates persisted model before Save`
- `[Medium] View body performs repeated sorting`
- `[Medium] Row reads broad app state`
- `[Medium] Destination registration is duplicated`
- `[Low] Button action can use direct method reference`

Avoid vague titles:

- `Issue in list`
- `Bad performance`
- `Needs cleanup`
- `Wrong code`

## Severity Guide

### High

Use for issues that can cause:

- crashes
- stale or incorrect UI
- data loss
- state reset
- broken navigation
- duplicate network or persistence writes
- unsafe migration behavior
- severe list/feed performance problems
- incorrect SwiftData save/cancel behavior

### Medium

Use for issues that can cause:

- unnecessary redraws
- fragile state ownership
- hard-to-test view logic
- repeated work in `body`
- deprecated APIs in active code
- broad row dependencies
- maintainability problems likely to grow
- missing migration planning for model changes

### Low

Use for:

- small readability improvements
- local modernization with low risk
- minor project convention drift
- small extraction opportunities
- simple direct-action cleanup

## Area Labels

Use one area label per finding:

- Modern API
- State/Data Flow
- MVVM
- SwiftData
- Migration
- Navigation
- Presentation
- List/Scroll
- View Composition
- Performance
- Layout
- Animation
- Image/Text
- Swift Hygiene
- Tests

## Inline Comment Format

For GitHub or code-review inline comments:

~~~markdown
[Medium] This row uses index identity. If the collection is deleted, inserted, or reordered, SwiftUI can reuse the wrong row state. Use stable model identity instead:

```swift
ForEach(items) { item in
    ItemRow(item: item)
}
```
~~~

Keep inline comments short. Put broad architectural guidance in the main review summary.

## Common Review Sections by Topic

### State/Data Flow

Check:

- view-owned state is private
- parent-provided values are not stored as `@State`
- `@Binding` is used only for child mutation
- owned observable models use `@State`
- injected observable models use `@Bindable` only when bindings are needed
- manual `Binding(get:set:)` is justified

### MVVM

Check:

- view model has a clear screen or feature responsibility
- view remains declarative
- dependencies are injected
- async operations expose loading/error state
- drafts are used for save/cancel edit flows
- navigation mechanics stay in the view unless project routing says otherwise

### SwiftData

Check:

- simple live lists use `@Query` where appropriate
- complex fetch/save logic lives in a repository or service
- save/cancel editing does not mutate persisted data prematurely
- migrations are planned for model shape changes
- migration tests cover existing stores

### Lists and Scrolling

Check:

- `ForEach` identity is stable
- dynamic lists do not use indices
- row inputs are narrow
- no row-level object creation
- no image decoding in rows
- no broad environment reads in every row
- vertical custom feeds use `LazyVStack`
- horizontal carousels use `LazyHStack`
- pagination is guarded

### Performance

Check:

- no heavy work in `body`
- hot-path state writes are gated
- async work uses `.task` where appropriate
- derived collections are not stale
- unnecessary `AnyView` is avoided
- repeated formatting is avoided

## Example Completed Review

~~~markdown
# SwiftUI Review

## Summary

Reviewed the project list and edit flow. Overall risk is Medium: the UI is structurally close, but list identity and edit-state ownership need attention before this becomes harder to maintain.

## Blocking Issues

### 1. [High] Project rows use index identity

- File: `ProjectListScreen.swift`
- Line: `42`
- Area: List/Scroll
- Rule: Dynamic `ForEach` content must use stable model identity.
- Impact: Deleting or reordering projects can reuse the wrong row state or render stale content.
- Recommendation: Iterate over identifiable projects directly.

```swift
// Current
ForEach(projects.indices, id: \.self) { index in
    ProjectRow(project: projects[index])
}

// Recommended
ForEach(projects) { project in
    ProjectRow(project: project)
}
```

## Important Improvements

### 2. [Medium] Edit screen mutates the persisted model before Save

- File: `EditProjectScreen.swift`
- Line: `18`
- Area: SwiftData
- Rule: Save/cancel edit flows should use a draft.
- Impact: Cancel cannot reliably discard edits because the model is already mutated.
- Recommendation: Edit a `ProjectDraft`, then apply it to the model only when Save is tapped.

## Review Checklist

- Modern SwiftUI APIs: Pass
- State ownership and bindings: Issues found
- SwiftData and migrations: Issues found
- Lists and scrolling: Issues found
- Performance: Pass

## Suggested Fix Order

1. Fix project list identity.
2. Add draft-based editing for the edit screen.
3. Add a regression test for cancel behavior.

## Sign-Off

Review status: Changes recommended.
~~~
