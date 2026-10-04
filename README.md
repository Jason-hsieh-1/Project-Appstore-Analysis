# App Store Analysis

An exploratory and statistical analysis of Apple App Store data in R. It looks at how app characteristics, pricing, and monetization relate to user satisfaction (ratings), to advise a company planning to publish new apps.

This is a university project (2023). The full analysis is in one R Markdown report: [`r_report.Rmd`](r_report.Rmd).

---

## Table of contents

- [Research questions](#research-questions)
- [Dataset](#dataset)
- [Approach](#approach)
- [Analyses and visualizations](#analyses-and-visualizations)
- [How to run](#how-to-run)
- [Repository structure](#repository-structure)
- [Known limitations and possible improvements](#known-limitations-and-possible-improvements)
- [License](#license)

---

## Research questions

The analysis asks which factors relate to an app's **user rating**:

- Do **paid** apps get better ratings than **free** apps? Does price matter?
- Which **app categories** have the most satisfied users, and which are crowded?
- Does App Store presentation, such as the **number of screenshots**, relate to ratings?
- Do **supported devices**, **supported languages**, or **app size** matter?
- How do ratings change between an app's **overall** rating and its **current version**? Does that differ by category and monetization?

---

## Dataset

The data file `apple_data.csv` (semicolon-delimited) is **not included in this repository**. Its columns match the public *Mobile App Store* dataset of about 7,200 iOS apps on Kaggle.

| Column | Description |
|---|---|
| `id`, `track_name` | App identifier and name |
| `size_bytes` | App size in bytes |
| `currency`, `price` | Price (all USD) |
| `rating_count_tot`, `rating_count_ver` | Number of ratings: all versions / current version |
| `user_rating`, `user_rating_ver` | Average rating: all versions / current version |
| `ver` | Current version |
| `cont_rating` | Content rating (4+, 9+, 12+, 17+) |
| `prime_genre` | Primary App Store genre |
| `sup_devices.num` | Number of supported devices |
| `ipadSc_urls.num` | Number of screenshots shown in the store |
| `lang.num` | Number of supported languages |
| `vpp_lic` | Volume Purchase Program licensing enabled |

---

## Approach

```
apple_data.csv ──► Cleaning ──► Feature creation & genre clustering ──► Outlier removal ──► Descriptive analysis ──► Correlation / regression / t-test ──► Per-category deep dives
```

### 1. Cleaning
- Drops apps with a missing price
- Sets apps that list 0 supported languages to 1
- Checks for duplicate IDs (none found) and confirms all prices are in USD
- Keeps only apps with **at least 5 ratings**, so averages aren't based on a handful of reviews

### 2. Feature engineering
- **`new_category`**: the 23 App Store genres grouped into 10 broader categories:

  | New category | Genres |
  |---|---|
  | Entertainment | Games, Music, Entertainment, Photo & Video |
  | Productivity | Productivity, Business, Finance |
  | Communication | Social Networking |
  | Utilities | Utilities, Weather |
  | Commerce | Shopping, Catalogs |
  | Educational | Education, Reference |
  | Mobility | Travel, Navigation |
  | Health | Health & Fitness, Medical |
  | Lifestyle | Lifestyle, Sports, Food & Drink |
  | Information | News, Book |

- **`size_mb`**: app size in megabytes
- **`monetization`**: `free` (price = 0) or `pay`
- **`rating_dev`**: whether the current version's rating `increased`, stayed `constant`, or `decreased` compared with the all-time rating

### 3. Outlier removal
The top **1%** of apps by price, total ratings, current-version ratings, supported languages, and app size are removed. This produces the final analysis dataset (`apple_data_v3`).

---

## Analyses and visualizations

**Descriptive**
- Number of apps per genre and per category (bar and pie charts)
- Average rating, rating count, price, languages, screenshots, and devices per category
- Share of free vs. paid apps, overall and per category
- Price frequency and price vs. average rating
- Rating distributions: overall, per category (log scale), and paid vs. free
- Average rating per category against the overall mean, highlighting categories averaging above 4 stars
- Distribution of screenshot counts

**Statistical**
- **Correlation matrix** (`ggcorrplot`) of price, rating counts, ratings, devices, screenshots, languages, and size, with non-significant pairs marked
- **Multiple linear regression** of `user_rating` on price, total rating count, supported devices, screenshots, languages, monetization, and an *update-frequency proxy*: the share of all ratings that come from the current version
- **Welch two-sample t-test** comparing ratings of apps with **0 vs. 5 screenshots**

**Per-category deep dives**
- Screenshots vs. rating (Entertainment)
- Price vs. rating, total rating count vs. rating, and supported devices vs. rating, faceted by category with linear trend lines
- Rating box plots by category
- Average current-version rating count vs. % rating change in the new version, by category and monetization

> The repository contains only the `.Rmd` source, not a rendered report. Knit the file to see the numbers and charts.

---

## How to run

1. Place `apple_data.csv` (semicolon-delimited) in the same folder as `r_report.Rmd`.
2. Install the R packages:

```r
install.packages(c(
  "tidyverse", "ggcorrplot", "leaps", "gridExtra", "lattice", "reshape2",
  "GGally", "ggpubr", "stargazer", "ggridges", "coefplot", "ggdist", "lme4",
  "sjPlot", "psych", "dagitty", "ggdag", "funModeling"
))
```

3. Knit to PDF or Word in RStudio, or run:

```bash
Rscript -e 'rmarkdown::render("r_report.Rmd")'
```

> The plot theme uses the `palatino` font family. If it isn't installed, change `family = "palatino"` in `theme_plots` (and in the `geom_text` calls). Rendering to PDF also needs a LaTeX installation, such as `tinytex`.

---

## Repository structure

```
Project-Appstore-Analysis/
├── r_report.Rmd   # Full analysis: cleaning, transformation, visualization, statistics
├── README.md
├── LICENSE        # GNU GPL v3
└── .gitignore     # Standard R / RStudio ignores
```

---

## Known limitations and possible improvements

- **The t-test inflates its sample.** To make both groups the same length, the 0-screenshot ratings are repeated (`rep(...)`) and cut to 4,145 values. This copies the same observations many times, which shrinks the standard error and overstates significance. Welch's t-test doesn't need equal group sizes, so `t.test(zeroscreenshots$user_rating, fivescreenshots$user_rating)` on the original data is the correct comparison. A non-parametric test such as Wilcoxon would also suit the non-normal ratings.
- **Correlation plot y-axis labels are shifted.** In the correlation section, `labely` starts with "No. of Ratings (total)" and ends with "Price", while the variables start with `price`. Every y-axis label is therefore off by one. The `labely` vector should use the same order as `labelx`.
- **`View()` calls** throughout the report open data viewer windows and can break non-interactive knitting. They should be removed or commented out in the final report.
- **Regression specification.** The model is written with `apple_data_v3$...` and `updates_and_satisfaction$...` terms instead of a `data =` argument, which makes the output harder to read and breaks `predict()`. `price` and `monetization` also overlap heavily, which complicates interpreting either coefficient.
- **Ratings are bounded and clumped** at half-star steps between 0 and 5, so OLS assumptions are only approximately met. An ordinal model, or reporting robust standard errors, would make the conclusions stronger.
- **Unused libraries** (e.g. `lme4`, `dagitty`, `ggdag`, `leaps`, `psych`) are loaded but never used.
- **No rendered output** is committed. Adding the knitted PDF or HTML would let readers see the results without running the code.

---

## License

This project is licensed under the **GNU General Public License v3.0**. See [`LICENSE`](LICENSE) for details.
