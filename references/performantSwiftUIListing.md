# Performant SwiftUI Listing

This is the company-standard guide for building performant iOS 17+ SwiftUI listing screens. It covers vertical lists, horizontal carousels, `ForEach`, `List`, `ScrollView`, lazy stacks, data shaping, row identity, and the common patterns that quietly make SwiftUI screens slow, fragile, or hard to maintain.

## Core Rule

A SwiftUI list is only as good as its identity, data shape, and row dependency boundary.

The fastest row is not the one with clever modifiers. It is the row that:

- has a stable ID
- receives only the data it needs
- does not do work in `body`
- does not read broad app state
- does not decode images
- does not perform formatting repeatedly
- does not create new objects during rendering
- does not hide its structure behind type erasure

## Choose the Right Container

| UI need | Preferred container |
|---|---|
| Standard iOS rows, swipe actions, edit mode, sections, refresh | `List` |
| Custom vertical feed/cards | `ScrollView` + `LazyVStack` |
| Horizontal carousel | horizontal `ScrollView` + `LazyHStack` |
| Paged horizontal cards | horizontal `ScrollView` + `LazyHStack` + `scrollTargetBehavior` |
| Grid of items | `LazyVGrid` or `LazyHGrid` |
| Dense iPad table-like data | `Table` with compact fallback |
| Tiny static group of views | `VStack` or `HStack` |

Do not use `ScrollView` + `VStack` for a large dynamic list. Do not use `List` and then fight every default behavior if the design is really a custom card feed.

## The Standard Vertical List Pattern

Use `List` when the UI is a standard iOS list:

```swift
struct ProjectListScreen: View {
    @State private var viewModel = ProjectListViewModel()

    var body: some View {
        List {
            Section("Active") {
                ForEach(viewModel.activeProjects) { project in
                    ProjectRow(
                        id: project.id,
                        name: project.name,
                        updatedAt: project.updatedAt,
                        status: project.status
                    )
                }
            }
        }
        .navigationTitle("Projects")
        .refreshable {
            await viewModel.refresh()
        }
        .task {
            await viewModel.load()
        }
    }
}
```

The row receives narrow values. The screen owns loading. The view model owns data shaping.

## The Standard Custom Vertical Feed Pattern

Use `ScrollView` + `LazyVStack` for custom cards:

```swift
struct ActivityFeedScreen: View {
    @State private var viewModel = ActivityFeedViewModel()

    var body: some View {
        ScrollView {
            LazyVStack(spacing: 12) {
                ForEach(viewModel.items) { item in
                    ActivityCard(
                        title: item.title,
                        subtitle: item.subtitle,
                        timestamp: item.timestamp,
                        thumbnailURL: item.thumbnailURL
                    )
                }
            }
            .padding(.horizontal, 16)
            .padding(.vertical, 12)
        }
        .refreshable {
            await viewModel.refresh()
        }
        .task {
            await viewModel.load()
        }
    }
}
```

Use `LazyVStack`, not `VStack`, for dynamic or potentially large content.

## The Standard Horizontal Carousel Pattern

Use a horizontal `ScrollView` with `LazyHStack`:

```swift
struct RecentProjectsCarousel: View {
    let projects: [ProjectSummary]
    let openProject: (ProjectSummary.ID) -> Void

    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack(spacing: 12) {
                ForEach(projects) { project in
                    ProjectCard(project: project) {
                        openProject(project.id)
                    }
                    .frame(width: 280)
                }
            }
            .padding(.horizontal, 16)
            .scrollTargetLayout()
        }
        .scrollIndicators(.hidden)
        .scrollTargetBehavior(.viewAligned)
        .contentMargins(.horizontal, 16, for: .scrollContent)
    }
}
```

Use `LazyHStack`, not `HStack`, when item count is dynamic or non-trivial.

## The Standard Paged Horizontal Pattern

For full-width pages:

```swift
struct OnboardingPages: View {
    let pages: [OnboardingPage]

    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack(spacing: 0) {
                ForEach(pages) { page in
                    OnboardingPageView(page: page)
                        .containerRelativeFrame(.horizontal)
                }
            }
            .scrollTargetLayout()
        }
        .scrollIndicators(.hidden)
        .scrollTargetBehavior(.paging)
    }
}
```

