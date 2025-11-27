# Oakland Residential Real Estate Pricing

This project builds data-driven pricing models for Oakland, CA residential properties using public sale data and neighborhood attributes. The goal is to help an investor identify **undervalued listings** and estimate **value creation from potential improvements** in targeted ZIP codes.

The work started as an analytics project for my UCLA MBA program and evolved into a reusable toolkit for exploring Oakland real estate.

---

## Key Questions

- What factors explain variation in **sale prices** across Oakland?  
- Which neighborhoods and ZIP codes appear systematically undervalued, after controlling for property and neighborhood characteristics?  
- Can we give a prospective investor a simple model to:
  - Flag potentially mispriced listings, and  
  - Estimate value creation from property improvements vs. renovation cost?

---

## Data & Sources

_All data are public or derived from public sources._

- **Redfin sale records** for Oakland single-family homes and condos  
  - Sale price, list price, days on market, beds/baths, year built, lot size, square footage, etc.
- **School quality**  
  - California School Dashboard metrics (ELA/Math performance by school), aggregated to neighborhoods / ZIP codes.
- **Transit & access**
  - Distance to nearest BART station  
  - Walkable access to key amenities (grocery, retail, parks, waterfront)
- **Historic redlining**
  - HOLC “residential security” maps for Oakland, joined to modern parcel / neighborhood boundaries.
- **Other neighborhood features**
  - Crime index and demographic context where available.

> Note: Raw files are not checked into this repo if they’re large or license-restricted. Where possible, I include cleaned/aggregated versions or describe how to obtain the original sources.

---

## Feature Engineering

Examples of engineered features used in the models:

- **Price metrics**
  - `price` = sale price  
  - `price_per_sqft` = `sale_price / finished_sqft`  
  - Log transforms (`log_price`, `log_ppsf`) for robustness checks

- **Location**
  - ZIP code dummies (`zip_94607`, `zip_94608`, …)  
  - Distance to nearest BART station (meters / miles)  
  - Indicators for “waterfront-adjacent”, “hillside”, “near major employer cluster`

- **Property characteristics**
  - Beds, baths, lot size, year built, age buckets  
  - Renovation indicators (e.g., “recently remodeled” flag if available)

- **School quality**
  - Average ELA and Math performance score for the nearest public school

- **Historic redlining**
  - Categorical HOLC grade (A–D) and/or dummy for historically redlined tracts

---

## Modeling Approach

All modeling is done in R using the tidyverse ecosystem. The core analysis uses a **three-step model ladder**, each step adding more structure to the pricing story.

### Model 1 – Size Only

    price ~ square_footage

- Baseline relationship between size and sale price  
- Useful as a “naive” benchmark for how much buyers pay per additional square foot on average

### Model 2 – Size + Beds/Baths

    price ~ square_footage + beds + baths

- Adds basic livability features  
- Captures how bedroom/bathroom mix shifts price at a given size

### Model 3 – Full Context Model

    price ~ square_footage + beds + baths +
            amenities + school_quality +
            redline_or_zip + time_sold

Adds:

- Amenity access (e.g., transit, parks, grocery, waterfront)  
- School quality indices  
- Historic redlining / ZIP-code controls  
- Time-sold indicators to account for market cycles / regime shifts  

This is the main model used to:

- Compare **predicted vs. listed prices** for current listings  
- Estimate **value creation** from hypothetical improvements (e.g., interior upgrade or amenity improvement)

For each model, I track changes in fit (e.g., adjusted R²) and the stability of key coefficients as neighborhood context is added.

---

## Undervaluation Score

For any current listing:

1. Use the final model to generate a **predicted sale price** (`predicted_price`).  
2. Compare it to the **current list price** (`list_price`).  
3. Define an “undervaluation score”:

\[
\text{score} = \frac{\text{predicted\_price} - \text{list\_price}}{\text{predicted\_price}}
\]

- **Positive scores** → potentially undervalued relative to fundamentals  
- **Negative scores** → potentially overpriced  

For interpretability, both values can also be translated into **price per square foot** by dividing by finished square footage, but the regression itself is estimated on price.

---

## Value Creation Calculator

For candidate properties, the repo includes helpers to:

1. **Simulate an improved condition**  
   - e.g., better amenity score, interior upgrade, or moving from “dated” to “updated” quality

2. **Re-run the model** to estimate a **post-renovation price**

3. **Compare value uplift vs. renovation cost**  
   - `margin = (post_reno_value - current_value - reno_cost)`

This gives a simple margin check: do the modeled gains justify the renovation budget?

---

## Key Findings (Sample-Specific)

These results are **illustrative for the sample analyzed** and not investment advice:

- After controlling for property characteristics, amenities, school quality, and year effects, **ZIP codes 94607 and 94608** show **strong fundamentals and higher model-predicted prices** relative to many other ZIPs.  
- The model flags a subset of listings in these ZIPs where **list price is below model-predicted price**, suggesting potential undervalued opportunities.  
- Historic redlining patterns remain visible: properties in formerly HOLC C/D tracts often trade at a **discount** even after controlling for structural features, highlighting both **investment potential** and **equity considerations**.  
- The value-creation tool suggests that **modest interior upgrades** in targeted ZIPs can create attractive margins when renovation cost is kept disciplined.

Results depend on data quality, sample selection, and modeling choices; they should be treated as one input into an investment process, not a standalone recommendation.

---

## Repo Structure (Example)


    /
    ├── data/           # cleaned / aggregated data
    ├── R/              # model scripts and helpers
    ├── notebooks/      # R Markdown notebooks for exploration and reporting
    └── README.md       # this file

---

## How to Run

1. Clone the repo and open the project in RStudio.  
2. Install required packages: `tidyverse`, `sf`, `broom`, `ggplot2`, etc.  
3. Run the scripts in `R/` in order, or knit the main notebook in `notebooks/` to reproduce the analysis and charts.
