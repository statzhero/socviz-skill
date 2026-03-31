---
name: socviz
description: |
  Data visualization with ggplot2 based on Kieran Healy's "Data Visualization: A Practical Introduction" (2nd ed., 2026). Use when creating, reviewing, or improving ggplot2 visualizations, choosing appropriate chart types, applying perceptual principles, faceting and grouping data, labeling and annotating plots, visualizing model results, drawing maps, or polishing figures for publication. Covers the grammar of graphics workflow, visual perception, color palettes, layered plot construction, and theme customization.
license: CC-BY-NC-ND
metadata:
  source: https://socviz.co
  author: Kieran Healy
  edition: 2nd (2026)
  publisher: Princeton University Press
allowed-tools: Read, Edit, Write, Grep, Glob, mcp__r-btw__*
---

# Data Visualization with ggplot2

Based on: *Data Visualization: A Practical Introduction* (2nd ed.)
Author: Kieran Healy (2026). Princeton University Press.
Free online: https://socviz.co

## Core philosophy

Visualization is a tool for understanding data, not decoration. Every graph encodes data through visual channels (position, length, color, shape) that the human perceptual system decodes with varying accuracy. Good visualization requires understanding both the grammar of graphics and the perceptual system that interprets the output.

## The ggplot2 workflow

Every visualization follows the same five steps:

1. **Tidy data** -- Long format: every variable a column, every observation a row
2. **Mapping** -- `aes()` connects data variables to visual properties (x, y, color, fill, size, shape)
3. **Geom** -- Geometric objects render the mapping (points, lines, bars, etc.)
4. **Scales and coordinates** -- Transform axes, set breaks, choose projections
5. **Labels and guides** -- Titles, legends, annotations, themes

```r
p <- ggplot(data = df,
            mapping = aes(x = var1, y = var2, color = group))
p + geom_point() +
  scale_x_log10(labels = scales::label_comma()) +
  labs(x = "X Label", y = "Y Label", title = "Title")
```

## When to use this skill

- User wants to create or improve a ggplot2 visualization
- User asks which chart type to use for their data
- User needs help with faceting, grouping, labeling, or annotating plots
- User wants to visualize model results (coefficients, predictions, residuals)
- User asks about color palettes, themes, or publication-quality figures
- User wants to draw maps with sf or statebins
- User asks about perceptual principles for data visualization

## Instructions

1. **Classify the request**:
   - Perceptual/design: Which chart type? What color palette? How to avoid misleading? --> Use `references/perception-principles.md`
   - Building a plot: How to facet, group, layer, label? --> Use the relevant reference file
   - Model visualization: Coefficients, predictions, residuals? --> Use `references/model-visualization.md`
   - Maps: Choropleth, cartogram, spatial data? --> Use `references/maps-spatial.md`
   - Polishing: Themes, color, layout, patchwork? --> Use `references/themes-polish.md`

2. **Read the relevant reference files** before answering

3. **Apply these principles** in every response:
   - Match visual encoding to data type (position for quantitative, color hue for categorical)
   - Prefer position on common scale over length, angle, or area
   - Use tidy (long-format) data; reshape before plotting if needed
   - Build plots layer by layer; inspect at each step
   - Distinguish mapping (inside `aes()`) from setting (inside `geom_*()`)
   - Calculate summaries with dplyr before graphing when `geom_bar()`/`stat_count()` gets confusing
   - Use `facet_wrap()` or `facet_grid()` instead of cramming too much into one panel

4. **Follow write-r conventions** for all R code (native pipe `|>`, `.by`, `join_by()`, etc.)

## Available resources

### Reference files (`references/`)

| File | Topic | Covers |
|------|-------|--------|
| `perception-principles.md` | Visual perception and chart choice | Preattentive features, gestalt rules, Cleveland-McGill ranking, color models, honesty/judgment tradeoffs |
| `ggplot-fundamentals.md` | Core ggplot2 patterns | Tidy data, aes() mapping vs setting, layering geoms, scales, saving plots |
| `grouping-faceting.md` | Grouped data and faceting | group aesthetic, facet_wrap/facet_grid, stat transformations, histograms, density plots |
| `labels-annotations.md` | Text, labels, and annotations | Pipes for summarization, text/label geoms, ggrepel, annotate(), scales/guides/themes framework |
| `model-visualization.md` | Visualizing model results | broom (tidy/augment/glance), coefficient plots, residuals, nested models, survey data, marginal effects |
| `maps-spatial.md` | Maps and spatial data | sf objects, geom_sf, choropleths, cartograms, statebins, viridis palettes, MAUP |
| `themes-polish.md` | Polishing and customization | Color palettes, layer-highlight-repeat, theme(), patchwork layouts, workhorses vs show ponies |

