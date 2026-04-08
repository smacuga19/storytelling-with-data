---
marp: true
theme: cdl-theme
math: katex
transition: fade 0.25s
author: Contextual Dynamics Lab
---

# Data visualization: approaches to creating effective figures
### PSYC 81.09: Storytelling with Data

Jeremy R. Manning
Dartmouth College
Spring 2026

---

## Why visualize data?

<div class="important-box" data-title="The power of vision">

Our visual systems **rapidly process massive amounts of information** and are adept at **pattern recognition**. We can leverage this to convey patterns in data — but only if we figure out how to **turn data into the right pictures**.

A good visualization can reveal what no table of numbers ever could. A bad visualization can obscure or mislead.

</div>

<div class="note-box" data-title="Key references">

- Edward Tufte, [*The Visual Display of Quantitative Information*](https://www.edwardtufte.com/tufte/books_vdqi) — the classic on data-ink ratio and visual clarity
- [Designing Effective Scientific Figures](https://www.dropbox.com/s/qdiqqt3a8i632hn/DesigningEffectiveScientificFigures_Zabala_afternoon_v00.pdf) by Aiora Zabala
- Hadley Wickham, [A Layered Grammar of Graphics](https://www.dropbox.com/s/xhpjth2f4aamn5u/layered-grammar.pdf)

</div>

---
<!-- _class: scale-90 -->

# Which representation is clearest?

![height:400px](figs/data_representations.svg)

---

## Anscombe's quartet

<div class="warning-box" data-title="Statistics can deceive">

These four datasets have **identical** summary statistics (same mean, variance, correlation, and regression line) — but look completely different when plotted!

</div>

![width:1150px](figs/anscombes_quartet.svg)

---

## The most important question in data visualization

<div class="important-box" data-title="Always ask yourself:">

**What message do you want your audience to take away?**

Every visualization decision — color, layout, chart type, annotation — should serve that message. **Start with the message, then choose the visualization.**

</div>

<div class="tip-box" data-title="A common mistake">

Many people pick a chart type first, then try to fit their data into it. Instead, decide what you want to *say*, then find the visualization that says it most clearly.

</div>

---
<!-- _class: scale-90 -->

## Design principles

<div class="tip-box" data-title="Guidelines for effective figures">

- **Data-to-ink ratio** ([Tufte](https://www.edwardtufte.com/tufte/books_vdqi)): maximize the share of ink devoted to data; minimize non-data ink (gridlines, borders, decorations)
- **Clarity over cleverness**: if the audience has to work to understand your figure, simplify it
- **Consistent encoding**: same color = same meaning throughout your presentation
- **Accessible design**: use [colorblind-friendly palettes](https://www.nature.com/articles/nmeth.1618); don't rely on color alone
- **Label everything**: axes, units, legends — if someone can't understand the figure without your narration, add more labels
- **Be willing to break all of the rules!** Sometimes the most effective figure violates a guideline

</div>

---

<!-- _class: scale-90 -->

## Grammar of graphics: figures are built from layers

![height:380px](figs/grammar_of_graphics.svg)

*Based on Wickham's [A Layered Grammar of Graphics](https://www.dropbox.com/s/xhpjth2f4aamn5u/layered-grammar.pdf) and Wilkinson's [The Grammar of Graphics](https://www.dropbox.com/s/4qwd16psogqdgi6/Wilk10.pdf)*

---
<!-- _class: scale-90 -->

## Choosing the right visualization

<div class="tip-box" data-title="Match the chart to your data and message">

| If you want to... | Consider... |
|-|-|
| **Compare categories** | Bar chart, grouped bar, stacked bar |
| **Show distributions** | Histogram, density plot, violin, box plot, strip plot |
| **Reveal relationships** | Scatter plot, bubble chart, heatmap, pair plot |
| **Track change over time** | Line plot, area chart, sparkline |
| **Show composition** | Pie chart, stacked area, treemap |
| **Display spatial data** | Choropleth map, bubble map |
| **Show connections** | Network graph, chord diagram, Sankey diagram |
| **Add a dimension** | Animation, small multiples, 3D projection |

</div>

---

# Chart gallery
### For each chart type, ask: *what kind of data does this show, and what story does it tell?*

---

# Comparisons
### Charts for comparing values across categories

---

## Bar chart: comparing values across categories

```chart
type: bar
labels: Spring, Summer, Autumn, Winter
data: 42, 78, 55, 31
caption: Average daily visitors by season
ylabel: Visitors
palette: cdl
height: 380px
```

---
<!-- _class: scale-90 -->

## Grouped bar chart: comparing subcategories

```chart
type: bar
labels: 2022, 2023, 2024, 2025
datasets:
  - label: Undergrad
    data: 320, 345, 410, 390
  - label: Graduate
    data: 85, 92, 105, 110
  - label: Faculty
    data: 45, 48, 50, 52
palette: cdl
caption: Course enrollment by year and group
ylabel: Students
height: 340px
```

---

## Multi-series bar chart: showing composition within categories

```chart
type: bar
labels: Fiction, Non-fiction, Science, History
datasets:
  - label: Print
    data: 150, 200, 80, 120
  - label: Digital
    data: 250, 180, 160, 90
  - label: Audio
    data: 100, 60, 40, 30
palette: cdl
caption: Book sales by format and genre (thousands)
ylabel: Sales (K)
height: 340px
```

---

# Distributions
### Charts for showing how values are spread

---

## Histogram: showing the shape of a distribution

```chart
type: bar
labels: 0-10, 10-20, 20-30, 30-40, 40-50, 50-60, 60-70, 70-80, 80-90, 90-100
data: 2, 5, 12, 25, 35, 30, 20, 10, 6, 3
caption: Distribution of exam scores (n = 148)
xlabel: Score
ylabel: Count
palette: cdl
height: 340px
```

<div class="note-box" data-title="Histograms vs. bar charts">

Histograms show **continuous** data binned into ranges. Bar charts show **categorical** data. The x-axis ordering matters in histograms but not in bar charts.

</div>

---

## Box plot: summarizing distributions with quartiles

<div class="definition-box" data-title="Anatomy of a box plot">

- **Box**: 25th to 75th percentile (interquartile range)
- **Line inside box**: median (50th percentile)
- **Whiskers**: extend to 1.5× IQR (or min/max)
- **Points beyond whiskers**: outliers

Box plots are great for **comparing distributions** across groups at a glance.

</div>

<div class="tip-box" data-title="When to use">

Use box plots when you need a compact summary of spread and central tendency across many groups. For more detail, consider violin plots or strip plots.

</div>

---

## Violin plot: distributions with density shape

<div class="definition-box" data-title="Box plot + density estimate">

A violin plot is a box plot with a **kernel density estimate** mirrored on each side, showing the full shape of the distribution — not just quartiles.

- **Wide sections** = many data points at that value
- **Narrow sections** = few data points
- Reveals **multimodality** that box plots hide

</div>

<div class="example-box" data-title="Example">

A violin showing exam scores might reveal two peaks (bimodal) — students who studied vs. those who didn't — while a box plot would just show one box.

</div>

---

# Relationships
### Charts for showing how variables relate

---

## Scatter plot: revealing relationships between two variables

<div class="definition-box" data-title="Two continuous variables">

A scatter plot shows the relationship between two continuous variables. Each **point** represents one observation.

- **Positive correlation**: points trend upward (more X → more Y)
- **Negative correlation**: points trend downward (more X → less Y)
- **No correlation**: points are scattered randomly
- Add a **trend line** to show the overall direction

</div>

<div class="warning-box" data-title="Correlation ≠ causation">

A trend in a scatter plot shows an **association**, not proof that one variable *causes* the other. Always consider confounding variables!

</div>

---

## Bubble chart: scatter plot + a third variable

<div class="definition-box" data-title="Adding a dimension with size">

A bubble chart is a scatter plot where each point's **size** encodes a third variable. This lets you show three dimensions of data in a 2D plot.

- x-axis: one variable
- y-axis: another variable
- Bubble size: a third variable (e.g., population, revenue, sample size)

The animated [Gapminder](https://www.gapminder.org/) visualization (coming up!) is a famous example.

</div>

---

## Heatmap: showing magnitude in a matrix

<div class="definition-box" data-title="A table where numbers become colors">

A heatmap displays values in a **grid** using **color intensity** instead of numbers. Darker or more saturated colors represent higher (or lower) values.

- **Rows and columns** represent two categorical or ordinal variables
- **Color** encodes the magnitude of each cell
- Great for: correlation matrices, gene expression, time-of-day patterns, confusion matrices

</div>

<div class="example-box" data-title="You've already seen one!">

The "which representation is clearest?" slide earlier was a heatmap — pixel brightness encoded the face data. The heatmap revealed a pattern that raw numbers could not!

</div>

---

## Pair plot: all pairwise relationships at once

<div class="definition-box" data-title="A matrix of scatter plots">

A pair plot (or scatter plot matrix) shows **every pairwise combination** of variables in a dataset:

- **Diagonal**: distribution of each variable (histogram or density)
- **Off-diagonal**: scatter plot of each pair

Pair plots are excellent for **exploratory data analysis** — quickly spotting which variables are related before diving deeper.

</div>

<div class="tip-box" data-title="When to use">

Best for datasets with 3–8 variables. More than that and the matrix becomes too dense to read. Use vibe coding to generate these — the syntax is one line: `sns.pairplot(df)`.

</div>

---

# Change over time
### Charts for showing trends and temporal patterns

---

## Line plot: tracking change over time

```chart
type: line
labels: Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec
datasets:
  - label: 2024
    data: 12, 15, 22, 28, 35, 42, 45, 43, 38, 28, 18, 14
  - label: 2025
    data: 14, 18, 25, 32, 40, 48, 50, 47, 40, 30, 20, 15
palette: cdl
caption: Average temperature (°C) by month
xlabel: Month
ylabel: Temperature (°C)
height: 340px
```

---

## Area chart: emphasizing cumulative magnitude

```chart
type: line
labels: 2020, 2021, 2022, 2023, 2024, 2025
datasets:
  - label: Solar
    data: 10, 18, 30, 45, 62, 85
  - label: Wind
    data: 25, 32, 38, 45, 55, 68
  - label: Hydro
    data: 40, 42, 43, 44, 45, 46
palette: cdl
alpha: 0.4
caption: Renewable energy capacity (GW) — area charts emphasize cumulative growth
ylabel: Capacity (GW)
height: 320px
```

---

# Composition
### Charts for showing parts of a whole

---

## Pie chart: parts of a whole (use sparingly!)

```chart
type: pie
labels: Python, R, SQL, Julia, Other
data: 45, 20, 18, 7, 10
caption: Programming languages used in data science (2025 survey)
palette: cdl
height: 350px
```

<div class="warning-box" data-title="Pie charts are controversial">

Humans are bad at comparing **angles and areas**. Bar charts are almost always more readable. Use pie charts only when you have **few categories** (≤5) and want to emphasize that they **sum to 100%**.

</div>

---

## Doughnut chart: a modern alternative to pie

```chart
type: doughnut
labels: Tuition, Research, Grants, Donations, Other
data: 35, 25, 20, 12, 8
caption: University revenue sources (%)
palette: cdl
height: 350px
```

---

# Spatial data
### Charts for showing geographic patterns

---

## Choropleth map: coloring regions by value

<div class="definition-box" data-title="Geographic heatmap">

A choropleth map colors geographic regions (countries, states, counties) by a data value. The color intensity represents magnitude.

- Great for showing **regional variation** (election results, income levels, disease prevalence)
- Can be misleading: large but sparsely populated areas dominate visually
- Consider **bubble maps** (sized circles on a map) as an alternative

</div>

![height:300px](figs/cloropleth.png)

---

# Connections
### Charts for showing relationships between entities

---

## Network graph: showing connections

<div class="definition-box" data-title="Nodes and edges">

Network graphs (or graphs) show **entities** (nodes) and **relationships** (edges) between them.

- **Node size** can encode importance (e.g., number of connections)
- **Edge thickness** can encode strength of relationship
- **Color** can encode categories or communities
- Layout algorithms (force-directed, hierarchical) position nodes to reveal structure

</div>

<div class="example-box" data-title="Applications">

Social networks, citation networks, brain connectivity, gene regulatory networks, web link structure, transportation systems.

</div>

---

# Animation
### Adding a temporal dimension to any chart

---

## Animation: life expectancy vs. GDP over time

<div class="note-box" data-title="Hans Rosling's Gapminder">

This famous visualization — popularized by [Hans Rosling](https://www.gapminder.org/) — shows how life expectancy and GDP per capita have changed across countries from 1952 to 2007. Bubble size represents population; color represents continent.

</div>

![height:400px](figs/gapminder.gif)

*Animated scatter plot built with [Plotly](https://plotly.com/python/) using the [Gapminder](https://www.gapminder.org/data/) dataset.*

---
<!-- _class: scale-90 -->

## General tips and tricks

<div class="tip-box" data-title="Putting it all together">

- **Start with the question**, not the chart type — what do you want the audience to learn?
- Apply Tufte's **data-to-ink ratio** — remove anything that doesn't convey information
- Use **consistent color schemes** across related figures
- **Label axes and units** — never make the audience guess
- Consider your audience — what level of detail do they need?
- **Iterate!** Your first visualization is rarely your best — refine, simplify, polish
- Use [colorblind-friendly palettes](https://www.nature.com/articles/nmeth.1618) — about 8% of men have color vision deficiency
- When in doubt, **ask someone else** if they can understand your figure without your explanation

</div>

---

# Questions? Want to chat more?

<div class="emoji-figure">
  <div class="emoji-col">
    <span class="emoji emoji-xl emoji-bg emoji-bg-navy">&#x1F4E7;</span>
    <span class="label"><a href="mailto:jeremy@dartmouth.edu">Email</a> me</span>
  </div>
  <div class="emoji-col">
    <span class="emoji emoji-xl emoji-bg emoji-bg-purple">&#x1F4AC;</span>
    <span class="label">Join our <a href="https://stories-about-data.slack.com">Slack</a></span>
  </div>
  <div class="emoji-col">
    <span class="emoji emoji-xl emoji-bg emoji-bg-green">&#x1F481;</span>
    <span class="label">Come to <a href="https://context-lab.com/scheduler">office hours</a></span>
  </div>
</div>

<div class="note-box" data-title="Up next...">

- **Thursday:** [Introduction to vibe coding](vibe-coding.html) — learn to use AI tools to create visualizations like these!
- **Friday:** Workshop data story ideas + Assignment 2 release

</div>