`containerRelativeFrame(.horizontal)` is preferable to hard-coding screen width.

## Identity Rules

Stable identity is non-negotiable in dynamic lists.

Good:

```swift
struct Project: Identifiable {
    let id: UUID
    var name: String
}

ForEach(projects) { project in
    ProjectRow(project: project)
}
```

Also acceptable:

```swift
ForEach(projects, id: \.serverID) { project in
    ProjectRow(project: project)
}
```

Bad:

```swift
ForEach(projects.indices, id: \.self) { index in
    ProjectRow(project: projects[index])
}
```

Bad:

```swift
ForEach(projects, id: \.name) { project in
    ProjectRow(project: project)
}
```

Names change and can collide. Identity must be unique and stable across updates.

## IDs That Are Not Good Enough

Avoid these as IDs for dynamic content:

- array index
- display title
- email address if duplicates are possible
- URL if duplicate URLs are possible
- date timestamp if collisions are possible
- `UUID()` generated inside `body`
- `\.self` for mutable values
- class default `ObjectIdentifier` when object lifetime can change

Create a real ID at the model boundary.

## Enumerated Rows

For display numbering in Swift 6.2+ projects, use the element identity:

```swift
ForEach(items.enumerated(), id: \.element.id) { offset, item in
    NumberedRow(number: offset + 1, item: item)
}
```

Never use the offset as the business identity for mutable lists. The number can change; the item identity should not.

If an older toolchain cannot use `enumerated()` directly in `ForEach`, convert only as a compatibility workaround and still identify by the element ID:

```swift
ForEach(Array(items.enumerated()), id: \.element.id) { offset, item in
    NumberedRow(number: offset + 1, item: item)
}
```

## Constant Row Structure

A `ForEach` element should usually produce one row view:

```swift
ForEach(items) { item in
    ItemRow(item: item)
}
```

Avoid producing a variable number of sibling views per element:

```swift
ForEach(items) { item in
    if item.isExpanded {
        HeaderRow(item: item)
        DetailRow(item: item)
    } else {
        HeaderRow(item: item)
    }
}
```

Prefer:

```swift
ForEach(items) { item in
    ItemRow(item: item)
}

struct ItemRow: View {
    let item: Item

    var body: some View {
        VStack(alignment: .leading) {
            HeaderRow(item: item)

            if item.isExpanded {
                DetailRow(item: item)
            }
        }
    }
}
```

This keeps the list identity model simple.

## Data Shaping Belongs Before the List

Do not sort, filter, group, map, or deduplicate in the `ForEach` expression for non-trivial data.

Bad:

```swift
ForEach(items.filter { $0.isVisible }.sorted { $0.name < $1.name }) { item in
    ItemRow(item: item)
}
```

Better:

```swift
ForEach(viewModel.visibleItems) { item in
    ItemRow(item: item)
}
```

The view model can own:

```swift
var visibleItems: [Item] {
    items
        .filter(\.isVisible)
        .sorted { $0.name.localizedStandardCompare($1.name) == .orderedAscending }
}
```

If the collection is large or recalculated often, cache it with explicit invalidation.

## Do Not Cache Derived Collections Casually

Bad:

```swift
@State private var filteredItems: [Item] = []
```

If the source changes and the cache is not invalidated perfectly, the UI becomes stale.

Use cached derived collections only when:

- the transform is expensive
- the source of truth is clear
- every source change updates the cache
- tests cover the invalidation

## Row Inputs Must Be Narrow

Bad:

```swift
TaskRow(task: task, appState: appState)
```

Better:

```swift
TaskRow(
    title: task.title,
    dueDate: task.dueDate,
    isOverdue: task.isOverdue,
    tint: appState.theme.taskTint
)
```

Rows should not read a broad app model unless they genuinely need broad app state.

## Avoid Environment Reads in Every Row

This is common and costly:

```swift
struct TaskRow: View {
    @Environment(AppModel.self) private var appModel
    let task: Task

    var body: some View {
        Text(task.title)
            .foregroundStyle(appModel.theme.tint)
    }
}
```

