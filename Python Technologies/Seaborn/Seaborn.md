# Seaborn Comprehensive, Structured, and Progressive Learning Roadmap

## From Statistical Visualization Foundations to Advanced Multi-Plot Grids, Customization, and Production Data Visualization Engineering

Seaborn is best learned as more than "a wrapper around Matplotlib." The progression should cover **Python prerequisites → Matplotlib prerequisites → Pandas prerequisites → statistical visualization fundamentals → relational plots → distribution plots → categorical plots → regression plots → matrix plots → multi-plot grids → customization → themes → color palettes → statistical estimation → advanced customization → performance → integration → production visualization engineering**.

---

# I. Seaborn Foundations

- **1. What Seaborn Is**
  - Seaborn
  - Seaborn history
  - Michael Waskom
  - Seaborn 0.9
  - Seaborn 0.10
  - Seaborn 0.11
  - Seaborn 0.12
  - Seaborn 0.13 (current)
  - Seaborn philosophy
    - Statistical visualization
    - Built on Matplotlib
    - Pandas integration
    - Dataset-oriented
    - Declarative
    - Beautiful defaults
  - Seaborn vs Matplotlib
  - Seaborn vs Plotly
  - Seaborn vs Bokeh
  - Seaborn vs Altair
  - Seaborn vs ggplot2
  - Seaborn use cases
    - Exploratory data analysis
    - Statistical visualization
    - Publication figures
    - Data science
    - Machine learning
    - Business analytics
    - Scientific research
    - Financial analysis
  - Seaborn in modern data science
  - Seaborn ecosystem
  - Seaborn API
  - Seaborn best practices

- **2. Prerequisites**
  - Python fundamentals
  - Variables
  - Data types
  - Control flow
  - Functions
  - Classes
  - Modules
  - NumPy
    - ndarray
    - Indexing
    - Slicing
    - Broadcasting
    - Aggregation
  - Pandas
    - Series
    - DataFrame
    - Index
    - Selection
    - Cleaning
    - Transformation
    - Grouping
    - Merging
    - Reshaping
  - Matplotlib
    - Figure
    - Axes
    - Plotting
    - Subplots
    - Styling
    - Legends
    - Annotations
  - Jupyter
    - Notebooks
    - Cells
  - Statistics
    - Descriptive statistics
    - Distributions
    - Correlation
    - Regression
  - Prerequisite best practices

- **3. Installing Seaborn**
  - Installation
    - pip
    - conda
    - mamba
    - uv
  - `pip install seaborn`
  - `conda install seaborn`
  - Version checking
  - `sns.__version__`
  - Dependencies
    - NumPy
    - Pandas
    - Matplotlib
  - Optional dependencies
    - SciPy
    - statsmodels
    - fastcluster
  - Installation best practices

- **4. Importing Seaborn**
  - `import seaborn as sns`
  - `import matplotlib.pyplot as plt`
  - `import pandas as pd`
  - `import numpy as np`
  - Import best practices
  - Namespace conventions

- **5. Seaborn API**
  - Figure-level functions
    - `relplot`
    - `displot`
    - `catplot`
    - `lmplot`
    - `clustermap`
    - `jointplot`
    - `pairplot`
    - `FacetGrid`
    - `PairGrid`
    - `JointGrid`
  - Axes-level functions
    - `scatterplot`
    - `lineplot`
    - `histplot`
    - `kdeplot`
    - `ecdfplot`
    - `rugplot`
    - `boxplot`
    - `violinplot`
    - `boxenplot`
    - `stripplot`
    - `swarmplot`
    - `barplot`
    - `pointplot`
    - `countplot`
    - `regplot`
    - `residplot`
    - `heatmap`
    - `clustermap`
  - Figure-level vs Axes-level
  - API best practices

- **6. First Seaborn Plot**
  - Dataset loading
  - Basic plot
  - Figure-level plot
  - Axes-level plot
  - First plot best practices

- **7. Built-in Datasets**
  - `sns.load_dataset()`
  - Datasets
    - `tips`
    - `titanic`
    - `iris`
    - `penguins`
    - `flights`
    - `diamonds`
    - `fmri`
    - `planets`
    - `exercise`
    - `car_crashes`
    - `dots`
    - `anscombe`
    - `attention`
    - `brain_networks`
    - `dowjones`
    - `geyser`
    - `glue`
    - `healthexp`
    - `mpg`
    - `paintings`
    - `seaice`
    - `smokers`
    - `space_launches`
    - `taxis`
    - `titanic`
  - Dataset exploration
  - Dataset best practices

