# Guide to Preparing Figures for Scientific Publications

> Maintained by Pengfei Liu (pengfei.liu@eas.gatech.edu)
> Last reviewed: 2026-08-10

## Purpose

Figures are often the fastest way for a reader to understand a paper. A successful figure communicates one scientific message accurately, efficiently, and accessibly. It should remain understandable when viewed at its final publication size, printed in grayscale, or read by someone outside the immediate specialty.

This guide provides default practices for students and postdoctoral researchers in the Liu group. It is not a substitute for the instructions of the target journal. Before submission, always check that journal's current author guidelines and use them when they differ from this document.

## Group defaults at a glance

Use these defaults while developing a paper, then adapt them to the target journal.

| Item | Recommended default |
| --- | --- |
| Figure width | Design at the intended final width: approximately 85–90 mm for one column or 175–183 mm for two columns |
| Font | One sans-serif family throughout the paper, preferably Arial or Helvetica |
| Text size | Usually 7–8 pt at final size; never rely on enlargement by the reader |
| Panel labels | Lowercase letters in reading order, typically 8–10 pt bold; follow the journal's preferred style, such as **a** or **(a)** |
| Line width | At least 0.5 pt at final size unless the journal specifies otherwise |
| Color mode | RGB unless the journal requests another color space |
| Line plots and diagrams | Vector PDF, SVG, or EPS with embedded fonts |
| Photographs and other raster images | TIFF or high-quality PNG; use JPEG only when appropriate for photographs |
| Raster resolution at final size | At least 300 dpi for color or photographic images; 600 dpi for grayscale; 600–1200 dpi for rasterized line art |
| Reproducibility | Preserve the plotting script, processed plot data, source artwork, and final exported file |