Prefer:

```swift
TaskRow(task: task, tint: appModel.theme.tint)
```

Environment is excellent for stable shared values. It is a poor place for high-frequency row state.

## Avoid Per-Row Work in Body

Bad:

```swift
struct InvoiceRow: View {
    let invoice: Invoice

    var body: some View {
        Text(formatter.string(from: invoice.date))
        Text(invoice.items.sorted { $0.name < $1.name }.first?.name ?? "")
    }
}
```

Better:

```swift
struct InvoiceRow: View {
    let title: String
    let dateText: String
    let primaryItemName: String

    var body: some View {
        Text(title)
        Text(dateText)
        Text(primaryItemName)
    }
}
```

Prepare display values before repeated rendering, or use `Text(value, format:)` for cheap system formatting:

```swift
Text(invoice.date, format: .dateTime.month().day().year())
```

## Image Loading in Lists

Rows must not decode large images synchronously.

Avoid:

```swift
if let image = UIImage(data: data) {
    Image(uiImage: image)
}
```

Prefer:

- `AsyncImage` for simple remote images
- a project image loader/cache for production feeds
- downsampled images sized for display
- placeholders that preserve row dimensions

```swift
AsyncImage(url: thumbnailURL) { phase in
    switch phase {
    case .success(let image):
        image.resizable().scaledToFill()
    case .failure:
        Image(systemName: "photo")
    case .empty:
        ProgressView()
    @unknown default:
        EmptyView()
    }
}
.frame(width: 56, height: 56)
.clipShape(.rect(cornerRadius: 8))
```

Stable image dimensions prevent row height churn.

## AnyView Is Not a Row Strategy

Bad:

```swift
ForEach(items) { item in
    AnyView(item.isSpecial ? SpecialRow(item: item) : RegularRow(item: item))
}
```

Better:

```swift
ForEach(items) { item in
    ItemRow(item: item)
}

struct ItemRow: View {
    let item: Item

    var body: some View {
        if item.isSpecial {
            SpecialRow(item: item)
        } else {
            RegularRow(item: item)
        }
    }
}
```

Use concrete view structure. Type erasure should be rare and justified.

## Avoid Creating View Models Inside Rows

Bad:

```swift
ForEach(items) { item in
    ItemRow(viewModel: ItemRowViewModel(item: item))
}
```

This creates new objects as the list renders and can reset row state.

Better:

- pass values directly for simple rows
- create row models in the parent view model if row models are truly needed
- key row models by stable item ID

```swift
ForEach(viewModel.rows) { row in
    ItemRow(row: row)
}
```

## Avoid Network Calls From Rows

Bad:

```swift
struct UserRow: View {
    let userID: User.ID

    var body: some View {
        Text("User")
            .task {
                await loadUser(userID)
            }
    }
}
```

If this row appears in a large list, the app can trigger a flood of requests as cells appear and disappear.

Prefer:

- parent-level loading
- batched fetches
- cached data
- pagination owned by the screen view model

Use row-level `.task(id:)` only for small, bounded, cancellable work that is genuinely row-specific.

## Pagination Pattern

Use a sentinel near the end of a lazy list:

```swift
struct ProjectFeedScreen: View {
    @State private var viewModel = ProjectFeedViewModel()

    var body: some View {
        ScrollView {
            LazyVStack(spacing: 12) {
                ForEach(viewModel.items) { item in
                    ProjectCard(item: item)
                        .onAppear {
                            viewModel.loadMoreIfNeeded(currentItemID: item.id)
                        }
                }

                if viewModel.isLoadingMore {
                    ProgressView()
                        .padding()
                }
            }
            .padding()
        }
        .task {
            await viewModel.loadInitialPage()
        }
    }
}
```

The view model must guard duplicate loads:

```swift
@Observable
@MainActor
final class ProjectFeedViewModel {
    var items: [ProjectSummary] = []
    var isLoadingMore = false
    var canLoadMore = true

    func loadMoreIfNeeded(currentItemID: ProjectSummary.ID) {
        guard currentItemID == items.last?.id else { return }
        guard !isLoadingMore, canLoadMore else { return }

        Task {
            await loadNextPage()
        }
    }

    func loadNextPage() async {
        guard !isLoadingMore, canLoadMore else { return }
        isLoadingMore = true
        defer { isLoadingMore = false }

        // Fetch next page and append.
    }
}
```

