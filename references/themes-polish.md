# Polishing and Customization

Source: Healy (2026), Chapter 8 "Refine Your Plots"

## Color palettes

### Three palette types

| Type | Use for | Example |
|------|---------|---------|
| **Qualitative** | Unordered categories (countries, groups) | Distinct hues, equal visual weight |
| **Sequential** | Magnitude (low to high) | Single-hue gradient |
| **Diverging** | Centered data with meaningful midpoint | Two-hue gradient from extremes |

Match palette structure to variable structure. ggplot treats `<dbl>` and `<int>` as continuous; `<fct>` and `<ord>` as categorical.

### Specifying colors

```r
# Named palettes (ColorBrewer built-in)
scale_color_discrete(palette = "Set2")
scale_color_discrete(palette = "Dark2")
scale_fill_brewer(palette = "Blues")

# Manual colors
scale_color_manual(values = c("red", "cornflowerblue", "#CC55DD"))

# Viridis (perceptually uniform, colorblind-safe)
scale_fill_viridis_d(option = "plasma")
scale_fill_viridis_c(option = "viridis")
```

### Palette resources
- **ColorBrewer**: Built into ggplot via palette argument
- **colorspace package**: Wide range of balanced palettes
- **paletteer package**: Comprehensive collection from many sources

### Considerations
- Test for colorblind accessibility
- Consider display conditions (projection, print vs. screen)
- Sequential palettes for ordered data; qualitative for unordered
- Never pick colors ad hoc

## Layer, highlight, repeat

Three-part strategy for building complex, presentation-quality visualizations:

### 1. Layer: base with all observations

```r
p0 <- data |>
  ggplot(aes(x = var1, y = var2)) +
  geom_point(alpha = 0.15, color = "gray50")
```

### 2. Highlight: add subset with contrasting appearance

```r
p1 <- p0 +
  geom_point(
    data = data |> filter(condition),
    aes(color = category)
  )
```

### 3. Repeat: apply consistent design elements

```r
p2 <- p1 +
  geom_text_repel(
    data = data |> filter(special_condition),
    aes(label = name)
  )
```

**Layer order matters**: Draw text layers before point layers so connectors appear underneath points.

### ggrepel fine-tuning

Arguments for `geom_text_repel()`:
- `force`: Repulsion force between labels
- `force_pull`: Attraction force toward data point
- `max.overlaps`: Maximum allowed overlaps before dropping labels
- `min.segment.length`: Minimum segment length to draw connector

### Presentation use

Build arguments progressively using consistent base layers. Reveal data aspects sequentially in talks.

## Themes

### Built-in themes

```r
theme_gray()       # Default (gray background)
theme_bw()         # Black and white
theme_minimal()    # Minimal
theme_dark()       # Dark background
theme_linedraw()   # Black lines
theme_void()       # Nothing (for maps)
```

### Set global theme

```r
theme_set(theme_bw())   # All subsequent plots use this
```

### ggthemes package

Additional themes: Economist, Wall Street Journal, Tufte, etc.

### Ink-and-paper adjustments (ggplot 4+)

```r
theme_bw(
  paper = "#073b4c",       # Background color
  ink = "#d9e5ed",         # Text, axes, points
  accent = "#F0E442"       # Smoother lines, special elements
)
```

### Customizing specific elements

**Element hierarchy** (dot-separated, from general to specific):
- `axis`, `legend`, `panel`, `plot`, `strip`
- More specific: `axis.title.x`, `axis.text.y`, `strip.text`

**Element types**:
- `element_text(size, face, color, angle, hjust, vjust)`
- `element_line(color, linewidth, linetype)`
- `element_rect(fill, color, linewidth)`
- `element_blank()` -- removes element entirely

```r
theme(
  plot.title = element_text(size = 16, face = "bold"),
  axis.text.x = element_text(angle = 45, hjust = 1),
  panel.grid.minor = element_blank(),
  legend.position = "bottom",
  strip.text = element_text(face = "italic")
)
```

### Building custom theme functions

```r
theme_custom <- function(base_size = 12, base_family = "serif") {
  theme_minimal(base_size = base_size, base_family = base_family) +
    theme(
      plot.title = element_text(face = "bold"),
      panel.grid.minor = element_blank(),
      legend.position = "bottom"
    )
}
```

