# Text, Labels, Annotations, and Scales

Source: Healy (2026), Chapter 5 "Graph Tables, Add Labels, Make Notes"

## Summarize data with pipes before plotting

Better to calculate frequency tables first, then plot the table, rather than embedding calculations within ggplot.

```r
rel_by_region <- gss_sm |>
  summarize(n = n(), .by = c(bigregion, religion)) |>
  mutate(
    freq = n / sum(n),
    pct = round(freq * 100, 0),
    .by = bigregion
  )

# Verify: pct should sum to 100 within each region
rel_by_region |>
  summarize(total = sum(pct), .by = bigregion)
```

Then use `geom_col()` for pre-computed data:

```r
ggplot(rel_by_region, aes(x = bigregion, y = pct, fill = religion)) +
  geom_col(position = "dodge2")
```

**Dodged charts are often better expressed as faceted plots** to remove legend dependency and improve readability.

## Boxplots and grouped continuous data

### Basic boxplot with readable labels

```r
# Put categories on y-axis for readability
ggplot(organdata, aes(x = donors, y = country)) +
  geom_boxplot()
```

**Rule**: "Save your reader the trouble of having to turn their heads sideways." Place categories on the y-axis when there are many levels.

### Meaningful ordering with reorder()

```r
ggplot(organdata, aes(x = donors,
                       y = reorder(country, donors, na.rm = TRUE))) +
  geom_boxplot() +
  labs(y = NULL)
```

`reorder(category, value)` orders by the mean of `value` by default. Can use any summary function: `reorder(country, donors, median)`.

### Alternatives to boxplots

- `geom_violin()`: Mirrored density distributions
- `geom_point()`: Individual observations
- `geom_jitter()`: Randomly nudged points to reduce overplotting

```r
# Control jitter amount (jitter perpendicular to continuous axis only)
geom_jitter(position = position_jitter(height = 0.15))
```

## Efficient summarization with across()

```r
by_country <- organdata |>
  summarize(
    across(
      where(is.numeric),
      list(mean = \(x) mean(x, na.rm = TRUE),
           sd = \(x) sd(x, na.rm = TRUE))
    ),
    .by = c(consent_law, country)
  )
```

Creates columns named `original_mean`, `original_sd` automatically.

Variable selection methods:
- `where(is.numeric)`: All numeric columns
- `starts_with("prefix")`: By name prefix
- `ends_with("suffix")`: By name suffix
- `contains("string")`: By substring

## Cleveland dot plots

Preferred over bar charts for single-point-per-category data:

```r
ggplot(by_country, aes(x = donors_mean,
                        y = reorder(country, donors_mean),
                        color = consent_law)) +
  geom_point(size = 3) +
  labs(x = "Donor Procurement Rate", y = NULL)
```

### Faceted dot plot with free scales

```r
ggplot(by_country, aes(x = donors_mean,
                        y = reorder(country, donors_mean))) +
  geom_point(size = 3) +
  facet_wrap(~ consent_law, scales = "free_y", ncol = 1) +
  labs(x = "Donor Procurement Rate", y = NULL)
```

### Dot-and-whisker plots

```r
ggplot(by_country, aes(x = donors_mean,
                        y = reorder(country, donors_mean))) +
  geom_pointrange(aes(xmin = donors_mean - donors_sd,
                       xmax = donors_mean + donors_sd))
```

Works for model coefficients with confidence intervals or any point estimate with error bounds. Use `ymin`/`ymax` if continuous variable is on y-axis.

## Labeling points

### geom_text() basics

```r
ggplot(by_country, aes(x = roads_mean, y = donors_mean)) +
  geom_point() +
  geom_text(aes(label = country), hjust = 0)
```

Problem: text overlaps points.

### ggrepel for non-overlapping labels

```r
library(ggrepel)

ggplot(by_country, aes(x = roads_mean, y = donors_mean)) +
  geom_point() +
  geom_text_repel(aes(label = country))
```

- `geom_text_repel()`: Repelled text labels
- `geom_label_repel()`: Repelled labels with background boxes

### Labeling selected points only

Subset data to label only points of interest:

```r
ggplot(by_country, aes(x = roads_mean, y = donors_mean)) +
  geom_point() +
  geom_text_repel(
    data = subset(by_country, gdp_mean > 25000 | health_mean < 1500),
    aes(label = country)
  )
```

### Dummy variable approach for selective labeling

```r
my_data <- data |>
  mutate(ind = id %in% c("Ita", "Spa") & year > 1998)

ggplot(my_data, aes(x = roads, y = donors, color = ind)) +
  geom_point() +
  geom_text_repel(data = subset(my_data, ind),
                   aes(label = id)) +
  guides(color = "none")
```

## annotate() for fixed annotations

`annotate()` is not a geom: it does not map data variables. It draws fixed elements in the plot area.

```r
annotate(geom = "text",
         x = 91, y = 33, size = 4,
         label = "A surprisingly high\nrecovery rate.",
         lineheight = 0.9, hjust = 0)
```

Use `\n` for line breaks. Adjust `lineheight` for spacing.

### Available annotation geoms

- `"text"`: Text
- `"rect"`: Rectangles (uses `xmin`, `xmax`, `ymin`, `ymax`, `fill`, `alpha`)
- `"segment"`: Line segments
- `"pointrange"`: Points with ranges

### Reference lines

```r
geom_hline(yintercept = 0.5, linewidth = 1.4, color = "gray80")
geom_vline(xintercept = 0.5, linewidth = 1.4, color = "gray80")
geom_abline(slope = 1, intercept = 0)  # 45-degree reference line
```

### Relative positioning with I()

Position annotations as proportions of the plot area (robust to data changes):

```r
annotate(geom = "rect",
         xmin = I(0.75), xmax = I(0.95),
         ymin = I(0.45), ymax = I(0.8),
         fill = "red", alpha = 0.2)
```

## Scales, guides, and themes framework

Three distinct systems for customizing plot appearance:

### Scales
Control how data maps to visual properties. Use when the change affects substantive interpretation.

```r
scale_x_log10()
scale_y_continuous(breaks = c(5, 15, 25),
                   labels = c("Five", "Fifteen", "Twenty Five"))
scale_color_discrete(labels = c("Corporatist", "Liberal", "Social Democratic"))
```

**Naming convention**: `scale_<mapping>_<kind>()`
- Mappings: x, y, color, fill, size, shape, alpha, linetype
- Kinds: continuous, discrete, log10, sqrt, date, manual, viridis, brewer

### Guides
Legends and keys. Primary use: removing redundant legends.

```r
guides(fill = "none")
guides(color = "none")
```

### Themes
Cosmetic features not connected to data (background, typeface, legend position). Use when the change does not affect geom interpretation.

```r
theme(legend.position = "top")
theme(legend.position = "bottom")
theme(legend.position = "none")
```

### Decision logic

If the change is connected to data column values: use `aes()` or a `scale_*()` function.
If the change is cosmetic: use a setting in `geom_*()` or `theme()`.

## Legend title via labs()

```r
labs(x = "Road Deaths", y = "Donor Procurement",
     color = "Welfare State",
     fill = "Category")
```
