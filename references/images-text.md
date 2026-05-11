# Images and Text

Images and text are common places for subtle quality and performance problems. Prefer native SwiftUI and Foundation formatting APIs.

## Images

- Use `AsyncImage` for simple remote images.
- Use a dedicated loader/cache when images need authentication, placeholders, retries, cancellation, or persistent caching.
- Avoid `UIImage(data:)` on large images in UI paths; downsample to the displayed size before rendering.
- Use generated asset symbols when available: `Image(.avatar)` instead of `Image("avatar")`.
- Use SF Symbols with `Label` or `Button` when they represent commands.

## AsyncImage

```swift
AsyncImage(url: avatarURL) { phase in
    switch phase {
    case .success(let image):
        image
            .resizable()
            .scaledToFill()
    case .failure:
        Image(systemName: "person.crop.circle.badge.exclamationmark")
    case .empty:
        ProgressView()
    @unknown default:
        EmptyView()
    }
}
```

## Text Formatting

Use `Text` and `FormatStyle` rather than C-style formatting:

```swift
Text(total, format: .currency(code: "USD"))
Text(progress, format: .percent.precision(.fractionLength(0)))
Text(Date.now, format: .dateTime.month().day().year())
```

Avoid:

```swift
Text(String(format: "%.2f", value))
```

## Search and Matching

For user-entered search, use localized matching:

```swift
items.filter { item in
    item.title.localizedStandardContains(query)
}
```

## Text Composition

Do not concatenate `Text` with `+`. Use interpolation with styled `Text` values:

```swift
let title = Text("Sale").foregroundStyle(.red)
let detail = Text(" ends today").foregroundStyle(.secondary)

Text("\(title)\(detail)")
```

## String Catalogs

When the project uses `Localizable.xcstrings`, prefer adding user-facing strings to the catalog and using generated symbols where configured:

```swift
Text(.settingsTitle)
```

Use `Text(verbatim:)` for strings that should never be localized, such as identifiers or debug labels.

## Names and Dates

- Prefer `PersonNameComponents` formatting for person names.
- Prefer `Date.now` over `Date()` for current time.
- Prefer `Date(myString, strategy: .iso8601)` for ISO date parsing.
- Avoid manual date format strings for user-facing display when a `FormatStyle` works.

