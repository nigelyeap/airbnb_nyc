# 5-Minute Speech Script: Airbnb NYC Pricing Analysis

**Estimated time:** ~5 minutes (650–700 words at normal pace)

---
Person 1

## Opening — Hook & Hypothesis (0:00–0:40)

Lets start by introducing our hypothesis. Our hypothesis: **Airbnb nightly prices are shaped by neighbourhood-level signals of desirability and safety.** Specifically, we tested five factors — median household income, distance to the CBD, crime density, listing competition, and tree coverage — and asked: which one matters most?

---

## Data & Setup (0:40–1:20)

To manage this data cleanly, we applied the **ANSI/SPARC three-schema architecture** from SC3021. All raw datasets were loaded into a SQLite database as the internal layer. The conceptual layer held the full logical tables — raw Airbnb listings, shooting records, arrests, trees. But the analysis layer never touched those directly. Instead, we exposed four SQL views as the external schema: `v_airbnb`, `v_shooting`, `v_arrests`, and `v_trees`. Each view stripped sensitive or irrelevant columns — for example, host names and IDs were excluded from `v_airbnb` to enforce PII access control, and `v_arrests` reduced a 5-million-row, 19-column table down to just the 3 columns the analysis actually needed. Temporal filters were pushed down to SQL, so only the relevant 2015–2019 records ever entered memory. This separation meant our analysis code was insulated from raw schema changes — a practical demonstration of logical data independence.

To answer that, we built a neighbourhood-level dataset by integrating six sources: Airbnb listings (2019), NYPD shooting and arrest records (2015–2019), the NYC Tree Census, median household income data from NYC Open Data, and NTA population figures for normalization.

We normalized crime figures as incidents per 10,000 residents, tree coverage as trees per square mile, and restricted our price analysis to entire-home listings to reduce room-type noise. All six datasets were joined at the neighbourhood level using spatial polygon mapping and fuzzy name matching — which I'll flag as a limitation later.


---

## Analysis Methods (1:20–2:10)

We used three complementary methods to identify the most impactful factor.

First, **Spearman rank correlation** — because relationships here are monotonic but not strictly linear. This confirmed that income and CBD proximity have the strongest directional associations with price.

Second, **stepwise R-squared drop** — we fit a full linear model on all predictors, then removed one variable at a time and measured how much the model's explanatory power dropped. This gives a unit-consistent, comparable impact score.

Third, we validated that ranking with three default machine learning models: **Decision Tree, Random Forest, and XGBoost** — not for prediction, but as a cross-model robustness check on variable importance.

---
Person 2

## Key Findings (2:10–3:30)

Here's what we found, ranked by overall consensus across all methods:

**Number one: Median household income.** Ranked first in three out of four models. High-income neighbourhoods don't just have wealthier residents — they have denser restaurants, better services, lower crime perception. Guests pay for that bundle, not just the apartment.

**Number two: Distance to the CBD.** This one is interesting. In the linear stepwise test, its R-squared drop was near zero — seemingly unimportant. But in Random Forest and XGBoost, it ranked first. Why? Because CBD distance has a **non-linear effect** — there's a steep price premium very close to Midtown and Lower Manhattan, then it flattens. Linear models miss this. Tree-based models catch it. This is not a contradiction; it's a signal that the effect is structural and non-linear.

**Number three: Listing density** — local market competition. More listings per square mile modestly compresses prices. This ranked strongly in the linear model but weaker in ML, suggesting a more additive, supply-side pressure.

**Number four and five: Shooting density and arrest density.** Both contribute, but less than income and location once those stronger signals are already in the model. Safety matters, but it's partially correlated with income — high-income neighbourhoods also tend to be lower-crime, so some of their effect overlaps.

**Last: Tree density.** Directionally positive — greener areas do price slightly higher — but it's consistently the weakest predictor. Greenery is a nice-to-have signal, not a primary driver.

---
Person 3

## Limitations (3:30–4:20)

A few important caveats.

**Temporal mismatch**: Airbnb data is 2019, tree census is 2015, crime spans 2006–2019. Features from different years introduce inconsistencies.

**Population baseline**: Per-capita crime rates use 2010 census population. Neighbourhoods that gentrified or grew between 2010 and 2019 will have distorted crime density figures.

**Reporting bias**: Crime datasets only capture reported incidents. Under-reporting in certain communities — particularly those with lower trust in law enforcement — means our safety metric may not reflect actual risk uniformly.

**Fuzzy matching errors**: Joining datasets with different neighbourhood naming conventions using fuzzy string matching introduces a small risk of incorrect linkages — especially for ambiguously named areas.

**Omitted variables**: Transit access, school quality, tourism proximity, and listing amenities are absent. These likely confound results — income in particular may be absorbing several of these signals simultaneously.

**Correlation, not causation**: Everything we found is associational. We cannot claim that raising tree density in a neighbourhood will raise Airbnb prices.

---

## Closing — So What? (4:20–5:00)

So what do these findings mean in practice?

For **new hosts**: price relative to your neighbourhood's income tier and CBD distance first — those set your floor and ceiling. Safety and greenery are secondary adjustments.

For **platforms and analysts**: segment NYC by income bands when building pricing models. A flat city-wide model will systematically underfit lower-income outer-borough neighbourhoods and overfit Manhattan.

For **urban planners**: the data suggests that neighbourhood-level investment in safety and amenities is not just a social good — it correlates with real economic signals in the short-term rental market.

Ultimately, what this project shows is that where a listing sits in the city's social and spatial fabric is at least as important as what's inside the apartment. Location isn't just an address. It's a bundle of signals — and guests price that bundle every time they book.

Thank you.

