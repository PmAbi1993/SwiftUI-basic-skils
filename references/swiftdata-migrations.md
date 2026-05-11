# SwiftData, MVVM, and Migrations

SwiftData integrates tightly with SwiftUI, but a company app still needs clear boundaries. Use SwiftData directly in simple views when that keeps the feature smaller; use repositories or view models when persistence decisions, testing, migrations, or local caching behavior becomes meaningful.

## Core SwiftData Pieces

| Type or macro | Role |
|---|---|
| `@Model` | Makes a class persistable and observable by SwiftData |
| `ModelContainer` | Owns schema, storage configuration, and migration plan |
| `ModelContext` | Fetches, inserts, deletes, tracks changes, and saves |
| `@Query` | SwiftUI property wrapper for live fetched model collections |
| `FetchDescriptor` | Reusable fetch configuration for predicates, sorting, limits, offsets, and prefetching |
| `#Predicate` | Type-checked predicate for filtering |
| `VersionedSchema` | Snapshot of model types for one schema version |
| `SchemaMigrationPlan` | Ordered schemas and migration stages |
| `MigrationStage` | One transition between schema versions |

## Model Container Setup

For simple apps:

```swift
RootScreen()
    .modelContainer(for: [Project.self, TaskItem.self])
```

For versioned apps, create the container with a migration plan:

```swift
struct AppContainer {
    let modelContainer: ModelContainer

    init() {
        do {
            let schema = Schema(versionedSchema: ProjectSchemaV2.self)
            modelContainer = try ModelContainer(
                for: schema,
                migrationPlan: ProjectMigrationPlan.self
            )
        } catch {
            fatalError("Could not create model container: \(error)")
        }
    }
}

RootScreen()
    .modelContainer(appContainer.modelContainer)
```

Use a project-standard startup error strategy if crashing is not acceptable.

## Using `@Query` in MVVM

`@Query` belongs in SwiftUI views, not view models. It is a view property wrapper that keeps fetched results in sync with the underlying store.

Good:

```swift
struct ProjectListScreen: View {
    @Query(sort: \Project.updatedAt, order: .reverse)
    private var projects: [Project]

    var body: some View {
        List(projects) { project in
            ProjectRow(project: project)
        }
    }
}
```

Use this for simple list screens where the view only displays and navigates.

## Passing SwiftData Models to Views

Passing `@Model` instances into rows and detail views is normal:

```swift
struct ProjectRow: View {
    let project: Project

    var body: some View {
        VStack(alignment: .leading) {
            Text(project.name)
            Text(project.updatedAt, format: .dateTime.month().day())
        }
    }
}
```

Do not wrap every SwiftData model in a DTO just to satisfy MVVM. Add a view model when behavior or testability requires it.

## Editing SwiftData Models

For direct editing where live mutation is acceptable:

```swift
struct EditProjectScreen: View {
    @Bindable var project: Project

    var body: some View {
        Form {
            TextField("Name", text: $project.name)
        }
    }
}
```

For save/cancel workflows, use a draft so cancel does not mutate the persisted model:

```swift
struct ProjectDraft: Equatable {
    var name: String
    var notes: String

    init(project: Project) {
        name = project.name
        notes = project.notes
    }
}
```

```swift
@Observable
@MainActor
final class EditProjectViewModel {
    var draft: ProjectDraft

    init(project: Project) {
        draft = ProjectDraft(project: project)
    }

    func apply(to project: Project) {
        project.name = draft.name
        project.notes = draft.notes
        project.updatedAt = .now
    }
}
```

The view can apply and save through the context:

```swift
@Environment(\.modelContext) private var modelContext

Button("Save") {
    viewModel.apply(to: project)
    try? modelContext.save()
}
```

Use proper error handling in production code instead of `try?`.

## ModelContext in Views

Access the context from the environment for local insert/delete/save actions:

```swift
@Environment(\.modelContext) private var modelContext

func addProject() {
    let project = Project(name: "New Project")
    modelContext.insert(project)
    try? modelContext.save()
}

func delete(_ project: Project) {
    modelContext.delete(project)
    try? modelContext.save()
}
```

Keep inline button closures small; call named methods or view model/repository methods.

## ModelContext in ViewModels

Prefer not to store `ModelContext` casually in long-lived view models. It is tied to a container/context lifecycle and is easiest to reason about at the view or repository boundary.

Acceptable patterns:

1. Pass `ModelContext` into a method:

```swift
@MainActor
func save(project: Project, in context: ModelContext) throws {
    apply(to: project)
    try context.save()
}
```

2. Inject a repository that owns persistence operations:

