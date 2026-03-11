# data_harvesting_project
Data Harvesting Project: Media Agenda and International Politics in Spanish Newspapers
# Data Harvesting: International Coverage in Spanish Newspapers

Final project for the **Data Harvesting** course — MSc in Computational Social Science, UC3M.

This project scrapes the international sections of four major Spanish newspapers and consolidates the data into a single unified dataset for comparative analysis of international news coverage.

---

## Project Goals

- Collect article metadata (title, author, date, URL) from the international sections of **eldiario.es**, **El Mundo**, **El País**, and **La Vanguardia**.
- Enrich each article with: city of writing origin, countries mentioned, political figures mentioned, and number of reader comments.
- Merge all four datasets into a single CSV file with a unique `article_id` and a `newspaper` identifier for cross-outlet analysis.

---

## Repository Structure

```
.
├── newspapers_final.qmd          # Main script — all scrapers + merge + EDA
├── README.md                     # This file
└── all_newspapers_DD_MM_YYYY.csv # Output: unified dataset (generated on run)
```

The `.qmd` file is organized into **five sections**, executed sequentially:

| Section | Newspaper | URL scraped |
|---|---|---|
| 1 | eldiario.es | `eldiario.es/internacional/` |
| 2 | El Mundo | `elmundo.es/internacional.html` |
| 3 | El País | `elpais.com/internacional/` |
| 4 | La Vanguardia | `lavanguardia.com/internacional` |
| 5 | **Merge + EDA** | — |

After each newspaper section, the environment is cleaned with `rm(list = setdiff(ls(), <keep>)); gc()`, retaining only the completed dataframes.

---

## How to Reproduce

### 1. Prerequisites

You need the following software installed on your machine:

| Requirement | Version | Notes |
|---|---|---|
| **R** | ≥ 4.1 | [Download from CRAN](https://cran.r-project.org/) |
| **RStudio** | any recent | Recommended IDE |
| **Java** | ≥ 11 | Required by the Selenium server (a `.jar` file that runs on the JVM). [Download from Oracle](https://www.oracle.com/java/technologies/downloads/) |
| **Google Chrome** | any recent | The browser driven by Selenium to scrape JavaScript-rendered comment counts |

> Java is a hard requirement — `selenium_server()` will fail silently or throw an error if Java is not installed or not on your system PATH. You can verify your Java installation by running `java -version` in a terminal.

Install all required R packages:

```r
install.packages(c(
  "rvest", "xml2", "httr", "scrapex",
  "dplyr", "tidyr", "stringr", "purrr",
  "tibble", "readr", "ggplot2",
  "countrycode", "selenium", "DataExplorer"
))
```

### 2. Clone the repository

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
```

### 3. Run the scraper

Open `newspapers_final.qmd` in RStudio and click **Render**, or run from the terminal:

```bash
quarto render newspapers_final.qmd
```

This executes all five sections in order. **Expected runtime: 30–60 minutes**, due to per-article HTTP requests, `Sys.sleep()` delays between requests (included to avoid server overload), and three separate Selenium sessions for comment scraping.

>  Do not remove the `Sys.sleep()` calls. They are intentional rate-limiting to respect the websites' servers.

### 4. Output

A single CSV file is saved to your working directory:

```
all_newspapers_DD_MM_YYYY.csv
```

where `DD_MM_YYYY` is today's date (e.g. `all_newspapers_11_03_2026.csv`).

---

## Output: Data Dictionary

Each row represents one article. The columns are:

| Column | Description |
|---|---|
| `article_id` | Sequential unique identifier across all newspapers (integer) |
| `newspaper` | Source outlet: `eldiario.es`, `elmundo.es`, `elpais.com`, `lavanguardia.com` |
| `title` | Article headline as it appears on the section front page |
| `author` | Author name(s); `NA` if not listed |
| `url` | Full URL of the article |
| `date` | Publication date in `DD/MM/YYYY` format |
| `city_writing_origin` | City from which the piece was filed; `NA` if unavailable |
| `countries_mentioned` | Comma-separated list of countries detected in the article body |
| `politicians_mentioned` | Comma-separated list of tracked political figures mentioned |
| `n_comments` | Number of reader comments at time of scraping; `NA` if unavailable |

**Tracked political figures:** Pedro Sánchez, Abascal, Donald Trump, Von der Leyen, Netanyahu, Zelenski, Putin, Milei, Maduro.

>  The list of tracked political figures can be easily customised. In each newspaper section, locate the character vector defined before the scraping function — for example:
> ```r
> politicians <- c("Pedro Sánchez", "Abascal", "Donald Trump", ...)
> ```
> Add, remove, or replace names as needed. No other changes to the code are required — the regex pattern is built dynamically from that vector.

**Country detection** uses the full Spanish-language country name list from the `countrycode` package (`cldr.name.es`), matched with word-boundary regex against the article body text.

---

## Potential Uses of This Dataset

This dataset was built to support research on **media agenda-setting and international politics in the Spanish press**. Below are the research questions it is designed to address, as well as broader analytical possibilities.

### Core research questions

- **Country salience** — Which countries dominate international political coverage in Spanish newspapers? Are some regions systematically underrepresented?
- **Geographical and cultural proximity bias** — Does proximity influence coverage? For example, is there disproportionate attention to Latin America or the Maghreb relative to other regions?
- **Eurocentric bias** — Does the Spanish press systematically favour European affairs over coverage of the Global South?
- **Cross-outlet agenda differences** — Do newspapers with different editorial lines diverge in which countries or conflicts they cover? Are there stories that some outlets consistently ignore?
- **Author and format effects** — Do certain article formats or author profiles generate higher country visibility or more in-depth coverage?
- **Event follow-up** — After a major event (war, revolution, climate disaster, corruption case), how many articles are published on average? Is there sustained follow-up coverage or a one-off spike?

### Additional analytical directions

- **Comment engagement vs. country covered** — Are articles about certain countries or regions more likely to generate reader discussion? Is there a relationship between geopolitical proximity and audience engagement?
- **Political figures and country co-occurrence** — Which political figures are most associated with which countries in coverage? Are certain leaders systematically linked to particular narrative frames?
- **City of writing origin as a proxy for correspondent networks** — Which newspapers have the broadest network of foreign correspondents? Which regions are covered remotely vs. on the ground?
- **Temporal analysis** — How does the volume of coverage of a specific country evolve over time? Can spikes be linked to specific events?
- **Comparative replication** — The scraping pipeline is fully reproducible and can be re-run on a different date to build a longitudinal panel dataset, enabling before/after comparisons around major geopolitical events.

---

## Notes per Newspaper

- **eldiario.es** — The front page mixes articles from other sections; only URLs containing `/internacional/` are kept. Date, city, countries, figures, and comments are all retrieved in a single per-article scraping pass.
- **El Mundo** — Dates are extracted from the `data-publish` attribute; articles without it fall back to parsing the date from the URL. The `kicker` column is dropped before merging. Comments are retrieved via Selenium.
- **El País** — Dates are extracted directly from the URL (`YYYY-MM-DD` format). Comments use the Disqus `span.disqus-comment-count` element and require Selenium. Duplicate URLs on the front page should be deduplicated with `distinct(ref, .keep_all = TRUE)` before joining.
- **La Vanguardia** — Dates are encoded as an 8-digit string (`YYYYMMDD`) in the URL. Comments use the `#social-comments-count` element and require Selenium.

---

## 👥 Authors
Carlos Morán García-Quijada, Paloma Navarro, Gaspar Vigneaux Poirot
MSc in Computational Social Science — Universidad Carlos III de Madrid (UC3M), 2025–2026.
