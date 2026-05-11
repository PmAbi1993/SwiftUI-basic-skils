# Swift Charts

Use Swift Charts for native iOS data visualization. Keep chart data identifiable, choose marks that match the question, and move expensive data shaping outside the chart body.

## Setup

Import Charts in files that use chart types:

```swift
import Charts
import SwiftUI
```

Use identifiable data:

```swift
struct SalesPoint: Identifiable {
    let id = UUID()
    let date: Date
    let revenue: Double
    let category: String
}
```

## Basic Chart

```swift
Chart(points) { point in
    LineMark(
        x: .value("Date", point.date),
        y: .value("Revenue", point.revenue)
    )
}
```

Values should be meaningful and typed. Avoid preformatted strings for numeric axes.

## Mark Selection

| Mark | Use for |
|---|---|
| `BarMark` | Comparing categories or discrete values |
| `LineMark` | Continuous trends |
| `AreaMark` | Magnitude over a continuous domain |
| `PointMark` | Individual observations |
| `RectangleMark` | Heat maps or interval blocks |
| `RuleMark` | Thresholds, averages, selected positions |
| `SectorMark` | Part-to-whole values when category count is small |

## Bar Chart

```swift
Chart(summary) { item in
    BarMark(
        x: .value("Category", item.category),
        y: .value("Count", item.count)
    )
    .foregroundStyle(by: .value("Category", item.category))
}
```

Avoid using pie-style charts for many categories. Bar charts are usually easier to compare.

## Line and Area

```swift
Chart(points) { point in
    AreaMark(
        x: .value("Date", point.date),
        y: .value("Revenue", point.revenue)
    )
    .foregroundStyle(.blue.opacity(0.18))

    LineMark(
        x: .value("Date", point.date),
        y: .value("Revenue", point.revenue)
    )
    .foregroundStyle(.blue)
}
```

Layer area and line marks when the area should support, not replace, the trend.

## SectorMark

```swift
Chart(slices) { slice in
    SectorMark(
        angle: .value("Total", slice.value),
        innerRadius: .ratio(0.6),
        angularInset: 1
    )
    .foregroundStyle(by: .value("Category", slice.category))
}
```

Use with a small number of categories. For precise comparison, prefer bars.

## Axes

Customize axes when defaults are noisy:

```swift
.chartXAxis {
    AxisMarks(values: .stride(by: .day)) { value in
        AxisGridLine()
        AxisTick()
        AxisValueLabel(format: .dateTime.weekday(.abbreviated))
    }
}
```

Set domains when a chart needs stable visual comparison across updates:

```swift
.chartYScale(domain: 0...100)
```

Do not clamp data unless the design intentionally hides outliers.

## Styling and Channels

Use visual channels intentionally:

```swift
.foregroundStyle(by: .value("Category", point.category))
.symbol(by: .value("Series", point.series))
.lineStyle(StrokeStyle(lineWidth: 3, lineCap: .round))
```

Avoid encoding too many dimensions at once. If the legend becomes hard to read, split the chart or simplify the question.

## Selection

Use chart selection for interactive inspection:

```swift
@State private var selectedDate: Date?

Chart(points) { point in
    LineMark(
        x: .value("Date", point.date),
        y: .value("Value", point.value)
    )

    if selectedDate == point.date {
        RuleMark(x: .value("Selected", point.date))
    }
}
.chartXSelection(value: $selectedDate)
```

Use range selection when the user needs a span:

```swift
@State private var selectedRange: ClosedRange<Date>?

Chart(points) { point in
    LineMark(
        x: .value("Date", point.date),
        y: .value("Value", point.value)
    )
}
.chartXSelection(range: $selectedRange)
```

## ChartProxy

Use `ChartProxy` for custom interactions not covered by selection APIs. Keep geometry math contained:

```swift
.chartOverlay { proxy in
    GeometryReader { geometry in
        Rectangle()
            .fill(.clear)
            .contentShape(Rectangle())
            .gesture(
                DragGesture()
                    .onChanged { value in
                        let origin = geometry[proxy.plotAreaFrame].origin
                        let x = value.location.x - origin.x
                        selectedDate = proxy.value(atX: x, as: Date.self)
                    }
            )
    }
}
```

Prefer built-in selection APIs when they fit.

## Annotations

Use annotations for selected or exceptional points, not every point:

```swift
PointMark(
    x: .value("Date", point.date),
    y: .value("Value", point.value)
)
.annotation(position: .top) {
    if point.id == selectedPointID {
        Text(point.value, format: .number)
            .padding(6)
            .background(.thinMaterial, in: .rect(cornerRadius: 8))
    }
}
```

## Scrollable Axes

Use scrollable chart axes for large time series:

```swift
.chartScrollableAxes(.horizontal)
.chartXVisibleDomain(length: 60 * 60 * 24 * 7)
```

Keep selected values stable when data updates.

## Chart3D

For iOS 26+ only, consider 3D charts when the extra dimension is genuinely useful:

```swift
if #available(iOS 26, *) {
    Chart3D(points) { point in
        SurfacePlot(
            x: .value("X", point.x),
            y: .value("Y", point.y),
            z: .value("Z", point.z)
        )
    }
}
```

Do not use 3D for decoration. It should answer a data question better than a 2D chart.

## Data Preparation

- Shape and aggregate chart data outside the chart body.
- Keep chart models stable and identifiable.
- Use numeric and date values, not preformatted axis strings.
- Cache expensive aggregation in the model with explicit invalidation.
- Keep formatting in labels, axes, or annotations.

## Review Checklist

- [ ] `import Charts` is present.
- [ ] Data values are typed and identifiable.
- [ ] Mark choice matches the data question.
- [ ] Expensive aggregation is outside the chart builder.
- [ ] Selection uses built-in APIs where possible.
- [ ] Axis domains are intentional.
- [ ] iOS 26 chart APIs are guarded when needed.