## Scale and axis formatting

```r
# Date axes
scale_x_date(date_breaks = "1 month", date_labels = "%b")

# Large numbers
scale_x_continuous(
  breaks = 10^(2:7),
  labels = scales::label_number(scale_cut = cut_short_scale())
)

# Capped axes
guides(
  x = guide_axis(cap = "both"),
  y = guide_axis(cap = "both", minor.ticks = TRUE)
)

# Expand/contract axis padding
scale_x_continuous(expand = expansion(add = c(0, 2)))
scale_y_continuous(expand = expansion(mult = c(0, 0.05)))
```

## Point shapes

- Shapes 0-20: `color` only (outlines)
- Shapes 21-25: both `color` (border) and `fill` (interior)

```r
geom_point(shape = 21, color = "black", fill = "steelblue", size = 3)
```

## Saying no to pie charts

Problems:
- Harder to estimate wedge values than bar heights
- Ordering obscures sequences
- Comparing multiple pies requires back-and-forth eye movement
- Often requires displaying all numerical values, defeating visualization purpose

Better alternatives:
- Faceted bar charts
- Stacked horizontal bars
- Cleveland dot plots

```r
# Stacked horizontal bars (replace pie chart)
ggplot(data, aes(x = pct, y = category, fill = group)) +
  geom_col(position = position_stack(reverse = TRUE)) +
  scale_x_continuous(labels = scales::label_percent(), expand = expansion(0))
```

## Legend customization

```r
guides(
  fill = guide_legend(
    reverse = TRUE,
    title.position = "top",
    label.position = "bottom",
    keywidth = 3,
    nrow = 1
  )
)
```

## Patchwork for multi-panel layouts

```r
library(patchwork)

# Side by side
p1 + p2

# Stacked
p1 / p2

# Complex layouts
(p1 + p2) / p3

# Layout control
(p1 + p2 + p3) +
  plot_layout(
    axes = "collect",
    heights = c(5, 2, 2),
    nrow = 2
  )

# Annotations
(p1 + p2) +
  plot_annotation(
    title = "Main Title",
    subtitle = "Subtitle",
    caption = "Caption"
  )

# Apply theme to all sub-plots with &
(p1 + p2 + p3) &
  theme(legend.position = "bottom")
```

The `&` operator applies changes to all sub-plots. The `+` operator applies only to the immediately preceding plot.

Patchwork automatically aligns axes between plots for comparison.

## Workhorses, show ponies, unicorns

- **Workhorses**: Standard plots (dot plots, histograms, scatterplots, columns). Direct, functional.
- **Show ponies**: Carefully refined workhorses with polish. Focus here.
- **Unicorns**: One-off creative visualizations. Rare, difficult to produce.

Most visualization work is upgrading workhorses to show ponies through deliberate color, typography, annotation, and layout choices.

## Conditional highlighting with flag variables

```r
data |>
  mutate(flag = month == "October" & day == 31) |>
  ggplot(aes(x = date, y = value, fill = flag)) +
  geom_col() +
  scale_fill_manual(values = c(NA, "firebrick")) +  # NA = transparent
  guides(fill = "none")
```

## Polar coordinates (use cautiously)

```r
coord_radial(
  expand = FALSE,
  rlim = c(0, 4.25),
  inner.radius = 0.25,
  r.axis.inside = TRUE
)
```

People judge relative lengths better than relative angles. Use polar coordinates for seasonal/cyclical data only when the circular structure is meaningful.

## Aspect ratio

- Narrow, tall plots emphasize trends
- Wide plots minimize perceived change
- Be deliberate about figure dimensions
- Use `ggsave()` with explicit `height` and `width`

## Relative sizing

```r
size = rel(4)              # Relative to theme base size
linewidth = rel(3)
unit(0, "pt")              # Explicit units
```

## Label wrapping

```r
# Wrap long axis labels
scale_y_discrete(labels = \(x) str_wrap(x, width = 13))

# Custom labeller for facets
facet_wrap(~ var, labeller = as_labeller(named_vector))
```
