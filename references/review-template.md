# SwiftUI Branch and Diff Review Template

Use this reference when reviewing an iOS SwiftUI PR, branch, local diff, or pasted diff, especially when the agent is expected to post a review. The review should be easy to act on, but it must also teach the reason behind each request. A finding without a clear "why" is usually a weak finding.

## Review Anchors

This template follows three public review patterns:

- Google Engineering Practices: optimize for improving code health over time, not perfection. See https://google.github.io/eng-practices/review/reviewer/standard.html
- Google review comments: be kind, explain reasoning, and label comment severity. See https://google.github.io/eng-practices/review/reviewer/comments.html
- Conventional Comments: use a parseable label plus optional decorations. See https://conventionalcomments.org/
- iOS checklist examples: cover architecture, warnings, single source of truth, force unwraps, permissions, performance, accessibility, and build configuration. See https://dev.to/bornfightcompany/ios-code-review-checklist-53ia and https://gist.github.com/fsferrara/fdb6885643dbf527163efc603b6d701c

Use iOS checklist-style coverage for app-specific risks: UI behavior, architecture, compatibility, performance, permissions, build configuration, tests, and accessibility.

## Agent Review Workflow

### 1. Establish Scope

For a GitHub PR:

- Identify the PR base branch, head branch, changed files, changed lines, CI status, and test status if available.
- Review the PR diff first, then read surrounding source for each changed Swift file.
- If posting inline comments, remember that most review systems only allow inline comments on changed lines.

For a local branch:

- Identify the current branch and likely base branch.
- If the user did not provide a base, infer `main`, `master`, or `develop` from the repo.
- Use the merge base, then inspect the branch diff:

```bash
git merge-base HEAD <base>
git diff --stat <base>...HEAD
git diff --name-status <base>...HEAD
git diff --unified=80 <base>...HEAD -- '*.swift'
```

For a pasted diff:

- Review only the pasted diff and any provided context.
- State that the review is limited if surrounding files, tests, project settings, or target versions are unavailable.

Always include `Scope reviewed` in the posted review body.

### 2. Load Relevant Skill References

Load `references/index.md`, then at minimum:

- `review-template.md`
- `api-modernization.md`
- `view-composition.md`
- `state-data-flow.md`
- `performance.md`

Load topic files only when the diff touches that area:

- Navigation, sheets, alerts, dialogs: `navigation-presentation.md`
- SwiftData, persistence, model changes, migrations: `swiftdata-migrations.md`
- Lists, feeds, tables, carousels: `lists-scroll.md` and possibly `performantSwiftUIListing.md`
- MVVM or dependency boundaries: `mvvm.md`
- Layout and visual consistency: `layout-design.md`
- Animation: `animations.md`
- Focus, keyboard, forms, search: `focus-input.md`
- Images, formatting, localized text: `images-text.md`
- Swift language, concurrency, tests, secrets: `swift-hygiene.md`

### 3. Read Beyond the Hunk

For each changed Swift file, inspect enough context to understand:

- the enclosing type, view, model, or extension
- property wrappers and state ownership
- navigation and presentation ownership
- data source and persistence lifetime
- async task lifetime and cancellation behavior
- identities used by `ForEach`, `List`, `Table`, and navigation paths
- nearby tests and previews if the behavior is user-facing
- deployment target and availability guards when new APIs are used

Do not make broad architecture claims from a small hunk unless surrounding code supports the claim.

### 4. Review In This Order

1. User-visible behavior and data safety.
2. State ownership, Observation, bindings, edit drafts, and single source of truth.
3. Navigation, presentation, modal ownership, and iPad split-view needs.
4. SwiftData, persistence boundaries, migrations, save/cancel behavior, and test coverage.
5. List identity, scrolling behavior, row dependencies, and hot-path rendering cost.
6. Async work, concurrency isolation, cancellation, duplicate tasks, and main-actor correctness.
7. API modernization, availability, platform conventions, and UIKit escape hatches.
8. Accessibility, Dynamic Type, localization, privacy strings, and permission flows.
9. Tests, previews, build warnings, lint issues, and maintainability.

### 5. Build Findings

Every non-praise finding must satisfy this contract:

- What: the exact problem.
- Why it matters: the SwiftUI, iOS, product, or project principle behind the request.
- Failure mode: what can actually go wrong for users, developers, data, performance, or future changes.
- Evidence: the file and line, plus nearby context if needed.
- Fix: one concrete, scoped recommendation.
- Severity: High, Medium, or Low based on impact, not reviewer preference.