### Function reference (via MCP, not bundled)

- **R**: Use `btw_tool_docs_help_page(package_name="ggplot2", topic="<function>")` for live docs
- **R**: Use Context7 with library ID for ggplot2 code examples
- Key packages: `ggplot2`, `scales`, `ggrepel`, `patchwork`, `sf`, `statebins`, `broom`, `colorspace`

## Quick reference: choosing a geom

| Data situation | Recommended geom | Avoid |
|----------------|------------------|-------|
| Two continuous variables | `geom_point()` | |
| Continuous + trend | `geom_point() + geom_smooth()` | |
| One continuous, one categorical | `geom_boxplot()`, `geom_violin()`, `geom_jitter()` | |
| Single continuous distribution | `geom_histogram()`, `geom_density()` | |
| Counts of categories | `geom_bar()` | Pie charts |
| Pre-computed summary | `geom_col()` or `geom_bar(stat = "identity")` | |
| Coefficients with CIs | `geom_pointrange()` | Bar charts for coefficients |
| Time series by group | `geom_line(aes(group = ...))` | |
| Trajectories + trend | Gray `geom_line()` + colored `geom_smooth()` | |
| Spatial/geographic | `geom_sf()` | `geom_polygon()` with map_data (legacy) |
| Two categorical cross-tab | `facet_grid(row ~ col)` | Stacked bars (except bottom category) |
| Many categories | Horizontal layout (categories on y-axis) | Rotated x-axis labels |
| Labeled points | `ggrepel::geom_text_repel()` | `geom_text()` without repulsion |

## Key anti-patterns

| Avoid | Do instead | Why |
|-------|-----------|-----|
| Pie charts | Faceted bar charts or dot plots | Angles decoded poorly (Cleveland-McGill) |
| 3D effects | Flat 2D encodings | Perspective distorts values |
| Stacked bars (comparing middle categories) | Faceted bars or Cleveland dot plots | Only bottom category has stable baseline |
| Mapping a string inside `aes()` | Set property in `geom_*()` directly | `aes(color = "purple")` creates unwanted legend |
| `geom_bar()` for pre-computed data | `geom_col()` | `geom_bar()` counts rows by default |
| Default alphabetical ordering | `reorder(category, value)` | Meaningful order aids comparison |
| Too many channels at once | Facet or simplify | Shape + color + size overwhelms preattentive processing |
| Picking colors ad hoc | Use ColorBrewer, viridis, or colorspace palettes | Perceptually uniform, colorblind-safe |
| `+ geom_point()` on next line | `geom_point() +` at end of line | ggplot `+` must end the line, not begin one |

## Example: complete workflow

```r
library(tidyverse)
library(scales)
library(ggrepel)

# 1. Summarize data
by_continent <- gapminder |>
  summarize(
    mean_life = mean(lifeExp),
    mean_gdp = mean(gdpPercap),
    n_countries = n_distinct(country),
    .by = continent
  )

# 2. Build plot layer by layer
p <- ggplot(by_continent,
            aes(x = mean_gdp, y = mean_life))

p +
  geom_point(aes(size = n_countries), alpha = 0.7) +
  geom_text_repel(aes(label = continent)) +
  scale_x_log10(labels = label_currency()) +
  scale_size_continuous(range = c(3, 10)) +
  labs(
    x = "Mean GDP per Capita",
    y = "Mean Life Expectancy (years)",
    size = "Countries",
    title = "Economic Development and Life Expectancy",
    caption = "Source: Gapminder."
  ) +
  theme_minimal() +
  theme(legend.position = "bottom")
```

## Best practices

- Always build plots incrementally and inspect at each layer
- Use `ggsave()` with explicit dimensions and vector formats (PDF, SVG) for publication
- Organize figures in a `figures/` subdirectory; use `here::here()` for paths
- Use `guides(fill = "none")` to remove redundant legends
- When debugging bizarre output, check the relationship between data structure and aesthetic mappings first
- Prefer faceting over dodging for grouped comparisons
- Place categories on the y-axis when there are many levels
- Order categories by the value being displayed, not alphabetically
- Use `scales::label_*()` functions for axis formatting