Do not let every row trigger pagination.

## Pull to Refresh

For `List`:

```swift
List(viewModel.items) { item in
    ItemRow(item: item)
}
.refreshable {
    await viewModel.refresh()
}
```

For custom scroll feeds, `.refreshable` can still be attached to the scroll container:

```swift
ScrollView {
    LazyVStack {
        ForEach(viewModel.items) { item in
            ItemCard(item: item)
        }
    }
}
.refreshable {
    await viewModel.refresh()
}
```

The refresh method should cancel or ignore duplicate refreshes.

## Empty, Loading, and Error States

Do not hide these states inside the list rows. Make them screen states:

```swift
Group {
    if viewModel.isLoadingInitial {
        ProgressView()
    } else if viewModel.items.isEmpty {
        ContentUnavailableView("No Items", systemImage: "tray")
    } else {
        itemList
    }
}
```

This keeps the list focused on listing data.

## Horizontal Carousel Anti-Patterns

Avoid:

```swift
ScrollView(.horizontal) {
    HStack {
        ForEach(items) { item in
            Card(item: item)
        }
    }
}
```

Use:

```swift
ScrollView(.horizontal) {
    LazyHStack(spacing: 12) {
        ForEach(items) { item in
            Card(item: item)
                .frame(width: 280)
        }
    }
    .padding(.horizontal, 16)
    .scrollTargetLayout()
}
.scrollIndicators(.hidden)
.scrollTargetBehavior(.viewAligned)
```

Also avoid:

- nested horizontal carousels where a vertical list row owns its own network loading
- variable card widths unless the design requires them
- full image decoding inside every card
- reading global state in every card
- using `id: \.self` for mutable card data

## Vertical Feed Anti-Patterns

Avoid:

```swift
ScrollView {
    VStack {
        ForEach(viewModel.items.indices, id: \.self) { index in
            FeedCard(item: viewModel.items[index])
        }
    }
}
```

Use:

```swift
ScrollView {
    LazyVStack(spacing: 12) {
        ForEach(viewModel.items) { item in
            FeedCard(item: item)
        }
    }
}
```

The bad version combines eager layout, index identity, and direct array indexing.

## Scroll Position

Use `ScrollViewReader` for commands like scroll-to-top or scroll-to-item:

```swift
ScrollViewReader { proxy in
    ScrollView {
        LazyVStack {
            ForEach(messages) { message in
                MessageRow(message: message)
                    .id(message.id)
            }
        }
    }
    .onChange(of: focusedMessageID) { _, id in
        if let id {
            withAnimation {
                proxy.scrollTo(id, anchor: .center)
            }
        }
    }
}
```

Use modern scroll-position APIs when the screen needs state-bound scroll position. Keep updates meaningful; do not store raw scroll offset every frame.

## Scroll Effects

Use iOS 17+ scroll APIs for visual effects:

```swift
ItemCard(item: item)
    .scrollTransition { content, phase in
        content
            .opacity(phase.isIdentity ? 1 : 0.75)
            .scaleEffect(phase.isIdentity ? 1 : 0.96)
    }
```

Use effects sparingly. If every row animates heavily during scroll, the feed will feel less responsive.

## Measuring Scroll Offset

Do not write raw offset into state every frame:

```swift
@State private var offset: CGFloat = 0
```

Prefer threshold state:

```swift
.onGeometryChange(for: Bool.self) { proxy in
    proxy.frame(in: .scrollView).minY < -80
} action: { shouldCollapse in
    if isHeaderCollapsed != shouldCollapse {
        isHeaderCollapsed = shouldCollapse
    }
}
```

The UI usually needs a state such as “collapsed,” not the exact pixel offset.

## Swipe Actions and Edit Mode

Use `List` for native row interactions:

```swift
List {
    ForEach(items) { item in
        ItemRow(item: item)
            .swipeActions {
                Button("Delete", role: .destructive) {
                    delete(item)
                }
            }
    }
    .onDelete(perform: delete)
}
```

