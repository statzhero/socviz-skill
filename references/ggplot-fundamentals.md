# Core ggplot2 Patterns

Source: Healy (2026), Chapters 2-3 "Get Started" and "Make a Plot"

## Tidy data

ggplot2 expects **long-format** (tidy) data: every variable a column, every observation a row. When struggling with ggplot, the best strategy is to reshape data into a longer format first.

Wide format (variables spread across columns):
```
country  1952  1957  1962
Ireland  66.9  68.5  70.3
France   67.4  68.9  70.5
```

Long format (ready for ggplot):
```
country  year  lifeExp
Ireland  1952  66.9
Ireland  1957  68.5
France   1952  67.4
```

## The ggplot2 template

```r
p <- ggplot(data = <DATA>,
            mapping = aes(<MAPPINGS>))
p + <GEOM_FUNCTION>()
```

Key components:
- `ggplot()` specifies data source and aesthetic mappings
- `mapping = aes(...)` links data variables to visual properties
- `geom_*()` specifies the geometric object to render
- Additional functions adjust scales, labels, coordinates, themes

## Building plots layer by layer

```r
# Step 1: Data and mappings (produces empty axis grid)
p <- ggplot(data = gapminder,
            mapping = aes(x = gdpPercap, y = lifeExp))

# Step 2: Add geom
p + geom_point()

# Step 3: Add smoother
p + geom_point() + geom_smooth()

# Step 4: Transform scale
p + geom_point() +
  geom_smooth() +
  scale_x_log10()

# Step 5: Labels
p + geom_point() +
  geom_smooth() +
  scale_x_log10(labels = scales::label_currency()) +
  labs(x = "GDP Per Capita", y = "Life Expectancy")
```

Inspect the plot at every step. Remove clauses one at a time to understand individual contributions.

## Mapping vs. setting aesthetics

**Critical distinction**:

- **Mapping** (inside `aes()`): Links a data variable to a visual property. ggplot chooses values automatically and creates a legend.
- **Setting** (inside `geom_*()`): Directly assigns a fixed visual property. No legend.

```r
# MAPPING: color varies by continent (legend created)
geom_point(mapping = aes(color = continent))

# SETTING: all points are purple (no legend)
geom_point(color = "purple")
```

**Common mistake**: Putting a string inside `aes()`.
```r
# WRONG: creates legend with "purple" label, uses default color (not purple)
geom_point(aes(color = "purple"))

# RIGHT: sets all points to purple
geom_point(color = "purple")
```

## Aesthetic properties

| Aesthetic | Used for | Mapped or set |
|-----------|----------|---------------|
| `x`, `y` | Position | Always mapped |
| `color` | Point/line color | Both |
| `fill` | Bar/polygon/ribbon interior | Both |
| `size` | Point/line size | Both |
| `shape` | Point shape | Both |
| `alpha` | Transparency (0 = invisible, 1 = opaque) | Both |
| `linetype` | Line pattern | Both |
| `linewidth` | Line width | Both |
| `group` | Grouping (no visual effect, controls drawing) | Mapped only |

## Per-geom mappings

Mappings in `ggplot()` are inherited by all geoms. Mappings in individual geoms override or extend for that layer only.

```r
# Color affects only points; smoother uses all data
ggplot(gapminder, aes(x = gdpPercap, y = lifeExp)) +
  geom_point(aes(color = continent)) +
  geom_smooth(method = "loess")
```

## Smoother methods

```r
geom_smooth()                              # Default: GAM
geom_smooth(method = "lm")                 # Linear model
geom_smooth(method = "loess")              # Local regression
geom_smooth(method = "lm",
            formula = y ~ splines::bs(x, 3))  # Cubic spline
geom_smooth(se = FALSE)                    # No confidence ribbon
```

## Continuous vs. discrete color

```r
# Discrete: creates categorical legend
geom_point(aes(color = continent))

# Continuous: creates gradient scale
geom_point(aes(color = log(pop)))

# Force discrete treatment of numeric variable
geom_point(aes(color = factor(year)))
```

## color vs. fill

- `color` affects lines and point borders
- `fill` affects bar interiors, polygon interiors, and smoother ribbons
- When mapping both, legends combine automatically

```r
ggplot(gapminder, aes(x = gdpPercap, y = lifeExp,
                      color = continent, fill = continent)) +
  geom_point() +
  geom_smooth()
```

## Labels

```r
labs(
  x = "GDP Per Capita",
  y = "Life Expectancy in Years",
  title = "Economic Growth and Life Expectancy",
  subtitle = "Data points are country-years",
  caption = "Source: Gapminder.",
  color = "Continent"          # Legend title for color aesthetic
)
```

## Scale formatting

```r
scale_x_log10(labels = scales::label_currency())
scale_x_continuous(labels = scales::label_comma())
scale_y_continuous(labels = scales::label_percent())
scale_x_sqrt()
scale_x_reverse()
```

## Saving plots

```r
# Save last plot
ggsave("my_figure.pdf")
ggsave("my_figure.png")

# Save specific plot object
ggsave(here::here("figures", "lifexp_vs_gdp.pdf"),
       plot = p_out,
       height = 8, width = 10, units = "in")
```

- **Vector formats** (PDF, SVG): scalable, best for print/journals
- **Raster formats** (PNG, JPG): compressed, web-friendly, resolution-dependent
- Create a `figures/` subdirectory; use `here::here()` for portable paths
- Use descriptive filenames without spaces

## Quarto figure sizing

In YAML header:
```yaml
format:
  html:
    fig-width: 8
    fig-height: 5
```

Per code chunk:
```r
#| fig-width: 12
#| fig-height: 9
```

## Common syntax pitfall

The `+` operator must end the line, not begin the next:

```r
# CORRECT
ggplot(data = mpg, aes(x = displ, y = hwy)) +
  geom_point()

# WRONG (will fail)
ggplot(data = mpg, aes(x = displ, y = hwy))
  + geom_point()
```