If you cannot explain the why, either convert the note to a question, mark it as non-blocking, or omit it.

### 6. Post the Review

Prefer inline comments for precise line-level issues. Use the top-level review body for summary, cross-file findings, architectural concerns, checklist status, and fix order.

Post one coherent review after drafting the findings. Avoid posting as you discover issues because early comments often lack context from later files.

Use these review statuses:

- Request changes: at least one High severity issue should block merge.
- Comment: no blockers, but Medium or Low issues should be considered.
- Approve: no required changes remain, and the user or workflow allows the agent to approve.

If posting through a GitHub-style API, map these to `REQUEST_CHANGES`, `COMMENT`, and `APPROVE`.

Do not block on taste, formatter output, naming preferences without project support, or speculative future architecture.

## Comment Format

Use Conventional Comments style:

```text
<label> (<severity>,<area>,<blocking|non-blocking>): <short finding title>
```

Recommended labels:

- `issue`: a concrete problem.
- `suggestion`: an improvement with a clear benefit.
- `question`: a concern that needs author context before becoming a finding.
- `nitpick`: trivial preference or polish, non-blocking by default.
- `chore`: process work such as generated files, docs, or CI.
- `note`: useful context, non-blocking.
- `praise`: specific positive feedback.

Recommended areas:

- `state`
- `binding`
- `navigation`
- `presentation`
- `swiftdata`
- `migration`
- `list`
- `performance`
- `concurrency`
- `api`
- `layout`
- `animation`
- `accessibility`
- `privacy`
- `tests`
- `hygiene`

## Inline Comment Template

Use this for GitHub, GitLab, or any line-level review comment:

~~~markdown
issue (high,state,blocking): Child view owns parent-provided state

Why it matters: `@State` creates view-owned storage. When this value actually belongs to the parent, the child can drift away from the source of truth and SwiftUI can reset it as view identity changes.

What can happen: The row may show stale data after the parent updates, and edits made in the child may not propagate reliably.

Fix: Pass a plain value for read-only display, or use `@Binding` only if the child needs to mutate the parent-owned value.
~~~

If a code suggestion is useful and safe, add a small patch:

~~~markdown
```suggestion
@Binding var project: Project
```
~~~

Only include suggestion blocks that are likely to compile in the surrounding code.

## Top-Level PR Review Template

Use this as the review body when posting a branch or PR review:

~~~markdown
# SwiftUI Review

Status: Request changes / Comment / Approve
Scope reviewed: <PR #, branch comparison, files, or pasted diff>
Overall risk: Low / Medium / High

## Summary

<2-4 sentences. Name the main risk, why it matters, and whether the branch is mergeable. Avoid repeating the full checklist here.>

## Blocking Findings

### 1. issue (high,<area>,blocking): <short finding title>

- File: `<Path/To/File.swift>`
- Line: `<line>`
- Why it matters: <Explain the SwiftUI/iOS/project principle.>
- Failure mode: <What can go wrong if this merges.>
- Evidence: <Point to the changed line and relevant surrounding behavior.>
- Recommended change: <Concrete, scoped fix.>

## Non-Blocking Findings

### 2. suggestion (medium,<area>,non-blocking): <short finding title>

- File: `<Path/To/File.swift>`
- Line: `<line>`
- Why it matters: <Explain the maintainability, performance, or product reason.>
- Tradeoff: <Why this can be follow-up instead of a blocker.>
- Recommended change: <Concrete fix or follow-up.>

## Questions

- question (<area>,non-blocking): <Ask only questions that affect correctness, product behavior, or review confidence.>

## Positive Notes

- praise (<area>): <Call out specific good decisions and why they are worth preserving.>

## Checklist

- Modern SwiftUI APIs and availability: Pass / Issues found / Not reviewed
- State ownership, Observation, and bindings: Pass / Issues found / Not reviewed
- MVVM boundaries and dependency injection: Pass / Issues found / Not reviewed
- SwiftData, persistence, and migrations: Pass / Issues found / Not reviewed
- Navigation and presentation: Pass / Issues found / Not reviewed
- Lists, identity, and scrolling: Pass / Issues found / Not reviewed
- View composition and layout: Pass / Issues found / Not reviewed
- Performance and hot paths: Pass / Issues found / Not reviewed
- Async/concurrency and main-actor safety: Pass / Issues found / Not reviewed
- Accessibility, Dynamic Type, localization, and permissions: Pass / Issues found / Not reviewed
- Tests, previews, warnings, and lint: Pass / Issues found / Not reviewed