---

# II. Relational Plots

- **8. Relational Plot Fundamentals**
  - Relational plots
  - Relationships between variables
  - `relplot`
  - `scatterplot`
  - `lineplot`
  - Relational plot best practices

- **9. Scatter Plots**
  - `scatterplot`
  - `relplot(kind='scatter')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `size`
    - `style`
    - `palette`
    - `sizes`
    - `markers`
    - `alpha`
    - `data`
  - Semantic mappings
  - Scatter plot best practices

- **10. Line Plots**
  - `lineplot`
  - `relplot(kind='line')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `size`
    - `style`
    - `units`
    - `estimator`
    - `errorbar`
    - `ci`
    - `n_boot`
    - `seed`
    - `sort`
    - `err_style`
    - `err_kws`
    - `dashes`
    - `markers`
  - Aggregation
  - Confidence intervals
  - Line plot best practices

- **11. Relational Plot Customization**
  - Faceting
  - `col`
  - `row`
  - `col_wrap`
  - `height`
  - `aspect`
  - `facet_kws`
  - Customization best practices

- **12. Relational Plot Best Practices**
  - Variable selection
  - Semantic mapping
  - Faceting
  - Plot aesthetics
  - Best practices

---

# III. Distribution Plots

- **13. Distribution Plot Fundamentals**
  - Distribution plots
  - Univariate distributions
  - Bivariate distributions
  - `displot`
  - `histplot`
  - `kdeplot`
  - `ecdfplot`
  - `rugplot`
  - Distribution plot best practices

- **14. Histograms**
  - `histplot`
  - `displot(kind='hist')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `bins`
    - `binwidth`
    - `binrange`
    - `discrete`
    - `cumulative`
    - `stat`
    - `element`
    - `fill`
    - `multiple`
    - `shrink`
    - `kde`
    - `kde_kws`
    - `hue_order`
  - Histogram best practices

- **15. KDE Plots**
  - `kdeplot`
  - `displot(kind='kde')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `bw_adjust`
    - `bw_method`
    - `common_norm`
    - `common_grid`
    - `cumulative`
    - `log_scale`
    - `levels`
    - `thresh`
    - `fill`
    - `multiple`
  - Bandwidth selection
  - KDE best practices

- **16. ECDF Plots**
  - `ecdfplot`
  - `displot(kind='ecdf')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `stat`
    - `complementary`
  - ECDF best practices

- **17. Rug Plots**
  - `rugplot`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `height`
    - `expand_margins`
  - Rug plot best practices

- **18. Bivariate Distributions**
  - `histplot` with `x` and `y`
  - `kdeplot` with `x` and `y`
  - Joint distributions
  - Bivariate best practices

- **19. Distribution Plot Customization**
  - Faceting
  - `col`
  - `row`
  - `col_wrap`
  - `height`
  - `aspect`
  - Customization best practices

---

# IV. Categorical Plots

- **20. Categorical Plot Fundamentals**
  - Categorical plots
  - Categorical data
  - `catplot`
  - `stripplot`
  - `swarmplot`
  - `boxplot`
  - `violinplot`
  - `boxenplot`
  - `pointplot`
  - `barplot`
  - `countplot`
  - Categorical plot best practices

- **21. Strip Plots**
  - `stripplot`
  - `catplot(kind='strip')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `order`
    - `hue_order`
    - `jitter`
    - `dodge`
    - `orient`
    - `color`
    - `palette`
    - `size`
    - `edgecolor`
    - `linewidth`
    - `native_scale`
    - `formatter`
    - `legend`
  - Strip plot best practices