If you rebuild all native list behaviors in custom scroll views, you own all interaction, state, and edge cases.

## Sectioned Data

Shape sections before rendering:

```swift
struct ItemSection: Identifiable {
    let id: String
    let title: String
    let items: [Item]
}

List {
    ForEach(viewModel.sections) { section in
        Section(section.title) {
            ForEach(section.items) { item in
                ItemRow(item: item)
            }
        }
    }
}
```

Do not group data inline inside the `List` builder.

## Search Results

Keep search filtering outside row rendering:

```swift
List(viewModel.filteredItems) { item in
    ItemRow(item: item)
}
.searchable(text: $viewModel.query)
```

The view model should debounce server-backed search if needed. Do not start a request per row.

## Large Dataset Rules

For large lists:

- use server-side or database-side paging when possible
- fetch only required fields when possible
- do not render hidden data
- keep row images downsampled and cached
- keep row models small
- avoid expensive row overlays and shadows
- avoid nested `GeometryReader`
- avoid nested scroll views unless the design truly needs them
- prefer stable placeholders to layout-changing loading states

## Nested Lists and Carousels

Vertical feeds with horizontal carousels are valid, but each carousel must be cheap:

```swift
ForEach(viewModel.sections) { section in
    SectionCarousel(
        title: section.title,
        items: section.items,
        openItem: viewModel.openItem
    )
}
```

Each carousel should receive ready-to-render items. It should not fetch, sort, filter, or build a service.

## Anti-Pattern Catalog

Flag these during review:

- `ForEach(items.indices, id: \.self)` for dynamic data.
- `ForEach(0..<items.count)` for dynamic data.
- `id: \.self` on mutable models.
- IDs based on display strings.
- `UUID()` generated inside `body`.
- `VStack` inside `ScrollView` for dynamic large data.
- `HStack` inside horizontal `ScrollView` for dynamic large data.
- `AnyView` in rows.
- Inline `filter`, `sorted`, `map`, grouping, or deduplication in `ForEach`.
- Image decoding in row bodies.
- Creating formatters in rows.
- Creating services, repositories, or view models in rows.
- Row-level network loading for unbounded lists.
- Broad environment model reads in every row.
- Storing raw scroll offset in state.
- Updating state on every scroll pixel.
- Variable sibling view counts per `ForEach` element.
- Multiple unrelated `.id(...)` modifiers to “force refresh.”
- Using `.id(UUID())` to fix stale UI.
- Hiding list bugs by wrapping everything in `AnyView`.
- Using `onAppear` pagination without duplicate-load guards.
- Caching filtered arrays without invalidation.
- Nesting scroll views without clear gesture behavior.
- Applying heavy shadows/materials/overlays to hundreds of rows.
- Making the row own navigation and data loading and business rules at once.

## Preferred Review Comment Language

Use direct, useful review language:

- “This list uses index identity; deleting or reordering items can produce wrong rows. Use stable model IDs.”
- “This `VStack` eagerly builds every row. Use `LazyVStack` for dynamic scroll content.”
- “This row reads the whole app model. Pass the specific values it needs.”
- “This filter runs during every body evaluation. Move data shaping to the view model.”
- “This row creates an object while rendering. Create it at the ownership boundary and pass it in.”
- “This pagination trigger can start duplicate requests. Add loading and end-of-data guards.”

## Final Checklist for Listing Screens

- [ ] Correct container: `List`, `LazyVStack`, `LazyHStack`, grid, or table.
- [ ] Stable model identity.
- [ ] No index-based dynamic identity.
- [ ] No generated IDs in `body`.
- [ ] Data is shaped before rendering.
- [ ] Rows receive narrow inputs.
- [ ] Rows do not create services or view models.
- [ ] Rows do not decode large images.
- [ ] No `AnyView` row type erasure.
- [ ] Pagination is guarded.
- [ ] Refresh is guarded.
- [ ] Empty/loading/error states are screen-level states.
- [ ] Scroll offset state is thresholded.
- [ ] Horizontal carousels use `LazyHStack`.
- [ ] Vertical custom feeds use `LazyVStack`.
- [ ] Standard interactions use `List` when appropriate.