## Suggested Fix Order

1. <Highest-risk behavior or data issue.>
2. <Dependent fixes.>
3. <Tests or cleanup after behavior is stable.>
~~~

Omit empty sections. Keep the checklist, but mark areas as `Not reviewed` when the diff does not provide enough evidence.

## Posting Payload Shape

When a connector or API needs structured review data, map the review like this:

~~~yaml
event: REQUEST_CHANGES | COMMENT | APPROVE
body: |
  <top-level review body from the template>
comments:
  - path: Sources/App/FeatureView.swift
    line: 42
    side: RIGHT
    body: |
      issue (high,state,blocking): <title>

      Why it matters: <why>
      What can happen: <failure mode>
      Fix: <recommendation>
~~~

Rules for posting:

- Put exact line-level findings in `comments`.
- Put cross-file, architectural, or non-diff-line findings in `body`.
- If a line is not commentable in the review tool, reference `File.swift:line` in the body instead.
- Keep each inline comment focused on one issue.
- Submit the review only after all comments and the body are coherent together.

## Short Review Template

Use this when the diff is small:

~~~markdown
# SwiftUI Review

Status: Comment / Approve / Request changes
Scope reviewed: <files or diff>
Overall risk: Low / Medium / High

## Findings

### 1. issue (high,<area>,blocking): <title>

- File: `<Path/To/File.swift>`
- Line: `<line>`
- Why it matters: <principle or user impact>
- Failure mode: <concrete risk>
- Fix: <concrete recommendation>

## Notes

- praise (<area>): <specific positive note, with why>

## Checklist

- State/data flow: Pass / Issues found / Not reviewed
- Modern APIs: Pass / Issues found / Not reviewed
- Performance: Pass / Issues found / Not reviewed
- Tests: Pass / Issues found / Not reviewed
~~~

## Severity Guide

### High

Use High for issues that can cause:

- crashes
- data loss or premature persistence
- stale or incorrect UI
- broken navigation or impossible dismissal
- duplicate network or persistence writes
- privacy, entitlement, or permission risk
- unsafe SwiftData migration behavior
- concurrency races, unbounded tasks, or main-thread violations
- severe scrolling, feed, image, or animation performance problems

High severity usually means `Request changes`.

### Medium

Use Medium for issues that can cause:

- fragile state ownership
- unnecessary view invalidation
- repeated work in `body`
- hard-to-test view logic
- deprecated APIs in active code
- broad row dependencies
- avoidable accessibility or Dynamic Type gaps
- missing tests for meaningful state, persistence, or async behavior
- maintainability problems likely to grow with normal feature work

Medium severity usually means `Comment`, unless several Medium findings combine into a clear merge risk.

### Low

Use Low for:

- small readability improvements
- local modernization with low risk
- minor project convention drift
- simple extraction opportunities
- minor naming clarity
- small direct-action cleanup

Low severity should be non-blocking.

## Why-First Checklist

Before posting any finding, answer these questions privately:

- Why does this matter for an iOS user, developer, data model, or app lifecycle?
- Is the concern supported by SwiftUI behavior, platform requirements, project conventions, or evidence in the diff?
- What concrete failure mode or maintenance cost follows from merging this?
- Why should it be fixed in this PR instead of a follow-up?
- Is the recommendation small enough for the author to act on without a rewrite?

If the answer to the first three questions is weak, do not post it as an issue.

## iOS SwiftUI Review Checklist

