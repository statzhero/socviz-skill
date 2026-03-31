# Grouped Data, Faceting, and Transformations

Source: Healy (2026), Chapter 4 "Show the Right Numbers"

## The group aesthetic

When plotting trajectories for multiple entities (countries, individuals over time), ggplot needs to know how observations are grouped.

```r
# WRONG: single tangled line connecting all observations
ggplot(gapminder, aes(x = year, y = gdpPercap)) +
  geom_line()

# RIGHT: one line per country
ggplot(gapminder, aes(x = year, y = gdpPercap)) +
  geom_line(aes(group = country))
```

**Diagnostic rule**: If output looks weirdly jagged or sharded, data are grouped in a way not represented by your mappings. Add the `group` aesthetic.

Note: When `color` or `fill` is mapped to a variable, grouping is implicit. You only need explicit `group` when no other aesthetic distinguishes the groups.

## Faceting: small multiples

### facet_wrap() for a single variable

```r
ggplot(gapminder, aes(x = year, y = gdpPercap)) +
  geom_line(color = "gray70", aes(group = country)) +
  geom_smooth(linewidth = 1.25, method = "loess", se = FALSE) +
  scale_y_log10(labels = scales::label_currency()) +
  facet_wrap(~ continent, ncol = 5) +
  labs(x = "Year", y = "GDP per capita")
```

Uses tilde (`~`) formula syntax. Control layout with `ncol` or `nrow`.

### facet_grid() for two variables

```r
ggplot(gss_sm, aes(x = age, y = childs)) +
  geom_point(alpha = 0.2) +
  geom_smooth(se = FALSE) +
  facet_grid(sex ~ race)
```

Two-sided formula: `facet_grid(row_variable ~ column_variable)` creates a contingency-table layout. Becomes unwieldy with many categories.

### Free scales in facets

```r
facet_wrap(~ consent_law, scales = "free_y", ncol = 1)
```

- `scales = "free_y"` allows y-axis to vary by panel
- `scales = "free_x"` for x-axis
- `scales = "free"` for both
- **Caution**: Free scales break comparability across panels. Generally not recommended for continuous-continuous comparisons, but useful for categorical axes with different levels per panel.

### Design pattern: gray context + colored focus

Layer all data in gray, then overlay the trend or focus in color:

```r
ggplot(gapminder, aes(x = year, y = gdpPercap)) +
  geom_line(color = "gray70", aes(group = country)) +
  geom_smooth(color = "steelblue", method = "loess", se = FALSE) +
  facet_wrap(~ continent, ncol = 5)
```

## Stat transformations

Every `geom_*()` has an associated `stat_*()` function that performs calculations before plotting.

- `geom_point()` uses `stat_identity()` (plots raw data)
- `geom_bar()` uses `stat_count()` (counts rows per category)
- `geom_smooth()` uses `stat_smooth()` (fits a model)
- `geom_histogram()` uses `stat_bin()` (bins data)

### geom_bar() counts automatically

```r
# Counts rows per category
ggplot(gss_sm, aes(x = bigregion)) +
  geom_bar()
```

### Proportions with after_stat()

```r
# ALL-category proportions (sum to 1)
ggplot(gss_sm, aes(x = bigregion,
                    y = after_stat(prop),
                    group = 1)) +
  geom_bar()
```

**Critical**: `group = 1` treats the entire dataset as one group for the denominator. Without it, each bar computes its proportion within its own group, and all bars equal 1.0.

### Stacked proportional bars

```r
# position = "fill" stacks to 1.0 within each x category
ggplot(gss_sm, aes(x = bigregion, fill = religion)) +
  geom_bar(position = "fill")
```

Trade-off: only the bottom category is easily comparable across groups.

### Dodged bars

```r
ggplot(gss_sm, aes(x = bigregion, fill = religion)) +
  geom_bar(position = "dodge")
```

Often better expressed as faceted plots to avoid grouping ambiguity.

### Pre-computed data: use geom_col()

```r
# geom_col() = geom_bar(stat = "identity")
# Plots values as-is, no counting
ggplot(summary_df, aes(x = category, y = value)) +
  geom_col()
```

### Positive and negative values

```r
ggplot(oecd_sum, aes(x = year, y = diff, fill = hi_lo)) +
  geom_col() +
  guides(fill = "none")
```

## Histograms and density plots

### Histograms

```r
ggplot(data, aes(x = variable)) +
  geom_histogram()                    # Default binning
  geom_histogram(bins = 20)           # By number
  geom_histogram(binwidth = 1)        # By width
```

Always experiment with binning. The choice significantly affects the visual message.

### Comparing distributions

```r
# Overlapping histograms with transparency
ggplot(subset_data, aes(x = variable, fill = group)) +
  geom_histogram(alpha = 0.4, bins = 20, color = "gray40")
```

### Kernel density

```r
ggplot(data, aes(x = variable)) +
  geom_density(fill = "steelblue", alpha = 0.3)
```

Advantages: smooth, no arbitrary binning. Disadvantages: can mask small sample size; y-axis (density) less intuitive than counts.

### Frequency polygon (alternative to histogram)

```r
geom_freqpoly(binwidth = 1, linewidth = 1)
```

Line-based histogram. Useful for overlaying on top of filled histograms.

### Scaled density for comparisons

```r
aes(y = after_stat(scaled))
```

Rescales density values to 0-1 range, making curves with different sample sizes comparable.

## color vs. fill

- `color` colors borders and lines
- `fill` colors shape interiors

```r
# Border color only
ggplot(gss_sm, aes(x = religion, color = religion)) +
  geom_bar()

# Interior fill (what you usually want)
ggplot(gss_sm, aes(x = religion, fill = religion)) +
  geom_bar() +
  guides(fill = "none")  # Remove redundant legend
```

## Removing redundant legends

```r
guides(fill = "none")    # Remove fill legend
guides(color = "none")   # Remove color legend
guides(x = "none")       # Remove x-axis display
```

## General principle: calculate before you graph

When `geom_bar()` proportions get confusing (which group is the denominator?), calculate the frequency table with dplyr first, then use `geom_col()`:

```r
summary <- data |>
  summarize(n = n(), .by = c(region, religion)) |>
  mutate(pct = n / sum(n), .by = region)

ggplot(summary, aes(x = religion, y = pct)) +
  geom_col() +
  facet_wrap(~ region)
```