```swift
protocol ProjectRepository {
    func projects(matching query: String) throws -> [Project]
    func createProject(name: String) throws
    func delete(_ project: Project) throws
}
```

3. Use a dedicated data actor/service for non-view persistence work when the app needs background or serialized data operations.

## Repository Pattern

Use a repository when:

- multiple screens use the same fetch/save logic
- tests need fake persistence
- fetch descriptors are complex
- data comes from SwiftData plus a server
- writes require validation, deduplication, or conflict handling

Example:

```swift
@MainActor
final class SwiftDataProjectRepository: ProjectRepository {
    private let context: ModelContext

    init(context: ModelContext) {
        self.context = context
    }

    func projects(matching query: String) throws -> [Project] {
        let descriptor = FetchDescriptor<Project>(
            predicate: query.isEmpty ? nil : #Predicate {
                $0.name.contains(query)
            },
            sortBy: [
                SortDescriptor(\.updatedAt, order: .reverse)
            ]
        )

        return try context.fetch(descriptor)
    }

    func createProject(name: String) throws {
        context.insert(Project(name: name))
        try context.save()
    }

    func delete(_ project: Project) throws {
        context.delete(project)
        try context.save()
    }
}
```

Keep repositories focused on persistence, not UI presentation.

`#Predicate` supports a constrained expression set. If the company search behavior requires fully localized matching that does not compile inside `#Predicate`, fetch a bounded candidate set with SwiftData and apply `localizedStandardContains()` in memory, or store a normalized search field.

## FetchDescriptor Guidelines

Use `FetchDescriptor` outside views or for reusable query definitions:

```swift
extension FetchDescriptor where T == Project {
    static func recent(limit: Int) -> FetchDescriptor<Project> {
        var descriptor = FetchDescriptor<Project>(
            sortBy: [SortDescriptor(\.updatedAt, order: .reverse)]
        )
        descriptor.fetchLimit = limit
        return descriptor
    }
}
```

Use:

- `predicate` to filter
- `sortBy` to order
- `fetchLimit` for bounded lists
- `fetchOffset` for paging-like flows
- `includePendingChanges` when unsaved local changes should participate
- `relationshipKeyPathsForPrefetching` when relationship access would otherwise cause repeated loads
- `propertiesToFetch` when only a subset is needed

## Fetching Counts and Identifiers

Use `fetchCount` when only the count is needed:

```swift
let descriptor = FetchDescriptor<Project>(
    predicate: #Predicate { $0.isArchived == false }
)
let count = try modelContext.fetchCount(descriptor)
```

This is not a live query by itself. The UI must have another update trigger if it needs to refresh automatically.

Use persistent identifiers for stable references across navigation or async boundaries when passing full model instances would be risky:

```swift
let id = project.persistentModelID
```

Resolve through the context when needed.

## Relationship Guidelines

- Use `@Relationship(deleteRule:)` deliberately.
- Prefer clear inverse relationships.
- Be careful with cascade deletes in company data.
- For static data categories, prefer `Codable` enums when they fit.
- Avoid very large relationship arrays in hot views; fetch what the screen needs.

## MVVM Boundaries With SwiftData

Use this decision table:

| Situation | Recommended approach |
|---|---|
| Simple live list | `@Query` in the view, rows receive models |
| Simple detail display | Pass the `@Model` instance into the view |
| Direct edit form | `@Bindable` model if live editing is acceptable |
| Save/cancel edit form | Draft in view model, apply to model on save |
| Complex fetch/filter | Repository using `FetchDescriptor` |
| Cross-screen persistence rules | Repository or use-case service |
| Cache from server | Repository/service coordinates server data and SwiftData storage |
| Unit-test-heavy behavior | View model depends on protocol, not raw context |
| Migration-sensitive data | Versioned schemas and explicit migration plan |

## Migration Strategy

For long-lived company apps, start versioning schemas before the model becomes complicated.

Basic structure:

```swift
enum AppSchemaV1: VersionedSchema {
    static var versionIdentifier = Schema.Version(1, 0, 0)

    static var models: [any PersistentModel.Type] {
        [Project.self, TaskItem.self]
    }

    @Model
    final class Project {
        var name: String
        var createdAt: Date

        init(name: String, createdAt: Date = .now) {
            self.name = name
            self.createdAt = createdAt
        }
    }
}
```

Create a new schema version when persisted model shape changes:

```swift
enum AppSchemaV2: VersionedSchema {
    static var versionIdentifier = Schema.Version(2, 0, 0)

    static var models: [any PersistentModel.Type] {
        [Project.self, TaskItem.self]
    }

    @Model
    final class Project {
        var name: String
        var createdAt: Date
        var updatedAt: Date

        init(name: String, createdAt: Date = .now, updatedAt: Date = .now) {
            self.name = name
            self.createdAt = createdAt
            self.updatedAt = updatedAt
        }
    }
}
```

