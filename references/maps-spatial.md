# Maps and Spatial Visualization

Source: Healy (2026), Chapter 7 "Draw Maps"

## Core principles

- A map is just points, lines, and shapes drawn on a plane -- like any other ggplot visualization
- Maps are powerful but can mislead. The same data at different aggregation levels produces different impressions.
- Maps are not always the answer. Consider whether a time-series, dot plot, or other visualization better serves your analytical question.

## The Modifiable Areal Unit Problem (MAUP)

Social science data is collected through administrative entities (counties, states, precincts) that may not align with what you are trying to show. Aggregating data at different levels produces different-looking maps with identical underlying data. Document why you chose specific geographical levels.

## Population density confound

U.S. county and state maps often inadvertently display population distribution rather than the variable of interest. Western counties are vastly larger geographically than eastern ones but have fewer people. Always consider creating separate population density maps for comparison.

## Simple Features (sf) -- the recommended approach

Store geometries in tibble columns. Compatible with dplyr and ggplot2.

```r
library(sf)

# sf objects have a geometry column
states_sf    # Simple feature collection

# Join data to geometries
elections_sf <- left_join(states_sf, election_data,
                          by = join_by(fips, st, state, census))

# Draw map
ggplot(elections_sf, aes(fill = party)) +
  geom_sf(color = "white") +
  scale_fill_manual(values = party_colors) +
  labs(title = "Election Results", fill = NULL) +
  theme_map()
```

`geom_sf()` automatically handles coordinate systems. No need to specify x/y aesthetics.

### Joining data to geometries

```r
# Multiple keys
left_join(states_sf, data, by = join_by(fips, st, state))

# Different column names
left_join(states_sf, data, by = join_by(fips, st, census == region))
```

Always verify FIPS codes and administrative boundaries align. Check for missing data (NAs show as blank areas on the map). Consider crosswalks when units change over time.

## Color scales for maps

### Single-ended gradient (magnitude)

```r
scale_fill_gradient(
  low = "white", high = "firebrick",
  labels = scales::label_percent()
)
```

### Diverging gradient (centered on midpoint)

```r
scale_fill_gradient2(
  low = "blue", mid = scales::muted("purple"), high = "red",
  labels = scales::label_percent()
)
```

`scales::muted()` reduces color saturation for the midpoint.

### Discrete categorical

```r
# ColorBrewer palette
scale_fill_brewer(palette = "Greens")
scale_fill_discrete(palette = "Greys")    # Note: "Greys" not "Grays"

# Manual palette from RColorBrewer
orange_pal <- RColorBrewer::brewer.pal(n = 6, name = "Oranges")
scale_fill_manual(values = orange_pal)
```

### Viridis (perceptually uniform, colorblind-safe)

```r
scale_fill_viridis_d(option = "plasma", direction = -1)   # Discrete
scale_fill_viridis_c(option = "viridis", direction = 1)   # Continuous
```

Options: `"viridis"` (default), `"plasma"`, `"inferno"`, `"magma"`, `"cividis"`.

### Handling extreme values

Outliers (e.g., D.C. in election data) compress the scale's gradation for other units. Options:
- Filter the outlier and note it separately
- Use a robust scale transformation
- Use discrete bins instead of continuous gradient

## Discrete binning for maps

```r
data |>
  mutate(rate_bin = cut_width(rate, width = 10, boundary = 0)) |>
  ggplot(aes(fill = rate_bin)) +
  geom_sf() +
  scale_fill_viridis_d(option = "plasma", direction = -1)
```

`cut_width()` creates bins of equal width. `boundary = 0` aligns breaks to round numbers.

## Cartograms with statebins

Equal-area state tiles. Each state gets the same visual weight regardless of geographic size.

```r
library(statebins)

ggplot(election_data, aes(state = st, fill = winner)) +
  geom_statebins(lbl_size = 5) +
  scale_fill_manual(values = party_colors) +
  coord_equal() +
  theme_void() +
  theme(legend.position = "bottom")

# Square tiles (no rounded corners)
geom_statebins(lbl_size = 5, radius = unit(0, "pt"))
```

## Small-multiple maps (faceted by time)

```r
data_sf |>
  filter(year > 2005) |>
  mutate(rate_bin = cut_width(rate, width = 10, boundary = 0)) |>
  st_as_sf() |>
  ggplot(aes(fill = rate_bin)) +
  geom_sf() +
  scale_fill_viridis_d(option = "plasma", direction = -1,
                        labels = \(x) ifelse(is.na(x), "No data", x)) +
  guides(fill = guide_legend(nrow = 1)) +
  facet_wrap(~ year, ncol = 3) +
  theme_map()
```

`st_as_sf()` reasserts sf object status after dplyr operations (safety measure).

## Legacy approach (avoid for new code)

```r
# Old: extract map data from maps package
us_states <- map_data("state") |> as_tibble()

ggplot(us_states, aes(x = long, y = lat, group = group, fill = region)) +
  geom_polygon(color = "gray90", linewidth = 0.1) +
  coord_map(projection = "albers", lat0 = 39, lat1 = 45) +
  guides(fill = "none")
```

Requires manual coordinate transformation. Use sf + `geom_sf()` instead.

## Non-spatial alternatives to maps

### Time-series with highlighted subsets

```r
# Layer 1: All states in gray
p0 <- data |>
  ggplot(aes(x = year, y = rate, group = state)) +
  geom_line(color = "gray40")

# Layer 2: Highlight specific states
p0 +
  geom_line(data = subset(data, highlight),
            aes(color = state), linewidth = 1) +
  geom_text_repel(
    data = subset(data, highlight & year == max(year)),
    aes(label = st),
    xlim = c(max(data$year), NA)
  )
```

### Faceted dot plot (non-geographic comparison)

```r
ggplot(election_data, aes(x = margin, y = reorder(state, margin),
                           color = party)) +
  geom_vline(xintercept = 0, color = "gray60") +
  geom_point(size = 2) +
  scale_color_manual(values = party_colors) +
  facet_wrap(~ reorder(region, margin),
             ncol = 1, scales = "free_y") +
  guides(color = "none") +
  labs(x = "Point Margin", y = NULL)
```

## Map themes

```r
theme_map()     # Removes graticules and axis labels (from socviz package)
theme_void()    # Removes all visual elements
```

## Building complex maps incrementally

Assign intermediate results to named objects to manage layer complexity:

```r
p0 <- base_data |> ggplot(aes(...)) + geom_line(...)
p1 <- p0 + geom_point(data = endpoint_data, ...)
p2 <- p1 + geom_label_repel(data = label_data, ...)
p2 + facet_wrap(...) + scale_color_manual(...) + theme(...)
```

## Projections

- **Albers equal-area**: Standard for U.S. maps. `coord_map(projection = "albers", lat0 = 39, lat1 = 45)`
- With sf objects, projection is embedded in the geometry and `geom_sf()` handles it automatically
- Use `coord_equal()` with statebins to maintain aspect ratio

## Data validation checklist

1. What is the unit of observation? (state, state-year, county?)
2. Do FIPS codes match between data and geometry?
3. Have administrative boundaries changed over time?
4. Are there missing units that will appear as blank areas?
5. Is the variable confounded with population density?
