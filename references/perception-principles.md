# Visual Perception and Chart Choice

Source: Healy (2026), Chapter 1 "Look at Data"

## Why visualize

Summary statistics can be identical for wildly different data patterns. Anscombe's Quartet: four datasets with the same mean, variance, and correlation (0.81) that reveal completely different structures when plotted. Always visualize data alongside numerical summaries.

## Three categories of bad figures

### 1. Bad taste (aesthetic problems)
Tacky design choices without functional benefit: 3D effects, drop shadows, gradient fills, redundant labels. Tufte's principle: maximize the data-to-ink ratio. However, some visual scaffolding aids comprehension. Anderson et al. (2011) found Tufte's minimalist boxplot was cognitively hardest to interpret. Visually embellished infographics are more memorable (Bateman et al. 2010; Borkin et al. 2013).

**Takeaway**: Simplify, but do not strip all visual cues. Labels and some scaffolding help readers.

### 2. Bad data (substantive problems)
Misleading representation through data selection, aggregation level, or measure choice. Example: reporting only top-score respondents (10 on 10-point scale) makes a modest decline look dramatic. Cherry-picking data, restricting to favorable comparisons. Good design does not protect against this.

### 3. Bad perception (perceptual problems)
Data encoded in ways humans perceive poorly. Examples:
- 3D bar charts: perspective makes bars appear shorter than true values
- Stacked bar charts: only the bottom category (sharing the baseline) is trackable; middle categories' trends are invisible
- Aspect ratio: narrow panels make curves appear to converge; wide panels minimize perceived change

## Visual channels ranked (Cleveland and McGill 1984, Heer and Bostock 2010)

### For ordered/quantitative data (best to worst)

1. **Position on common scale** (bar charts, dot plots, scatterplots)
2. **Position on unaligned scales** (small multiples with different baselines)
3. **Length** (stacked bars, error bars)
4. **Tilt/angle** (pie charts -- poor)
5. **Area** (bubble charts -- poor)
6. **Luminance/saturation** (heatmaps)
7. **Curvature**
8. **Volume** (3D objects -- worst)

### For unordered categorical data (best to worst)

1. **Spatial region** (faceting)
2. **Color hue**
3. **Motion**
4. **Shape**

**Rule**: Use position on a common scale for accurate quantitative comparisons. Reserve area and angle for rough impressions only.

## Preattentive features

Some visual features "pop out" before conscious search:
- **Color** produces strong pop-out (find a blue dot among yellow dots -- instant)
- **Shape** produces weaker pop-out (find a circle among triangles -- slower)
- **Dual-channel searches** (find blue circle among mixed colors and shapes) are very slow

**Implication**: Adding multiple channels to a single graph overtaxes viewers quickly. Think carefully before representing different variables by shape, color, and size simultaneously.

## Gestalt principles

The perceptual system infers relationships from sparse visual information:

1. **Proximity**: Spatially near things seem related
2. **Similarity**: Alike things seem related
3. **Connection**: Visually tied things seem related (stronger than shape)
4. **Continuity**: Partially hidden objects completed to familiar shapes
5. **Closure**: Incomplete shapes perceived as complete
6. **Figure and ground**: Foreground vs. background separation
7. **Common fate**: Elements moving together perceived as a unit

**Practical consequence**: Viewers will find structure even in random data. Connection is stronger than shape; proximity is stronger than alignment.

## Color

### Three components
- **Hue**: Dominant wavelength (red, blue, green)
- **Luminance**: Perceived brightness
- **Chroma**: Intensity/vividness

### Perceptual non-uniformity
- Equal numerical gaps in RGB space are perceived unequally
- Edge contrasts in monochrome are stronger than in color
- Never pick colors ad hoc

### Solution: perceptually uniform palettes
- Use HCL (Hue-Chroma-Luminance) color space
- **Sequential palettes** (one direction): for magnitude data
- **Diverging palettes** (two directions from midpoint): for data centered on a meaningful value
- **Qualitative palettes** (distinct hues): for unordered categories
- Use ColorBrewer, viridis, or colorspace package palettes
- All are colorblind-safe by design

## Honesty and judgment

### The zero-baseline debate
- **Bar charts** should generally include zero (bars encode length; zero is the natural reference)
- **Dot plots** can legitimately start at the data minimum (they encode position, not length)
- Restricting the scale to the data range shows actual variation more clearly, but can be exploited to exaggerate small differences
- Expanding the scale to include zero can waste space and obscure real variation
- No rule of thumb solves this. Document your choice; label axes clearly.

### Aspect ratio
- Rate-of-change perception depends heavily on panel width
- Same curves can appear converging or equidistant depending on aspect ratio
- Be deliberate about figure dimensions

## Summary decision rules

| Question | Guidance |
|----------|----------|
| Comparing quantities? | Position on common scale (dot plot, bar chart) |
| Showing distribution? | Histogram or density (experiment with bin width) |
| Showing trend over time? | Line chart with `group` aesthetic |
| Comparing categories? | Faceted small multiples over stacked/dodged bars |
| Color encoding? | Perceptually uniform palettes (viridis, ColorBrewer) |
| Multiple variables? | Limit channels; facet instead of overloading |
| Pie chart? | Almost never. Use bar chart or dot plot instead. |
| 3D? | Never for statistical graphics |