These values are deliberately conservative. For example, Nature commonly uses widths near 89 mm and 183 mm and asks for 5–7 pt text in figures, whereas ACS commonly uses widths near 3.25 in. and 7 in. and requires lettering no smaller than 4.5 pt. Check the [Nature final-submission guidance](https://www.nature.com/nature/for-authors/final-submission), [ACS graphics guidance](https://pubs.acs.org/page/4authors/submission/graphics_prep.html), or the relevant journal instructions before final export.

## Choose software that supports reproducibility

MATLAB, Python, R, and Igor Pro can all produce publication-quality figures. The best choice is usually the tool used for the analysis because it allows the figure to be regenerated when data or calculations change.

- Use a scripting language for final data analysis and plotting whenever practical.
- Excel is useful for quick inspection, but avoid manual spreadsheet operations that cannot be reproduced or audited.
- Assemble multi-panel figures in the plotting code when possible. Adobe Illustrator, Inkscape, Affinity Designer, PowerPoint, or Acrobat may be used for final layout, but retain the editable source file and document any changes made outside the plotting script.
- Do not use a graphics editor to move data points, reshape curves, remove inconvenient features, or otherwise alter the scientific content.
- Keep the original instrument or model output separate from processed data and figure files.

At least one other group member should be able to regenerate the main figures from the archived code and data without relying on undocumented manual steps.

## Start with the scientific message

Before adjusting colors or fonts, write one sentence stating what the figure should show. Every panel should contribute to that sentence or provide evidence needed to interpret it.

1. Decide the comparison or relationship the reader must see.
2. Select the plot type that represents that relationship directly.
3. Remove panels, annotations, and decorative elements that do not help interpretation.
4. Arrange panels in the order in which a reader should examine them.
5. Design at the final publication size and inspect the exported figure at 100% zoom.

Do not create a multi-panel figure merely to fit more results into the paper. Panels should be logically connected, use consistent terminology, and share visual encodings where appropriate.

## Figure size, typography, and panel layout

### Size

Set the figure canvas to its expected published dimensions rather than to US Letter or another manuscript page size. Journal column widths differ. A figure that looks good at 8.5 × 11 in. may become unreadable after reduction.

- Use one-column width for simple plots and two-column width for complex multi-panel figures, maps, or figures with long labels.
- Leave room for the caption on the journal page; maximum figure height is usually substantially less than the full page height.
- Avoid excessive white space, but do not crowd axes, legends, or panel labels.
- Make panels only as large as their information content requires. A simple time series should not dominate a detailed map simply because both are panels.

### Typography

- Use one sans-serif font family consistently across all figures.
- Use sentence case for axis labels and annotations unless disciplinary convention requires otherwise.
- Include a space between a number and its unit where appropriate, and format mathematical symbols consistently with the manuscript.
- Keep text horizontal whenever possible. Long vertical labels and heavily rotated tick labels slow reading.
- Embed fonts in vector files. Do not convert text to outlines unless the journal explicitly requires it.
- Avoid colored text when position, a symbol, or a line sample can identify the same item more accessibly.

### Panel layout

- Arrange panels in a predictable reading order, normally left to right and then top to bottom.
- Place panel labels in the same relative location in every panel.
- Align plot areas, axes, and color bars precisely.
- Use the same axis range for panels intended for direct comparison. If ranges must differ, make the difference obvious.
- Use consistent encodings across panels: the same color, symbol, or line style should represent the same quantity.
- Avoid repeating legends, axis titles, and labels when a shared element is unambiguous.
- Do not put a large title inside each figure; the figure caption provides the title and context.

## Select an appropriate plot type

Match the graphic to the data and scientific question.

- **Comparison of distributions:** Prefer dot plots, box plots, violin plots, or interval plots over bars showing only a mean. Display individual observations when the number of points permits.
- **Change over time or another continuous variable:** Use lines only when connecting observations has a scientific meaning. Show gaps rather than connecting across missing data without explanation.
- **Relationship between variables:** Use scatter plots. Report the fitted model and uncertainty when a fit is central to the interpretation.
- **Composition:** Stacked bars or areas can be difficult to compare away from the baseline. Consider small multiples or direct plots of the important components.
- **Two-dimensional fields:** Use contours, raster maps, or meshes with an appropriate color scale. Do not interpolate beyond the support or resolution of the data without explaining the method.
- **Very dense datasets:** Consider transparency, hexagonal binning, density contours, or aggregation, but state what transformation was used.

Three-dimensional perspective plots, exploded pie charts, decorative gradients, shadows, and pictograms rarely improve a scientific figure. Avoid them unless the third dimension or visual form encodes real information that cannot be shown more clearly in two dimensions.

## Axes, ticks, and grids

Axes are navigation aids; the data should have the greatest visual emphasis. This follows the principle summarized by Krzywinski in [*Axes, ticks and grids*](https://www.nature.com/articles/nmeth.2337): keep navigational elements distinct but unobtrusive.

- Label every quantitative axis with the variable and unit, for example, `PM2.5 concentration (µg m⁻³)`.
- Use a small number of meaningful major ticks. Minor ticks should assist estimation rather than add noise.
- Keep tick directions, decimal precision, and scientific notation consistent across related panels.
- Use light, thin grid lines only when they help compare values. Do not let grids compete with curves or points.
- A linear axis should normally preserve equal visual spacing for equal numerical differences.
- Clearly identify logarithmic, normalized, broken, or transformed axes. State the transformation in the caption or Methods.
- Bar charts should normally begin at zero because bar length encodes magnitude. For line or scatter plots, a nonzero limit can be appropriate, but it must not exaggerate the effect or conceal relevant variation.
- Avoid broken axes when a direct alternative is possible. If a break is necessary, mark it prominently and do not compare visual lengths across the break.
- Avoid dual y-axes because they can create arbitrary visual correlations. If two scales are scientifically necessary, consider aligned panels with a shared x-axis instead.
- Show the full relevant data range, including nonsignificant or negative results, unless exclusions are scientifically justified and documented.

## Color and color maps

Color must represent data rather than decorate the page. The review by Crameri and colleagues, [*The misuse of colour in science communication*](https://www.nature.com/articles/s41467-020-19160-7), explains how nonuniform color maps can introduce false boundaries, hide real gradients, and exclude readers with color-vision deficiencies.

### Continuous data

- Use a perceptually uniform **sequential** map for values that progress from low to high.
- Use a perceptually uniform **diverging** map only when the data have a meaningful central value, such as zero anomaly. Give the two sides comparable visual weight and place the neutral color at the scientific reference value.
- Use a **cyclic** map for cyclic variables such as wind direction or time of day, where the two ends represent the same state.
- Avoid rainbow, `jet`, and similar maps. Their uneven lightness can create artificial features and hide real ones.
- Do not truncate or stretch selected portions of a color map to emphasize a preferred range.
- Label every color bar with the variable and unit. Use interpretable limits and ticks.
- Use the same normalization and limits for maps or panels that are meant to be compared.
- Give missing, masked, below-detection, and out-of-domain values a clearly distinguishable treatment; do not let them look like valid zero values.

Good starting points include `viridis` and `cividis`, the [Scientific colour maps](https://www.fabiocrameri.ch/colourmaps/), and domain-specific collections such as `cmocean`. The choice should be based on the data, not on visual novelty.

### Categories and multiple series

- Use a limited, color-vision-deficiency-friendly categorical palette.
- Do not rely on red versus green alone.
- Encode important distinctions redundantly using symbols, line styles, direct labels, or panel position.
- Check the figure in grayscale and with a color-vision-deficiency simulator.
- Reserve saturated colors for the most important data. Use gray for context or secondary series.
- Keep a category's color consistent throughout the manuscript and supplement.

RGB is the safest default for online publication and is recommended by several major publishers, but the target journal's current instructions take precedence.

## Lines, markers, annotations, and legends

- Make lines thick enough to survive reduction and printing. As a group default, use at least 0.5 pt at final size.
- Select marker sizes that remain distinct without obscuring nearby data.
- Use filled and open markers, shapes, or line styles in addition to color when several series must be distinguished.
- Put annotations near the data they describe. Use arrows sparingly and ensure that their target is unambiguous.
- Prefer direct labels for a small number of curves. When a legend is necessary, order entries to match the visual order of the data.
- Explain all symbols, shaded regions, abbreviations, and reference lines in the caption or legend.
- Do not use asterisks as the only description of statistical results. Define every symbol and provide exact values where the journal requires them.
- Distinguish measured data, model output, fits, and uncertainty visually and describe each encoding.

## Uncertainty, sample size, and statistical results

A visually polished figure cannot compensate for unclear statistical reporting.

- Define the observational or experimental unit and report the exact sample size, `n`.
- State what every center line and error bar represents, such as mean ± standard deviation, median and interquartile range, standard error, or 95% confidence interval.
- Do not use standard deviation, standard error, and confidence intervals interchangeably. They answer different questions.
- Show individual data points when practical, especially for small samples. A bar showing only a mean and an error bar can hide the distribution, outliers, pairing, and sample size.
- For paired measurements, consider connecting paired observations or plotting within-subject differences.
- For regression or model fits, show an uncertainty interval when relevant and report the model, fitting method, and meaning of the interval.
- Use exact *P* values when possible and state the statistical test, whether it was one- or two-sided, and any correction for multiple comparisons.
- Do not infer statistical significance solely from whether error bars overlap. See Krzywinski and Altman's [*Error bars*](https://www.nature.com/articles/nmeth.2659) for common interpretation problems.
- Define how values below detection limits, missing observations, excluded points, and outliers were handled. Exclusion rules should be established scientifically, not chosen to improve the appearance of a result.
- Match the number of displayed significant digits to the precision of the measurement or model.

The figure caption should provide enough information to interpret the displayed uncertainty without searching the Methods section.

## Maps and geospatial figures

Maps require all of the preceding principles plus explicit treatment of geography, projection, and resolution.

- Select a projection appropriate for the region and quantity being interpreted. Do not use Web Mercator by default for quantitative area comparisons.
- State the projection when it affects interpretation.
- Provide longitude and latitude labels, a scale bar, a north arrow, or geographic context only when they are useful; not every map needs all of them.
- Use coastlines and political boundaries as quiet reference features, not as the dominant visual elements.
- Ensure that the plotted resolution and interpolation method do not imply more spatial detail than the observations or model provide.
- Distinguish measurement locations from gridded or interpolated fields.
- Use identical geographic extents and color limits for maps intended for direct comparison.
- Mark missing data and regions outside the analysis domain explicitly.
- State the source and date of basemaps, boundary files, satellite imagery, and other geographic data. Include required attribution or copyright statements.
- Check labels near the international date line, poles, and map boundaries for clipping or duplication.

Copernicus journals provide detailed requirements for figure accessibility, plot data, maps, file formats, and attribution in their [manuscript-preparation guidance](https://publications.copernicus.org/for_authors/manuscript_preparation.html).

## Photographs, microscopy, and instrument images

- Preserve the original, unprocessed image and metadata.
- Apply brightness, contrast, and color adjustments to the complete image and equally to controls. Do not adjust a selected region to strengthen or hide a feature.
- Do not use cloning, healing, content-aware fill, generative fill, or other tools that add, remove, or disguise image data.
- Do not use generative AI to create, replace, extend, or retouch scientific data images. For purely illustrative artwork or a graphical abstract, check the target journal's policy and disclose the tool and use when required.
- Clearly mark and describe cropping, splicing, juxtaposition, insets, or pseudocoloring.
- Do not stretch images nonuniformly. Preserve the aspect ratio.
- Include a scale bar in the image when physical dimensions matter; do not rely only on a magnification value in the caption.
- Use the same processing and display range for images being compared.
- Disclose nonlinear adjustments, thresholding, smoothing, or channel-specific processing.
- Check carefully for accidental duplicated panels or images before submission.

These practices are consistent with the [Nature Portfolio image-integrity guidance](https://research-figure-guide.nature.com/figures/image-integrity/). A journal may request original files during review, so retain them in an organized archive.

## Spectra and time series

These points are particularly important for aerosol, atmospheric chemistry, and mass-spectrometry work.

- Label axes with physical units and state whether the signal is absolute, normalized, background-subtracted, or scaled.
- Describe baseline correction, smoothing, binning, peak fitting, and other processing in the Methods. Do not smooth solely to make a noisy series appear more convincing.
- When spectra are vertically offset, state the offset and ensure the y-axis is not interpreted as an absolute common baseline.
- Label only peaks needed for interpretation; avoid covering the data with annotations.
- For time series, state the time zone and distinguish local time from UTC.
- Show data gaps honestly. Do not connect across long gaps unless the connection represents a model estimate and is identified as such.
- State the averaging interval and whether points represent instantaneous, integrated, median, or mean values.
- When comparing instruments or models, distinguish measurement uncertainty, temporal variability, and fit uncertainty.

## Figure captions

A figure and its caption should be understandable without repeatedly consulting the main text. A strong caption normally contains:

1. A brief title stating what the figure displays.
2. A panel-by-panel description in reading order.
3. Definitions of colors, symbols, lines, shading, abbreviations, and reference values.
4. The sample size and the meaning of error bars or uncertainty bands.
5. Essential statistical tests or processing information needed to interpret the figure.
6. Data-source, map-credit, or image-processing information when relevant.

Do not use the caption to repeat the Results section. Describe what is shown and how to read it; reserve extended interpretation for the main text.

## File formats and export

### Vector artwork

Use vector output for line plots, scatter plots, text, equations, and diagrams whenever possible.

- Preferred working and delivery formats include PDF, SVG, EPS, and editable source formats accepted by the journal.
- Embed fonts and keep text editable.
- Inspect the exported file for missing symbols, substituted fonts, altered transparency, and clipped labels.
- Very dense scatter plots or maps can make vector files excessively large. Rasterize only the dense data layer while keeping axes, labels, and line art as vectors.

### Raster artwork

- Export at the final physical dimensions and required resolution. Changing only the dpi metadata or upsampling a low-resolution image does not create new detail.
- Use TIFF or PNG for lossless raster graphics. Use a high-quality JPEG only for photographic content when accepted by the journal.
- Avoid repeated JPEG editing and saving, which introduces compression artifacts.
- Keep an uncompressed or lossless master copy even if the submission system requests a compressed derivative.
- Confirm that thin lines, marker edges, and small text are not blurred after export.

Current examples of publisher requirements are summarized below. These requirements change, so consult the linked source before submission.

| Publisher example | Typical size and file guidance |
| --- | --- |
| [Nature](https://www.nature.com/nature/for-authors/final-submission) | Approximately 89 mm or 183 mm wide; editable vector artwork for graphs and line art; 300–600 dpi for photographic images; RGB recommended |
| [ACS Publications](https://pubs.acs.org/page/4authors/submission/graphics_prep.html) | Approximately 3.25 in. or 7 in. wide; 300 dpi color, 600 dpi grayscale, and 1200 dpi black-and-white line art when rasterized |
| [Copernicus Publications](https://publications.copernicus.org/for_authors/manuscript_preparation.html) | Figure width at least 8 cm; PDF/EPS preferred for line graphics; accepted raster formats generally require 300 dpi; fonts must be embedded |

## Reproducibility and file organization

For every main-text figure, preserve enough information to regenerate and audit it.

A practical directory pattern is:

```text
figures/
  fig01/
    fig01.py              # or .m, .R, .ipynb, .pxp, etc.
    plot_data/            # the data actually used by the plotting code
    source/               # editable artwork or unprocessed images
    fig01.pdf             # publication-quality figure
    fig01_preview.png     # optional review copy
    README.md             # manual steps, software version, and data provenance
```

- Use version control for plotting code and text-based configuration files.
- Record software and package versions when they can affect the output.
- Save the data needed to reproduce each published plot in a FAIR-aligned repository when possible, subject to privacy, license, and storage constraints.
- Separate raw data, processed data, plotting data, source artwork, and final exports.
- If a panel is edited manually, record exactly what was changed and retain the pre-edit version.
- Use stable filenames such as `fig01.pdf` and `figS01.pdf`, and avoid ambiguous names such as `final2_revised_new.pdf`.

## Final quality-control checklist

Review every item at the actual final figure size.

### Scientific content

- [ ] The figure communicates one clear scientific message.
- [ ] All displayed data are correct and traceable to their source.
- [ ] No observations, ranges, or processing choices are omitted in a misleading way.
- [ ] Sample size, uncertainty, statistical tests, and exclusions are defined where relevant.
- [ ] Comparisons use consistent scales, limits, normalization, and visual encodings.

### Readability and accessibility

- [ ] All text, symbols, and thin lines remain readable at final size.
- [ ] Fonts, capitalization, units, significant digits, and panel labels are consistent.
- [ ] The palette is perceptually appropriate and readable with common color-vision deficiencies.
- [ ] Color is not the only way to distinguish essential categories.
- [ ] The figure remains interpretable in grayscale.
- [ ] Axes, ticks, grids, legends, and annotations support rather than dominate the data.

### Integrity and reproducibility

- [ ] Original images and raw or primary data have been retained.
- [ ] Image adjustments are global, scientifically justified, and documented.
- [ ] There are no accidental duplicated, stretched, clipped, or swapped panels.
- [ ] The plotting code, plot data, and editable source files are archived.
- [ ] Another group member can regenerate or audit the figure.

### Submission files

- [ ] The target journal's current size, resolution, format, and naming requirements have been checked.
- [ ] Fonts are embedded and text is editable in vector files.
- [ ] Raster layers have sufficient resolution at the final physical dimensions.
- [ ] The exported file has been opened independently and checked at 100% zoom.
- [ ] Captions define every panel, abbreviation, symbol, color, and uncertainty indicator.
- [ ] Required map, image, or reused-material permissions and credits are included.

## References and further guidance

- [Nature formatting guide](https://www.nature.com/nature/for-authors/formatting-guide)
- [Nature final-submission and artwork guidance](https://www.nature.com/nature/for-authors/final-submission)
- [Nature research figure guide](https://research-figure-guide.nature.com/)
- [Crameri et al., *The misuse of colour in science communication*](https://www.nature.com/articles/s41467-020-19160-7)
- [Krzywinski, *Axes, ticks and grids*](https://www.nature.com/articles/nmeth.2337)
- [Krzywinski and Altman, *Error bars*](https://www.nature.com/articles/nmeth.2659)
- [ACS guidance for preparing manuscript graphics](https://pubs.acs.org/page/4authors/submission/graphics_prep.html)
- [Copernicus Publications manuscript-preparation guidance](https://publications.copernicus.org/for_authors/manuscript_preparation.html)

