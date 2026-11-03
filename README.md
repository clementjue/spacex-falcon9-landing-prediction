# SpaceX Falcon 9 — First-Stage Landing Prediction

Can we predict whether a Falcon 9 first stage will land? SpaceX lists a Falcon 9 launch at about $62 million, while providers that cannot reuse the first stage charge $165 million and more (figures from the course brief). Whether the booster comes back is therefore a good proxy for the cost of a launch, which is what a competitor bidding against SpaceX needs to estimate.

Capstone project of the **IBM Data Science Professional Certificate** (Applied Data Science Capstone), by Jue WANG.

📄 [Final presentation (PDF)](00_SpaceX_Final%20Presentation_Jue%20WANG.pdf)

## At a glance

- **Data:** 90 Falcon 9 launches from June 2010 to November 2020, 3 launch sites, 11 orbit types. Two out of three first stages landed (66.7%).
- **Sources:** SpaceX REST API v4 and the Wikipedia list of Falcon 9 launches (scraped with BeautifulSoup).
- **Stack:** Python, pandas, NumPy, SQL (SQLite), Matplotlib, Seaborn, Folium, Plotly Dash, scikit-learn.

## Pipeline

| # | Step | What it does |
|---|------|--------------|
| 01 | [Data collection: API](01_SpaceX_Data_Collection_API.ipynb) | Calls the SpaceX API (launches, rockets, launchpads, payloads, cores), flattens the JSON, keeps Falcon 9 launches, fills missing payload mass with the mean |
| 02 | [Web scraping](02_SpaceX_Web_Scraping.ipynb) | Parses the launch tables of a fixed Wikipedia revision into a DataFrame |
| 03 | [Data wrangling](03_SpaceX_Data_Wrangling.ipynb) | Maps the 8 landing outcome labels (drone ship, ground pad, ocean × success, failure, no attempt) to a binary target `Class` |
| 04 | [EDA with SQL](04_SpaceX_EDA_SQL.ipynb) | Loads the launch table into SQLite and answers 10 questions in SQL |
| 05 | [EDA with visualization](05_SpaceX_EDA_Data_Visualization.ipynb) | Success by flight number, payload, orbit, launch site and year |
| 06 | [Maps with Folium](06_SpaceX_Interactive_Visual_Analytics_Folium.ipynb) | Launch sites, success/failure marker clusters, distances to coastline, railway, highway and city |
| 07 | [Plotly Dash app](07_SpaceX_Interactive_Visual_Analytics_Plotly.py) | Dashboard with a site selector, success pie chart and payload-range scatter by booster version |
| 08 | [Predictive analysis](08_SpaceX_Predictive_Analytics.ipynb) | Standardisation, 80/20 split, 10-fold GridSearchCV on logistic regression, SVM, decision tree and KNN |

## Findings

**Exploratory analysis**

- Landing success rose steadily: no successful landing before 2014, 62% in 2016, 83% in 2017, 84–90% in 2019–2020.
- KSC LC-39A (77%, 22 launches) and VAFB SLC-4E (77%, 13 launches) do better than CCAFS SLC-40 (60%, 55 launches).
- SSO missions landed 5 out of 5. ES-L1, GEO and HEO also show 100%, but on a single launch each. GTO, the most frequent orbit (27 launches), is the hardest at 52%.
- Heavy payloads are not a handicap in this sample: 88% of launches above 7,000 kg landed (25 launches).
- The first successful ground-pad landing was on 22 December 2015 (SQL).
- Launch sites sit right on the coast: CCAFS SLC-40 is 0.86 km from the shoreline, but 22–27 km from the nearest railway, highway and city.

**Classification** (test set: 18 launches)

| Model | Best parameters | CV accuracy (10-fold) | Test accuracy |
|-------|-----------------|----------------------:|--------------:|
| Logistic regression | C = 0.01 | 84.6% | 83.3% |
| SVM | sigmoid kernel, C = 1, γ ≈ 0.032 | 84.8% | 83.3% |
| Decision tree | entropy, max depth 8 | 87.5% | 94.4% |
| KNN | k = 10, Manhattan distance | 84.8% | 83.3% |

How to read this table: with 18 test launches, one launch is worth 5.6 points of accuracy. The decision tree result also depends on its random seed. Re-running its grid search with seeds 0 to 9 gave test accuracies from 72.2% to 94.4% (median 83.3%). On this amount of data the four models cannot be told apart. More launches, or repeated cross-validation on the full dataset, would be needed to pick one.

## Running it

```bash
pip install -r requirements.txt
jupyter lab                                          # notebooks
python 07_SpaceX_Interactive_Visual_Analytics_Plotly.py   # dashboard on http://127.0.0.1:8050
```

Notebooks 02 to 08 and the dashboard were re-run in November 2026 with Python 3.13, pandas 2.2, scikit-learn 1.9 and Dash 4.5. Data is loaded from the IBM course files and a fixed Wikipedia revision, so no local data file is needed.

Notebook 01 keeps the outputs of the original 2024 run. It calls the public SpaceX API (api.spacexdata.com), which did not respond when tested on 3 November 2026 (HTTP 522).

## Changelog

- **2026-11, notebooks 04 and 08:** fixed notebook 08, where the decision tree's test score and confusion matrix were computed with the SVM model. Updated the SQL queries of notebook 04 to the current version of the IBM dataset (ISO dates, renamed landing outcome column) and removed the lab-specific Db2 connection cell. Replaced JupyterLite-only code in 04 and 08 with standard Python and re-executed both.
- **2026-11, dashboard and remaining notebooks:** ported the Dash app to Dash 4 and made it load its data from the course file. Fixed its pie charts: the per-site chart could swap the success and failure labels, and the all-sites chart showed average success rates instead of success counts. Notebooks 03, 05 and 06 now load data with `pd.read_csv`; notebook 02 sends a User-Agent, which Wikipedia now requires. All four re-executed.
- **2024-12:** first version.

## Data and credits

SpaceX REST API v4 · Wikipedia, *List of Falcon 9 and Falcon Heavy launches* (revision 1027686922) · IBM Skills Network course datasets and notebook templates.
