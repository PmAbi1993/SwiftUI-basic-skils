# Layout and Visual Design

Use native iOS layout and controls first. Designs should feel consistent, flexible, and system-aware without custom work where the system already provides a good answer.

## Layout Principles

- Avoid `UIScreen.main.bounds`; use relative layout, container APIs, or `GeometryReader` only when measurement is truly needed.
- Avoid fixed frames unless content is guaranteed to fit.
- Prefer `containerRelativeFrame()`, `visualEffect()`, `onGeometryChange`, or custom `Layout` before reaching for a broad `GeometryReader`.
- Put spacing, padding, radii, timing, and repeated colors in shared constants when the app has a design system.
- Keep interactive controls at least 44x44 points.

## System Components

- Use `Label` for icon-and-text pairs.
- Use `ContentUnavailableView` for empty states.
- Use `LabeledContent` for title/value rows and controls inside `Form`.
- Use system hierarchical styles such as `.secondary` instead of manual opacity when the system style is the intent.
- Use `Form` for settings and data-entry screens when it fits the app.
- Use `Button` styles, `ControlGroup`, `Menu`, `Picker`, `Stepper`, `Slider`, and `Toggle` instead of custom controls when native controls cover the need.

## Text and Typography

- Prefer semantic fonts such as `.body`, `.headline`, and `.title3`.
- Use `bold()` instead of `fontWeight(.bold)`.
- Use custom weights only when there is a clear design reason.
- Avoid overusing tiny text such as `.caption2`.
- Avoid hard-coded font sizes unless the design system requires them.

## Shapes and Styling

- Use `clipShape(.rect(cornerRadius:))` for rounded clipping.
- Use chained `fill` and `stroke` on shapes rather than overlay hacks when targeting modern iOS.
- `RoundedRectangle` uses continuous corners by default; do not specify it unless clarity helps.
- Prefer asset catalog colors for brand colors.

## Forms and Inputs

- Use numeric `TextField` format initializers for numeric input.
- Use `TextField(axis: .vertical)` for lightweight multi-line text entry when placeholder behavior matters.
- Use `TextEditor` when full rich or long-form editing is required.
- Extract validation and save logic out of the input view body.

