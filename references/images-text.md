# Images and Text

Image and text code often looks harmless while creating memory, formatting, localization, or project-quality problems. Prefer native SwiftUI and Foundation formatting APIs.

## AsyncImage

Use `AsyncImage` for simple remote images:

```swift
AsyncImage(url: avatarURL) { phase in
    switch phase {
    case .empty:
        ProgressView()
    case .success(let image):
        image
            .resizable()
            .scaledToFill()
    case .failure:
        Image(systemName: "person.crop.circle.badge.exclamationmark")
    @unknown default:
        EmptyView()
    }
}
```

Use a custom loader when you need:

- Authentication headers.
- Persistent caching.
- Retry policy.
- Request cancellation beyond basic view lifetime.
- Progressive loading.
- Shared image pipelines.

## Downsampling

Avoid decoding huge images directly into UI memory:

```swift
UIImage(data: data)
```

For large local or downloaded images, downsample to the displayed size before rendering. This is especially important in scrolling grids and feeds.

Keep downsampling in an image service or loader, not in a row body.

## Asset Catalog Images

Use generated asset symbols when the project is configured for them:

```swift
Image(.avatarPlaceholder)
```

Prefer this over stringly typed assets:

```swift
Image("avatarPlaceholder")
```

Generated symbols catch typos at compile time.

## SF Symbols

Use SF Symbols through `Image(systemName:)`, `Label`, and `Button` initializers:

```swift
Label("Favorites", systemImage: "star")
Button("Add", systemImage: "plus", action: add)
```

Use symbol variants and rendering modes intentionally:

```swift
Image(systemName: "star.fill")
    .symbolRenderingMode(.hierarchical)
```

## Text Formatting

Use `FormatStyle`:

```swift
Text(total, format: .currency(code: "USD"))
Text(progress, format: .percent.precision(.fractionLength(0)))
Text(Date.now, format: .dateTime.month().day().year())
Text(fileSize, format: .byteCount(style: .file))
```

Avoid:

```swift
Text(String(format: "%.2f", value))
```

## Foundation Formatting

Prefer modern parsing and formatting:

```swift
let date = try? Date(input, strategy: .iso8601)
```

For person names, use `PersonNameComponents` formatting instead of manual string joins when data is structured.

## Text Composition

Do not concatenate `Text` with `+`:

```swift
Text("Hello").foregroundStyle(.red) + Text("World")
```

Use interpolation with styled `Text` values:

```swift
let sale = Text("Sale").foregroundStyle(.red)
let suffix = Text(" ends today").foregroundStyle(.secondary)

Text("\(sale)\(suffix)")
```

## Verbatim vs Localized Text

Use normal `Text("...")` for user-facing strings that should be localized.

Use `Text(verbatim:)` for:

- IDs.
- Debug values.
- User-generated strings that should not be looked up as localization keys.
- Server-provided labels that are already localized.

```swift
Text(verbatim: user.username)
```

## String Catalogs

When the project uses `Localizable.xcstrings`, prefer symbol keys and generated accessors:

```swift
Text(.settingsTitle)
```

For new user-facing strings in such projects, add the string to the catalog rather than scattering raw literals.

## Search Strings

Use localized standard matching for user-entered search:

```swift
items.filter { item in
    item.title.localizedStandardContains(query)
}
```

Avoid simple `contains()` for user-facing search.

## Grammar Agreement

Use inflection for plural-sensitive copy where supported:

```swift
Text("^[\(people.count) person](inflect: true)")
```

This avoids hand-written plural branches for common cases.

## Rich Text

For iOS 26+, use `TextEditor` with `AttributedString` when rich editing is required:

```swift
@State private var text: AttributedString = "Notes"

TextEditor(text: $text)
```

Use plain `String` text editing for simple fields.

## Review Checklist

- [ ] Large images are not decoded directly in hot UI paths.
- [ ] Repeated image loading is cached or centralized when needed.
- [ ] Generated asset symbols are used where available.
- [ ] User-facing numbers and dates use `FormatStyle`.
- [ ] Search uses localized standard matching.
- [ ] `Text` is not concatenated with `+`.
- [ ] Verbatim text is used for non-localized dynamic strings.
- [ ] String catalog conventions are followed when present.