| Area | Look For | Why It Matters |
|---|---|---|
| Behavior | The changed flow still matches the product intent and handles empty, loading, error, and cancellation states. | UI code can compile while still failing the user's real path. |
| State/Data Flow | `@State` owns only local view state, parent data is not copied into child state, `@Binding` is used only for child mutation, and `@Bindable` is used only when bindings are needed. | SwiftUI identity and Observation depend on clear ownership; unclear ownership causes stale UI, resets, and write paths that are hard to reason about. |
| Edit Flows | Save/cancel screens use drafts instead of mutating persisted models immediately. | Users expect cancel to discard edits, and premature mutation can persist or display data before confirmation. |
| Navigation | `NavigationStack` or iPad-appropriate `NavigationSplitView` is used consistently, destinations are registered once, and sheets use model-driven presentation where appropriate. | Navigation bugs often appear as stuck screens, duplicate pushes, or incorrect restoration. |
| SwiftData | Fetching, saving, relationships, delete behavior, and migrations are explicit and tested when model shape changes. | Persistence mistakes can corrupt or lose user data and are hard to repair after release. |
| Lists/Scrolling | Dynamic content uses stable identity, rows read narrow inputs, expensive work is outside row/body hot paths, and pagination is guarded. | SwiftUI reuses views aggressively; unstable identity and broad dependencies cause stale row state and scroll hitches. |
| Performance | `body` stays pure and cheap, heavy transforms are cached or moved, images are sized/decoded appropriately, and animations are scoped. | iOS performance problems are often introduced by small repeated costs during normal view updates. |
| Async/Concurrency | View-bound work uses `.task` or `.task(id:)`, duplicate work is avoided, cancellation is respected, and UI state is updated on the correct actor. | Async bugs become duplicate network calls, stale results, memory leaks, or UI updates after the view is gone. |
| APIs/Availability | Deprecated APIs are avoided in touched code, iOS 26 APIs are gated when the app supports lower versions, and UIKit bridges are justified. | Platform drift increases migration cost and availability mistakes can crash on supported devices. |
| Layout/Accessibility | Dynamic Type, VoiceOver labels, contrast, dark mode, safe areas, keyboard, iPad, and small-screen layouts still work. | The first broken thing users notice is often layout or accessibility, not architecture. |
| Images/Text | Images are cached/downsampled where needed, symbols use SF Symbol conventions, user-facing strings are localizable, and formatting uses `FormatStyle`. | Media and formatting issues create memory pressure, inconsistent UI, and localization debt. |
| Tests/Previews | Meaningful behavior changes include tests where the project has test coverage, and previews are useful for visual states. | Tests preserve the why after the review conversation disappears. |
| Build Hygiene | No new warnings, secrets, debug logs, broad formatting churn, or unrelated rewrites. | Noise hides the real change and makes rollbacks and future reviews harder. |

## Common Finding Examples

### State Ownership

~~~markdown
issue (high,state,blocking): The edit screen stores a parent model in `@State`

Why it matters: `@State` is for view-owned storage. A parent-provided model should keep one source of truth, otherwise the edit screen can show a copied value that no longer matches the parent.

What can happen: A parent refresh can leave this screen editing stale data, and saving may overwrite newer changes.

Fix: Pass the model as a plain dependency for read-only display, or use a draft plus an explicit save action for mutation.
~~~

### List Identity

~~~markdown
issue (high,list,blocking): `ForEach` uses indices for dynamic rows

Why it matters: SwiftUI uses identity to preserve row state. Array indices change when items are inserted, removed, sorted, or filtered.

What can happen: The wrong row can retain focus, selection, disclosure state, animations, or cached child state after deletion or reordering.

Fix: Iterate over stable model identity:

```swift
ForEach(projects) { project in
    ProjectRow(project: project)
}
```
~~~

### Performance

~~~markdown
suggestion (medium,performance,non-blocking): The view sorts projects inside `body`

Why it matters: SwiftUI may re-evaluate `body` frequently. Sorting in `body` repeats work during unrelated state updates and can become visible as list size grows.

Tradeoff: This is not a merge blocker if the list is small today, but it is a good follow-up because this screen is a repeated interaction surface.

Fix: Move sorting to the view model, query descriptor, or a cached derived value with explicit invalidation.
~~~

## When Not To Comment

Skip the comment or mark it as non-blocking when:

- the only reason is personal taste
- the formatter or linter already owns the issue
- the code is outside the reviewed scope
- the concern depends on speculative future requirements
- the recommendation would require a broad rewrite unrelated to the PR
- you cannot explain the user, data, performance, or maintainability impact

## Review Result Phrases

Use clear status language:

- `Request changes`: "I found one blocking issue that can cause <impact>. I would fix that before merge."
- `Comment`: "No blockers from this pass. I left a few non-blocking notes around <area>."
- `Approve`: "Looks good from this SwiftUI review. The changed flow keeps <important behavior> intact."

Never write "LGTM" alone. Include the reason the branch is safe to merge.
