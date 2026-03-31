# Visualizing Model Results

Source: Healy (2026), Chapter 6 "Work with Models"

## Three principles for model-based graphics

1. **Present findings in substantive terms**: Show results at meaningful variable values (means, medians, percentiles). Convert log-odds to predicted probabilities. Generate predicted values for substantively meaningful ranges.
2. **Show degree of confidence**: Include uncertainty measures. Use `geom_pointrange()`, `geom_errorbar()`, `geom_ribbon()`.
3. **Show data when you can**: Layer original data with model estimates using ggplot's layer-by-layer approach.

## Multiple smoothers with a legend

Map color/fill aesthetics to model names as strings within `aes()`, then use `scale_*_manual()` to create legends:

```r
model_colors <- RColorBrewer::brewer.pal(3, "Dark2") |>
  set_names(c("OLS", "Cubic spline", "LOESS"))

ggplot(gapminder, aes(x = log(gdpPercap), y = lifeExp)) +
  geom_point(alpha = 0.2) +
  geom_smooth(method = "lm",
              aes(color = "OLS", fill = "OLS")) +
  geom_smooth(method = "loess",
              aes(color = "LOESS", fill = "LOESS")) +
  scale_color_manual(name = "Models", values = model_colors) +
  scale_fill_manual(name = "Models", values = model_colors) +
  theme(legend.position = "top")
```

## Tidy model objects with broom

The broom package converts model objects into tidy data frames for plotting.

### Three functions

```r
library(broom)

out <- lm(lifeExp ~ gdpPercap + pop + continent, data = gapminder)
```

**Component-level: `tidy()`** -- One row per coefficient

```r
tidy(out)                          # estimate, std.error, statistic, p.value
tidy(out, conf.int = TRUE)         # adds conf.low, conf.high
```

**Observation-level: `augment()`** -- One row per observation

```r
augment(out)                       # .fitted, .resid, .hat, .cooksd, .std.resid
augment(out, data = gapminder)     # includes original data columns
```

**Model-level: `glance()`** -- One row per model

```r
glance(out)                        # r.squared, adj.r.squared, sigma, AIC, BIC, nobs
```

## Coefficient plots (dot-and-whisker)

```r
out_conf <- tidy(out, conf.int = TRUE)

ggplot(out_conf,
       aes(x = estimate,
           xmin = conf.low,
           xmax = conf.high,
           y = reorder(term, estimate))) +
  geom_vline(xintercept = 0, color = "gray80") +
  geom_pointrange() +
  labs(x = "OLS Estimate", y = NULL)
```

Order coefficients by estimate size. Add a vertical reference line at zero. Use `geom_pointrange()` for point + CI. Consider dropping the intercept if it compresses the scale.

## Residual diagnostics

```r
out_aug <- augment(out, data = gapminder)

ggplot(out_aug, aes(x = .fitted, y = .resid)) +
  geom_point(alpha = 0.2) +
  geom_hline(yintercept = 0, color = "gray80") +
  labs(x = "Fitted Values", y = "Residuals")
```

## geom_smooth() methods reference

```r
geom_smooth(method = "lm")                                    # OLS
geom_smooth(method = "loess")                                  # Local regression
geom_smooth(method = "gam")                                    # GAM (default)
geom_smooth(method = MASS::rlm)                                # Robust regression
geom_smooth(method = "lm",
            formula = y ~ splines::bs(x, 3), se = FALSE)      # Cubic spline
```

### Quantile regression

```r
geom_quantile(color = "tomato",
              method = "rqss",
              lambda = 1,
              quantiles = c(0.20, 0.5, 0.85))
```

## Split-apply-combine with nested models

Fit many models across groups using nest + map:

```r
fit_ols <- function(df) {
  lm(lifeExp ~ log(gdpPercap), data = df)
}

out_tidy <- gapminder |>
  nest(.by = c(continent, year)) |>
  mutate(
    model = map(data, fit_ols),
    tidied = map(model, tidy)
  ) |>
  unnest(cols = c(tidied)) |>
  filter(term != "(Intercept)")
```

### Plot nested results with position dodging

```r
ggplot(out_tidy,
       aes(x = year, y = estimate,
           ymin = estimate - 2 * std.error,
           ymax = estimate + 2 * std.error,
           color = continent)) +
  geom_pointrange(position = position_dodge(width = 1)) +
  scale_x_continuous(breaks = unique(gapminder$year)) +
  theme(legend.position = "top")
```

## Marginal effects with marginaleffects

```r
library(marginaleffects)

out_bo <- glm(obama ~ polviews_m + sex * race,
              family = "binomial",
              data = my_gss,
              na.action = na.omit)

# Average marginal effects
avg_slopes(out_bo) |>
  as_tibble() |>
  ggplot(aes(x = estimate,
             xmin = conf.low,
             xmax = conf.high,
             y = reorder(contrast, estimate))) +
  geom_vline(xintercept = 0, color = "gray80") +
  geom_pointrange() +
  labs(x = "Average Marginal Effect", y = NULL)
```

For deeper marginaleffects usage, see the `/marginaleffects` skill.

## Survey-weighted estimates

```r
library(survey)
library(srvyr)

options(survey.lonely.psu = "adjust")
options(na.action = "na.pass")

gss_wt <- data |>
  mutate(stratvar = interaction(year, vstrat)) |>
  as_survey_design(
    ids = vpsu,
    strata = stratvar,
    weights = wtssps,
    nest = TRUE
  )

out_grp <- gss_wt |>
  filter(year %in% seq(1976, 2016, by = 4)) |>
  summarize(
    prop = survey_mean(vartype = "ci", na.rm = TRUE),
    .by = c(year, race, degree)
  ) |>
  drop_na(degree)
```

## Quick tabulation shortcuts

```r
# count() combines group_by + summarize(n = n())
data |> count(bigregion, religion)

# tally() is summarize(n = n()) with existing groups
data |> group_by(region) |> tally()
```

## Faceting strategy for model results

**Prefer faceting by category with time on x-axis** (lines showing trends) over faceting by time with categories as bars. Lines make trends visible; bars across facets require back-and-forth eye movement.

```r
ggplot(out_grp, aes(x = year, y = prop, color = race)) +
  geom_line() +
  geom_ribbon(aes(ymin = prop_low, ymax = prop_upp, fill = race),
              alpha = 0.2) +
  facet_wrap(~ degree, ncol = 2) +
  scale_y_continuous(labels = scales::label_percent())
```