- **22. Swarm Plots**
  - `swarmplot`
  - `catplot(kind='swarm')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `order`
    - `hue_order`
    - `dodge`
    - `orient`
    - `color`
    - `palette`
    - `size`
    - `edgecolor`
    - `linewidth`
    - `native_scale`
    - `formatter`
    - `legend`
    - `warn_thresh`
  - Swarm plot best practices

- **23. Box Plots**
  - `boxplot`
  - `catplot(kind='box')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `order`
    - `hue_order`
    - `orient`
    - `color`
    - `palette`
    - `saturation`
    - `width`
    - `dodge`
    - `fliersize`
    - `linewidth`
    - `whis`
    - `fill`
    - `gap`
  - Box plot best practices

- **24. Violin Plots**
  - `violinplot`
  - `catplot(kind='violin')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `order`
    - `hue_order`
    - `orient`
    - `color`
    - `palette`
    - `saturation`
    - `width`
    - `dodge`
    - `inner`
    - `split`
    - `scale`
    - `scale_hue`
    - `bw`
    - `bw_method`
    - `cut`
    - `gridsize`
    - `density_norm`
    - `common_norm`
    - `linewidth`
    - `fill`
    - `gap`
  - Violin plot best practices

- **25. Boxen Plots**
  - `boxenplot`
  - `catplot(kind='boxen')`
  - Letter-value plots
  - Boxen plot best practices

- **26. Point Plots**
  - `pointplot`
  - `catplot(kind='point')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `order`
    - `hue_order`
    - `estimator`
    - `errorbar`
    - `ci`
    - `n_boot`
    - `units`
    - `seed`
    - `markers`
    - `linestyles`
    - `dodge`
    - `join`
    - `scale`
    - `orient`
    - `color`
    - `palette`
    - `errwidth`
    - `capsize`
    - `err_kws`
  - Point plot best practices

