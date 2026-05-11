# Focus, Keyboard, Search, and Input

Focus and input handling should be state-driven and local. Use focus state for keyboard movement, submit behavior for form actions, and searchable APIs for search surfaces.

## FocusState

Use `@FocusState` to control field focus:

```swift
enum Field: Hashable {
    case email
    case password
}

@FocusState private var focusedField: Field?

TextField("Email", text: $email)
    .focused($focusedField, equals: .email)

SecureField("Password", text: $password)
    .focused($focusedField, equals: .password)
```

Keep focus state private to the view that owns the input flow.

## Submit Flow

Use `onSubmit` to advance or commit:

```swift
TextField("Email", text: $email)
    .focused($focusedField, equals: .email)
    .submitLabel(.next)
    .onSubmit {
        focusedField = .password
    }

SecureField("Password", text: $password)
    .focused($focusedField, equals: .password)
    .submitLabel(.go)
    .onSubmit(signIn)
```

Avoid old `TextField` commit callbacks.

## Default Focus

Set initial focus from a lifecycle task when it improves the flow:

```swift
.task {
    focusedField = .email
}
```

Do not repeatedly set focus during every redraw.

## Nested Fields

When a child view owns the actual `TextField`, pass a focus binding only if the parent needs to coordinate fields. Otherwise let the child own local focus.

```swift
struct NameFields: View {
    @Binding var firstName: String
    @Binding var lastName: String
    @FocusState.Binding var focusedField: Field?

    var body: some View {
        TextField("First Name", text: $firstName)
            .focused($focusedField, equals: .firstName)
    }
}
```

Avoid global focus models for simple forms.

## Keyboard Types

Use keyboard type as a hint, not as validation:

```swift
TextField("Amount", value: $amount, format: .number)
    .keyboardType(.decimalPad)
```

For numeric input, bind to a numeric value with a format initializer whenever possible.

## Search

Use `searchable` for search surfaces:

```swift
@State private var query = ""

List(filteredItems) { item in
    ItemRow(item: item)
}
.searchable(text: $query, prompt: "Search")
```

For user-entered search, use `localizedStandardContains()`:

```swift
var filteredItems: [Item] {
    guard !query.isEmpty else { return items }
    return items.filter { $0.title.localizedStandardContains(query) }
}
```

## Search Focus

Control search focus when the flow needs it:

```swift
@FocusState private var isSearchFocused: Bool

ContentView()
    .searchable(text: $query)
    .searchFocused($isSearchFocused)
```

For iOS 26+, use minimizable search toolbar behavior when the design needs compact search:

```swift
.searchToolbarBehavior(.minimizable)
```

## Text Input Modifiers

- Use `textInputAutocapitalization(_:)`.
- Use `autocorrectionDisabled(_:)`.
- Use `submitLabel(_:)`.
- Use `textContentType(_:)` for system-assisted entry when appropriate.
- Use `keyboardType(_:)` as a keyboard hint.

## Avoid Focus Loops

Do not write focus state in response to every focus-related change unless the transition is meaningful. Focus writes can create frustrating keyboard jumps.

## Review Checklist

- [ ] Focus state is private and local.
- [ ] Submit flow is explicit.
- [ ] Numeric text entry uses value binding and format where practical.
- [ ] Search filtering uses localized standard matching.
- [ ] Focus is not repeatedly reset during redraws.
- [ ] Keyboard type is not treated as validation.