Migration plan:

```swift
enum AppMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [AppSchemaV1.self, AppSchemaV2.self]
    }

    static let migrateV1ToV2 = MigrationStage.lightweight(
        fromVersion: AppSchemaV1.self,
        toVersion: AppSchemaV2.self
    )

    static var stages: [MigrationStage] {
        [migrateV1ToV2]
    }
}
```

Wire the plan into `ModelContainer` at app startup.

## Lightweight Migration

Lightweight migration is appropriate for changes SwiftData can infer, such as:

- adding optional properties
- adding properties with safe default values
- simple compatible relationship additions
- renaming with `@Attribute(originalName:)`
- compatible schema additions that do not require data transformation

Use `@Attribute(originalName:)` for renamed fields:

```swift
@Attribute(originalName: "title")
var name: String
```

Do not rely on lightweight migration when SwiftData must invent data or resolve conflicts.

## Custom Migration

Use custom migration when:

- adding a non-optional property that needs derived values
- splitting one property into several
- merging several properties
- changing types
- deduplicating data before adding uniqueness
- changing relationship structure
- backfilling values from related models
- cleaning invalid historical data

Shape:

```swift
static let migrateV1ToV2 = MigrationStage.custom(
    fromVersion: AppSchemaV1.self,
    toVersion: AppSchemaV2.self,
    willMigrate: { context in
        let descriptor = FetchDescriptor<AppSchemaV1.Project>()
        let oldProjects = try context.fetch(descriptor)

        for project in oldProjects {
            // Prepare old data or remove invalid records.
        }

        try context.save()
    },
    didMigrate: { context in
        let descriptor = FetchDescriptor<AppSchemaV2.Project>()
        let projects = try context.fetch(descriptor)

        for project in projects {
            project.updatedAt = project.createdAt
        }

        try context.save()
    }
)
```

Use `willMigrate` for work against the old schema and `didMigrate` for work against the new schema.

## Migration Safety Rules

- Keep every schema version’s model definitions available.
- Include all model types required by that schema in `models`.
- Test migration from every released schema version to the newest version.
- Do not test only fresh installs.
- Make migrations idempotent where practical.
- Avoid deleting old data unless the migration explicitly owns that decision.
- Keep migration code deterministic; do not call network services.
- Back up or export valuable data before risky migration work during development.
## Common Migration Mistakes

- Adding a required property without a default or custom backfill.
- Renaming a property without `originalName`.
- Changing a type and expecting lightweight migration to infer intent.
- Forgetting to add a model type to a versioned schema’s `models`.
- Initializing `ModelContainer` somewhere else without the migration plan.
- Using one schema for debug and another for release without testing upgrade paths.
- Mutating view code to compensate for bad persisted data instead of migrating it.

## Testing SwiftData Migrations

For each migration:

1. Install or create a store using the old app/schema.
2. Populate realistic data, including edge cases.
3. Launch the new app with the migration plan.
4. Assert the container opens successfully.
5. Fetch migrated models and verify required fields, relationships, counts, ordering, and defaults.
6. Test save/edit/delete after migration.
7. Repeat for every released starting schema.

Keep sample old stores for regression testing when the app is data-critical.

## Review Checklist

- [ ] Simple live lists use `@Query` rather than unnecessary manual fetch plumbing.
- [ ] Complex persistence behavior lives in a repository or service.
- [ ] View models do not casually store long-lived contexts.
- [ ] Save/cancel edit screens use drafts.
- [ ] Fetch descriptors have predicates, sort order, and limits where needed.
- [ ] Migrations are versioned before risky model changes.
- [ ] Lightweight vs custom migration choice is justified.
- [ ] Required new data is backfilled.
- [ ] Migration tests cover existing stores, not just fresh installs.

## Official References

- SwiftData overview: https://developer.apple.com/documentation/SwiftData
- ModelContainer: https://developer.apple.com/documentation/swiftdata/modelcontainer
- ModelContext fetch: https://developer.apple.com/documentation/SwiftData/ModelContext/fetch%28_%3A%29
- FetchDescriptor: https://developer.apple.com/documentation/swiftdata/fetchdescriptor
- Query: https://developer.apple.com/documentation/SwiftData/Query
- SwiftUI modelContext environment: https://developer.apple.com/documentation/swiftui/environmentvalues/modelcontext
- SchemaMigrationPlan: https://developer.apple.com/documentation/swiftdata/schemamigrationplan
- VersionedSchema: https://developer.apple.com/documentation/swiftdata/versionedschema
