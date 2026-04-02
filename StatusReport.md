# IS477 Interim Status Report: Do Critics Matter? Analyzing the Relationship Between Critic Scores and Movie Popularity

## Overview

This status report summarizes progress made since the submission of the project plan. Data acquisition, cleaning, and integration are now complete. The project has a fully merged dataset ready for exploratory analysis in the final milestone. Below we provide updates on each planned task, an updated timeline, a description of changes from the original plan, and a summary of challenges encountered.

---

## Task Updates

### ✅ Repository Setup
**Status: Complete**

The GitHub repository has been fully initialized with the required project structure, including a `.gitignore` to protect the API key, an `.env.example` file as a credential placeholder, a `LICENSE` (MIT), and a `requirements.txt` listing all project dependencies. The `datasets/` folder stores all raw and processed data files.

Artifacts: `.gitignore`, `.env.example`, `requirements.txt`, `LICENSE`, `ProjectPlan.md`

---

### ✅ Data Acquisition — Rotten Tomatoes (`movie_info.csv`)
**Status: Complete**

The Rotten Tomatoes dataset was downloaded from [r/datasets](https://www.reddit.com/r/datasets/) and stored unmodified at `datasets/movie_info.csv`. The dataset contains 12,413 rows and 5 columns: `title`, `url`, `release_date`, `critic_score`, and `audience_score`. Scores are stored as percentage strings (e.g., `"84%"`) and `release_date` uses two inconsistent formats, both of which were handled during the cleaning phase.

Artifact: `datasets/movie_info.csv`

---

### ✅ Data Acquisition — TMDB API (`tmdb_movies.csv`)
**Status: Complete**

TMDB movie metadata was collected using `tmdb_data_script.ipynb`, which queries the TMDB REST API `/search/movie` endpoint using titles from the RT dataset. The script collected **700 matched records** covering movies from approximately 1970–1973. The output was saved to `datasets/tmdb_movies.csv` with columns: `id`, `title`, `release_date`, `vote_average`, `vote_count`, `popularity`, and `overview`. Only 4 titles returned no TMDB match, representing a >99% match rate for titles attempted. The API key was loaded securely via `python-dotenv` and never committed to the repository.

Artifacts: `datasets/tmdb_movies.csv`, `tmdb_data_script.ipynb`

---

### ✅ Data Cleaning
**Status: Complete**

Both datasets have been cleaned in `analysis.ipynb`. The specific steps and outcomes for each dataset are documented below.

**Rotten Tomatoes (`movie_info.csv`):**
- Stripped `%` from `critic_score` and `audience_score` and cast both to numeric (`float64`)
- Extracted `release_year` from two inconsistent date formats using regex (`"Released Dec 16, 1970"` and bare `"1970"`)
- Normalized `title` into a `title_clean` column (lowercase, punctuation removed, whitespace collapsed)
- Dropped 3,114 rows with missing `critic_score` or unparseable `release_year`
- Deduplicated by keeping the highest `critic_score` entry per `title_clean` + `release_year`, removing 1,194 duplicate rows
- **Final RT clean shape: 8,105 rows × 7 columns**

**TMDB (`tmdb_movies.csv`):**
- Parsed `release_year` from `YYYY-MM-DD` format using `pd.to_datetime().dt.year`
- Normalized `title` into `title_clean` using the same function as above
- Dropped 2 rows with missing `release_date`
- Deduplicated on `id`, removing 101 duplicate TMDB records caused by repeated title queries in the fetch script
- **Final TMDB clean shape: 597 rows × 9 columns**

Artifact: `analysis.ipynb`, `datasets/rt_clean.csv`, `datasets/tmdb_clean.csv`

---

### ✅ Data Integration
**Status: Complete**

The two cleaned datasets were merged in `analysis.ipynb` using `pandas.merge()` with an inner join on `title_clean` and `release_year`. Original columns from both sources are retained in the merged output for provenance.

- **Merged shape: 255 rows × 12 columns**
- **Match rate vs TMDB clean: 42.7%** (255 of 597 unique TMDB records matched an RT entry)
- Only 3 rows have a missing `audience_score`; all other columns are fully populated
- Key columns in the merged dataset: `title`, `release_year`, `critic_score`, `audience_score`, `popularity`, `vote_count`, `vote_average`, `overview`

**Descriptive statistics of key analysis columns:**

| Statistic | critic_score | popularity | vote_count | vote_average |
|---|---|---|---|---|
| count | 255 | 255 | 255 | 255 |
| mean | 67.77 | 1.84 | 589.43 | 6.40 |
| std | 23.69 | 3.42 | 1685.09 | 0.74 |
| min | 0.00 | 0.16 | 3.00 | 4.11 |
| max | 100.00 | 41.09 | 22512.00 | 8.69 |

Artifact: `analysis.ipynb`, `datasets/merged_movies.csv`

---

### ⏳ Data Quality Assessment
**Status: Pending**

A formal data quality assessment will be conducted using the merged dataset. Planned checks include: completeness rates, duplicate documentation, match rate analysis, and outlier identification in `critic_score` and `popularity`.

---

### ⏳ Workflow Documentation
**Status: Pending**

End-to-end pipeline documentation will be written after the analysis phase is complete.

---

### ⏳ Exploratory Analysis
**Status: Pending**

Correlation analysis between `critic_score` and TMDB `popularity` and `vote_count` will be conducted in the final milestone using `merged_movies.csv`.

---

### ⏳ Metadata & Data Dictionary
**Status: Pending**

A data dictionary documenting all column definitions, types, and provenance notes will be written by Harlow once the final schema is confirmed.

---

### ⏳ Final Report
**Status: Pending**

The final project report in Markdown will be compiled collaboratively once analysis is complete.

---

## Updated Timeline

| Task | Description | Responsible | Status |
|---|---|---|---|
| Repository setup | Initialize GitHub repo, `.gitignore`, `.env.example`, folder structure | Flynn | ✅ Done |
| Data acquisition — RT | Load and inspect `movie_info.csv`, verify structure | Flynn | ✅ Done |
| Data acquisition — TMDB | Run `tmdb_data_script.ipynb` to collect 700 matched records via API | Flynn | ✅ Done |
| Data cleaning | Parse dates, strip score formatting, handle missing values, normalize titles | Flynn | ✅ Done |
| Data integration | Merge datasets on `title_clean` + `release_year` | Flynn | ✅ Done |
| Data quality assessment | Document completeness, duplicate rates, match rate, and outliers | Harlow | ⏳ Pending |
| Workflow documentation | Document end-to-end pipeline steps for reproducibility | Harlow | ⏳ Pending |
| Exploratory analysis | Correlation analysis, vote count distribution, decade-level trends | Both | ⏳ Pending |
| Metadata & data dictionary | Write column definitions and provenance notes | Harlow | ⏳ Pending |
| Final report | Compile and write full project report in Markdown | Both | ⏳ Pending |

---

## Changes to the Project Plan

### TMDB Fetch Target Expanded: 500 → 700 Records

The original plan targeted 500 TMDB records. After running the script, we observed that the RT dataset contains many duplicate and closely related title entries (e.g., *The Aristocats*, *The French Connection* each appear multiple times). Because the script fetches one TMDB result per RT row rather than per unique title, the raw output included repeated TMDB records. We expanded the target to 700 to ensure a sufficient number of unique records survive deduplication. After cleaning, 597 unique TMDB records remained.

### Merged Dataset Scope is 1970–1973

The TMDB fetch script iterated from the top of the RT dataset, which is sorted chronologically starting from 1970. As a result, all 700 fetched TMDB records cover films from approximately 1970–1973. The merged dataset of 255 records therefore reflects this era. This is a known scope limitation that will be documented in the data quality section and final report.

### No Changes to Research Questions

Both core research questions remain unchanged from the project plan. Genre analysis remains excluded from scope.

### No Feedback-Driven Changes

Graded feedback on Milestone 2 had not been received at the time of this report. Any feedback received before final submission will be incorporated into the final report.

---

## Challenges and Issues

### Challenge 1: Duplicate Entries from the RT Dataset

**Issue:** The RT dataset contains multiple rows for the same movie (e.g., *The Aristocats* appears with and without a critic score; *The French Connection* appears several times). These duplicates were passed as separate TMDB search queries, resulting in repeated TMDB records.

**Resolution:** Resolved during cleaning. RT duplicates were resolved by keeping the entry with the highest `critic_score` per `title_clean` + `release_year`, removing 1,194 rows. TMDB duplicates were resolved by deduplicating on `id`, removing 101 rows.

### Challenge 2: Inconsistent `release_date` Formats in RT Dataset

**Issue:** The `release_date` column in `movie_info.csv` uses two different formats: `"Released Dec 16, 1970"` for most entries and a bare `"1970"` for others. Standard date parsing fails on both.

**Resolution:** Resolved using a regex pattern (`\b(19|20)\d{2}\b`) to extract the 4-digit year from either format. This worked reliably across all rows with a parseable date.

### Challenge 3: Score Columns Stored as Strings

**Issue:** `critic_score` and `audience_score` are stored as strings with a `%` suffix (e.g., `"84%"`), making them unusable for numerical analysis directly.

**Resolution:** Stripped the `%` character using `str.replace()` and cast to numeric using `pd.to_numeric(..., errors='coerce')`, which also safely handles any remaining non-numeric values by converting them to `NaN`.

### Challenge 4: Merged Dataset Limited to 1970–1973

**Issue:** Because the TMDB fetch script started from the top of the RT dataset (sorted chronologically), all fetched records cover only ~1970–1973 films. The resulting merged dataset of 255 rows covers a narrow time window, which limits decade-level trend analysis.

**Plan:** This will be documented as a known scope limitation in the final report. As a potential improvement, future iterations could re-run the TMDB fetch script targeting a broader or more recent slice of the RT dataset to improve temporal coverage.

---

## Individual Contribution Summaries

### Flynn Huynh - fhuynh2

I completed the technical work for this milestone. This included setting up the full repository structure, writing and running `tmdb_data_script.ipynb` to collect 700 TMDB records, and implementing the data cleaning and integration pipeline in `analysis.ipynb`. Cleaning steps covered score parsing, date extraction, title normalization, missing value handling, and deduplication for both datasets. The final merge produced `datasets/merged_movies.csv` with 255 records ready for analysis.

### Harlow Nguyen - harlown2

> [**Harlow: nhớ viết contribution của b vs cả commit directly to the repo để mấy cha TA đỡ bắt bẻ nhé and feel free to change anything.**]