- **27. Bar Plots**
  - `barplot`
  - `catplot(kind='bar')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `order`
    - `hue_order`
    - `estimator`
    - `errorbar`
    - `ci`
    - `n_boot`
    - `units`
    - `seed`
    - `orient`
    - `color`
    - `palette`
    - `saturation`
    - `width`
    - `dodge`
    - `gap`
    - `log_scale`
    - `native_scale`
    - `formatter`
    - `legend`
    - `errcolor`
    - `errwidth`
    - `capsize`
    - `err_kws`
  - Bar plot best practices

- **28. Count Plots**
  - `countplot`
  - `catplot(kind='count')`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `order`
    - `hue_order`
    - `orient`
    - `color`
    - `palette`
    - `saturation`
    - `dodge`
    - `gap`
    - `log_scale`
    - `native_scale`
    - `formatter`
    - `legend`
    - `stat`
  - Count plot best practices

- **29. Categorical Plot Customization**
  - Faceting
  - `col`
  - `row`
  - `col_wrap`
  - `height`
  - `aspect`
  - `kind`
  - Customization best practices

---

# V. Regression Plots

- **30. Regression Plot Fundamentals**
  - Regression plots
  - `lmplot`
  - `regplot`
  - `residplot`
  - Regression plot best practices

- **31. Linear Regression Plots**
  - `regplot`
  - `lmplot`
  - Parameters
    - `x`
    - `y`
    - `hue`
    - `data`
    - `order`
    - `logistic`
    - `lowess`
    - `robust`
    - `logx`
    - `x_estimator`
    - `x_bins`
    - `x_ci`
    - `scatter`
    - `fit_reg`
    - `ci`
    - `n_boot`
    - `units`
    - `seed`
    - `marker`
    - `color`
    - `scatter_kws`
    - `line_kws`
    - `truncate`
    - `dropna`
    - `x_jitter`
    - `y_jitter`
    - `label`
  - Linear regression best practices

- **32. Polynomial Regression**
  - `order`
  - Polynomial regression
  - Polynomial best practices

- **33. Logistic Regression**
  - `logistic=True`
  - Logistic regression
  - Logistic best practices

- **34. LOWESS Regression**
  - `lowess=True`
  - LOWESS
  - LOWESS best practices

- **35. Robust Regression**
  - `robust=True`
  - Robust regression
  - Robust best practices

- **36. Residual Plots**
  - `residplot`
  - Residuals
  - Residual plot best practices

- **37. Regression Plot Customization**
  - Faceting
  - `col`
  - `row`
  - `col_wrap`
  - `height`
  - `aspect`
  - Customization best practices

---

# VI. Matrix Plots

- **38. Matrix Plot Fundamentals**
  - Matrix plots
  - Heatmaps
  - Clustermaps
  - Matrix plot best practices

- **39. Heatmaps**
  - `heatmap`
  - Parameters
    - `data`
    - `vmin`
    - `vmax`
    - `cmap`
    - `center`
    - `robust`
    - `annot`
    - `fmt`
    - `annot_kws`
    - `linewidths`
    - `linecolor`
    - `cbar`
    - `cbar_kws`
    - `cbar_ax`
    - `square`
    - `xticklabels`
    - `yticklabels`
    - `mask`
    - `ax`
    - `dtype`
  - Heatmap best practices

- **40. Clustermaps**
  - `clustermap`
  - Parameters
    - `data`
    - `pivot_kws`
    - `method`
    - `metric`
    - `z_score`
    - `standard_scale`
    - `figsize`
    - `dendrogram_ratio`
    - `colors_ratio`
    - `cbar_pos`
    - `tree_kws`
    - `cmap`
    - `center`
    - `robust`
    - `annot`
    - `fmt`
    - `annot_kws`
    - `linewidths`
    - `linecolor`
    - `cbar`
    - `cbar_kws`
    - `cbar_ax`
    - `square`
    - `xticklabels`
    - `yticklabels`
    - `mask`
    - `dtype`
  - Clustermap best practices

- **41. Correlation Matrices**
  - Correlation matrices
  - `df.corr()`
  - Heatmap
  - Correlation matrix best practices

- **42. Pivot Tables**
  - Pivot tables
  - Heatmap
  - Pivot table best practices

---

# VII. Multi-Plot Grids

- **43. Multi-Plot Grid Fundamentals**
  - Multi-plot grids
  - Faceting
  - Grids
  - Multi-plot grid best practices

- **44. FacetGrid**
  - `FacetGrid`
  - Parameters
    - `data`
    - `row`
    - `col`
    - `hue`
    - `row_order`
    - `col_order`
    - `hue_order`
    - `palette`
    - `hue_kws`
    - `height`
    - `aspect`
    - `layout_pad`
    - `legend_out`
    - `sharex`
    - `sharey`
    - `margin_titles`
    - `facet_kws`
    - `despine`
  - Mapping functions
    - `.map()`
    - `.map_dataframe()`
  - FacetGrid best practices

- **45. PairGrid**
  - `PairGrid`
  - Parameters
    - `data`
    - `hue`
    - `hue_order`
    - `palette`
    - `vars`
    - `x_vars`
    - `y_vars`
    - `height`
    - `aspect`
    - `despine`
    - `corner`
    - `diag_sharey`
    - `layout_pad`
    - `dropna`
  - Mapping functions
    - `.map()`
    - `.map_diag()`
    - `.map_offdiag()`
    - `.map_lower()`
    - `.map_upper()`
  - PairGrid best practices

- **46. JointGrid**
  - `JointGrid`
  - Parameters
    - `x`
    - `y`
    - `data`
    - `hue`
    - `palette`
    - `height`
    - `ratio`
    - `space`
    - `dropna`
    - `xlim`
    - `ylim`
    - `marginal_ticks`
  - Mapping functions
    - `.plot()`
    - `.plot_joint()`
    - `.plot_marginals()`
  - JointGrid best practices

- **47. Pairplot**
  - `pairplot`
  - Pairwise relationships
  - Parameters
    - `data`
    - `hue`
    - `hue_order`
    - `palette`
    - `vars`
    - `x_vars`
    - `y_vars`
    - `kind`
    - `diag_kind`
    - `markers`
    - `height`
    - `aspect`
    - `corner`
    - `dropna`
    - `plot_kws`
    - `diag_kws`
    - `grid_kws`
    - `size`
  - Pairplot best practices

- **48. Jointplot**
  - `jointplot`
  - Joint distribution
  - Parameters
    - `data`
    - `x`
    - `y`
    - `hue`
    - `kind`
    - `height`
    - `ratio`
    - `space`
    - `dropna`
    - `xlim`
    - `ylim`
    - `color`
    - `palette`
    - `hue_order`
    - `marginal_ticks`
    - `joint_kws`
    - `marginal_kws`
    - `annot_kws`
  - Jointplot best practices

- **49. Faceting**
  - Faceting
  - `col`
  - `row`
  - `col_wrap`
  - `hue`
  - Faceting best practices

---

# VIII. Customization

- **50. Aesthetics**
  - Aesthetics
  - Aesthetic mappings
  - Semantic mappings
  - Aesthetic best practices

- **51. Themes**
  - `set_theme()`
  - `set_style()`
  - `set_context()`
  - `set_palette()`
  - `reset_defaults()`
  - `reset_orig()`
  - Themes
    - `darkgrid`
    - `whitegrid`
    - `dark`
    - `white`
    - `ticks`
  - Contexts
    - `paper`
    - `notebook`
    - `talk`
    - `poster`
  - Theme best practices

- **52. Color Palettes**
  - Color palettes
  - `color_palette()`
  - `set_palette()`
  - Qualitative palettes
    - `deep`
    - `muted`
    - `pastel`
    - `bright`
    - `dark`
    - `colorblind`
  - Sequential palettes
    - `Blues`
    - `Greens`
    - `Reds`
    - `Oranges`
    - `Purples`
    - `Greys`
    - `BuGn`
    - `BuPu`
    - `GnBu`
    - `OrRd`
    - `PuBu`
    - `PuBuGn`
    - `PuRd`
    - `RdPu`
    - `YlGn`
    - `YlGnBu`
    - `YlOrBr`
    - `YlOrRd`
  - Diverging palettes
    - `RdBu`
    - `RdGy`
    - `RdYlBu`
    - `RdYlGn`
    - `Spectral`
    - `coolwarm`
    - `bwr`
    - `seismic`
  - Custom palettes
  - Palette best practices

- **53. Figure Aesthetics**
  - Figure size
  - Figure DPI
  - Figure background
  - Figure titles
  - Figure best practices

- **54. Axes Aesthetics**
  - Axes titles
  - Axes labels
  - Axes limits
  - Axes ticks
  - Axes spines
  - Axes grid
  - Axes best practices

- **55. Legend Customization**
  - Legend location
  - Legend title
  - Legend labels
  - Legend frame
  - Legend best practices

- **56. Annotations**
  - Text annotations
  - Arrow annotations
  - Annotation best practices

- **57. Despining**
  - Despine
  - `despine()`
  - Despine best practices

---

# IX. Advanced Topics

- **58. Statistical Estimation**
  - Statistical estimation
  - Estimators
  - Bootstrapping
  - Confidence intervals
  - Error bars
  - Statistical estimation best practices

- **59. Aggregation**
  - Aggregation
  - `estimator`
  - `errorbar`
  - `n_boot`
  - Aggregation best practices

- **60. Weighted Data**
  - Weighted data
  - `weights`
  - Weighted best practices

- **61. Log Scales**
  - Log scales
  - `log_scale`
  - Log scale best practices

- **62. Categorical Ordering**
  - Categorical ordering
  - `order`
  - `hue_order`
  - Ordering best practices

- **63. Pandas Integration**
  - Pandas DataFrames
  - Pandas Series
  - Long-form data
  - Wide-form data
  - Pandas integration best practices

- **64. NumPy Integration**
  - NumPy arrays
  - NumPy integration best practices

- **65. Matplotlib Integration**
  - Matplotlib integration
  - Axes-level functions
  - Figure-level functions
  - Matplotlib integration best practices

- **66. Subplots**
  - Subplots
  - `plt.subplots()`
  - Axes-level functions
  - Subplot best practices

- **67. Custom Functions**
  - Custom plotting functions
  - Custom functions best practices

- **68. Statistical Models**
  - Statistical models
  - statsmodels
  - Linear models
  - Statistical models best practices

---

# X. Performance

- **69. Performance Fundamentals**
  - Performance
  - Rendering time
  - Memory usage
  - Performance metrics
  - Performance best practices

- **70. Large Datasets**
  - Large datasets
  - Downsampling
  - Aggregation
  - Data reduction
  - Large dataset best practices

- **71. Rendering Performance**
  - Rendering
  - Backend selection
  - Agg backend
  - Performance optimization
  - Rendering best practices

- **72. Memory Optimization**
  - Memory optimization
  - Figure cleanup
  - `plt.close()`
  - Memory best practices

- **73. Profiling**
  - Profiling
  - `cProfile`
  - `line_profiler`
  - `memory_profiler`
  - Profiling best practices

- **74. Benchmarking**
  - Benchmarking
  - `timeit`
  - `%timeit`
  - Benchmarking best practices

---

# XI. Export and Publication

- **75. Saving Figures**
  - `savefig()`
  - File formats
    - PNG
    - PDF
    - SVG
    - EPS
    - JPEG
    - TIFF
  - DPI
  - Figure size
  - Bounding box
  - Saving best practices

- **76. Publication-Quality Figures**
  - Publication quality
  - DPI
  - Font sizes
  - Line widths
  - Figure size
  - Color schemes
  - Publication best practices

- **77. LaTeX Integration**
  - LaTeX
  - `usetex`
  - LaTeX rendering
  - LaTeX best practices

- **78. Vector Graphics**
  - SVG
  - PDF
  - EPS
  - Vector graphics best practices

---

# XII. Seaborn Projects by Difficulty

## Beginner Projects

- **1. Tips Dataset Analysis**
  - Dataset loading
  - Scatter plot
  - Histogram
  - Box plot
  - Bar plot

- **2. Iris Dataset Visualization**
  - Pairplot
  - Scatter plot
  - Box plot
  - Violin plot

- **3. Titanic Survival Analysis**
  - Count plot
  - Bar plot
  - Distribution plot
  - Categorical plot

- **4. Flights Dataset Visualization**
  - Heatmap
  - Line plot
  - Pivot table
  - Time series

- **5. Penguins Dataset Analysis**
  - Pairplot
  - Scatter plot
  - Box plot
  - Distribution plot

---

## Intermediate Projects

- **6. Exploratory Data Analysis**
  - Multiple plots
  - Faceting
  - Customization
  - Themes
  - Color palettes

- **7. Statistical Visualization**
  - Regression plots
  - Residual plots
  - Distribution plots
  - Correlation matrices

- **8. Time Series Visualization**
  - Line plots
  - Faceting
  - Aggregation
  - Confidence intervals

- **9. Categorical Data Analysis**
  - Box plots
  - Violin plots
  - Bar plots
  - Point plots

- **10. Correlation Analysis**
  - Heatmaps
  - Clustermaps
  - Pair plots
  - Joint plots

---

## Advanced Projects

- **11. Publication-Quality Figures**
  - Themes
  - Color palettes
  - Customization
  - Export
  - LaTeX

- **12. Dashboard Visualization**
  - Multi-plot grids
  - Faceting
  - Customization
  - Interactivity

- **13. Machine Learning Visualization**
  - Feature importance
  - Confusion matrix
  - ROC curve
  - Learning curves

- **14. Financial Data Visualization**
  - Time series
  - Candlestick
  - Correlation
  - Risk analysis

- **15. Scientific Data Visualization**
  - Heatmaps
  - Clustermaps
  - Distribution plots
  - Regression plots

---

## Expert Projects

- **16. Custom Visualization Library**
  - Custom functions
  - Custom themes
  - Custom palettes
  - Documentation
  - Testing

- **17. Interactive Dashboard**
  - Seaborn
  - Plotly
  - Dash
  - Streamlit
  - Deployment

- **18. Automated Reporting**
  - Seaborn
  - Pandas
  - Jupyter
  - Papermill
  - Automation

- **19. High-Performance Visualization**
  - Large datasets
  - Downsampling
  - Aggregation
  - Performance optimization

- **20. Production Visualization Platform**
  - Data pipelines
  - Visualization
  - Reporting
  - Deployment
  - Monitoring

---

# XIII. Progressive Seaborn Learning Sequence

## Level 1 — Seaborn Fundamentals

- Master:
  - Installation
  - Import
  - API
  - First plot
  - Built-in datasets

## Level 2 — Relational Plots

- Master:
  - Relational plot fundamentals
  - Scatter plots
  - Line plots
  - Relational plot customization
  - Relational plot best practices

## Level 3 — Distribution Plots

- Master:
  - Distribution plot fundamentals
  - Histograms
  - KDE plots
  - ECDF plots
  - Rug plots
  - Bivariate distributions
  - Distribution plot customization

## Level 4 — Categorical Plots

- Master:
  - Categorical plot fundamentals
  - Strip plots
  - Swarm plots
  - Box plots
  - Violin plots
  - Boxen plots
  - Point plots
  - Bar plots
  - Count plots
  - Categorical plot customization

## Level 5 — Regression Plots

- Master:
  - Regression plot fundamentals
  - Linear regression plots
  - Polynomial regression
  - Logistic regression
  - LOWESS regression
  - Robust regression
  - Residual plots
  - Regression plot customization

## Level 6 — Matrix Plots

- Master:
  - Matrix plot fundamentals
  - Heatmaps
  - Clustermaps
  - Correlation matrices
  - Pivot tables

## Level 7 — Multi-Plot Grids

- Master:
  - Multi-plot grid fundamentals
  - FacetGrid
  - PairGrid
  - JointGrid
  - Pairplot
  - Jointplot
  - Faceting

## Level 8 — Customization

- Master:
  - Aesthetics
  - Themes
  - Color palettes
  - Figure aesthetics
  - Axes aesthetics
  - Legend customization
  - Annotations
  - Despining

## Level 9 — Advanced Topics

- Master:
  - Statistical estimation
  - Aggregation
  - Weighted data
  - Log scales
  - Categorical ordering
  - Pandas integration
  - NumPy integration
  - Matplotlib integration
  - Subplots
  - Custom functions
  - Statistical models

## Level 10 — Performance

- Master:
  - Performance fundamentals
  - Large datasets
  - Rendering performance
  - Memory optimization
  - Profiling
  - Benchmarking

## Level 11 — Export and Publication

- Master:
  - Saving figures
  - Publication-quality figures
  - LaTeX integration
  - Vector graphics

## Level 12 — Production Engineering

- Master:
  - Visualization pipelines
  - Automated reporting
  - Dashboard integration
  - Accessibility
  - Reproducibility
  - Production best practices

---

# XIV. Final Seaborn Competency Map

- **Foundations**

  - Installation
  - Import
  - API
  - First plot
  - Built-in datasets

- **Relational Plots**

  - Relational plot fundamentals
  - Scatter plots
  - Line plots
  - Relational plot customization

- **Distribution Plots**

  - Distribution plot fundamentals
  - Histograms
  - KDE plots
  - ECDF plots
  - Rug plots
  - Bivariate distributions
  - Distribution plot customization

- **Categorical Plots**

  - Categorical plot fundamentals
  - Strip plots
  - Swarm plots
  - Box plots
  - Violin plots
  - Boxen plots
  - Point plots
  - Bar plots
  - Count plots
  - Categorical plot customization

- **Regression Plots**

  - Regression plot fundamentals
  - Linear regression plots
  - Polynomial regression
  - Logistic regression
  - LOWESS regression
  - Robust regression
  - Residual plots
  - Regression plot customization

- **Matrix Plots**

  - Matrix plot fundamentals
  - Heatmaps
  - Clustermaps
  - Correlation matrices
  - Pivot tables

- **Multi-Plot Grids**

  - Multi-plot grid fundamentals
  - FacetGrid
  - PairGrid
  - JointGrid
  - Pairplot
  - Jointplot
  - Faceting

- **Customization**

  - Aesthetics
  - Themes
  - Color palettes
  - Figure aesthetics
  - Axes aesthetics
  - Legend customization
  - Annotations
  - Despining

- **Advanced Topics**

  - Statistical estimation
  - Aggregation
  - Weighted data
  - Log scales
  - Categorical ordering
  - Pandas integration
  - NumPy integration
  - Matplotlib integration
  - Subplots
  - Custom functions
  - Statistical models

- **Performance**

  - Performance fundamentals
  - Large datasets
  - Rendering performance
  - Memory optimization
  - Profiling
  - Benchmarking

- **Export**

  - Saving figures
  - Publication-quality figures
  - LaTeX integration
  - Vector graphics

- **Production**

  - Visualization pipelines
  - Automated reporting
  - Dashboard integration
  - Accessibility
  - Reproducibility

---

## Recommended Overall Progression

**Seaborn Fundamentals → Relational Plots → Distribution Plots → Categorical Plots → Regression Plots → Matrix Plots → Multi-Plot Grids → Customization → Advanced Topics → Performance → Export and Publication → Production Engineering**
