# Oakland Residential Real Estate Pricing

This project builds data-driven pricing models for Oakland, CA residential properties using public sale data and neighborhood attributes. The goal is to help an investor identify **undervalued listings** and estimate **value creation** from potential improvements in targeted ZIP codes.

The work started as an analytics project for my UCLA MBA program and evolved into a reusable toolkit for exploring Oakland real estate.

---

## Key Questions

1. **What factors explain variation in price per square foot across Oakland?**  
2. **Which neighborhoods and ZIP codes appear systematically undervalued, after controlling for property and neighborhood characteristics?**  
3. **Can we give a prospective investor a simple model to:**
   - Flag potentially mispriced listings, and  
   - Estimate value creation from property improvements vs. renovation cost?

---

## Data & Sources

*(All data are public or derived from public sources.)*

- **Redfin sale records** for Oakland single-family homes and condos  
  - Sale price, list price, days on market, beds/baths, year built, lot size, square footage, etc.
- **School quality**  
  - California School Dashboard metrics (ELA/Math performance by school) aggregated to neighborhoods / ZIP codes.
- **Transit & access**  
  - Distance to nearest BART station.  
  - Walkable access to key amenities (grocery, retail, parks, waterfront).
- **Historic redlining**  
  - HOLC “residential security” maps for Oakland, joined to modern parcel / neighborhood boundaries.
- **Other neighborhood features**  
  - Crime index and demographic context where available.  

> **Note:** Raw files are not checked into this repo if they’re large or license-restricted. Where possible, I include cleaned/aggregated versions or describe how to obtain the original sources.

---

## Feature Engineering

Examples of engineered features used in the models:

- **Price metrics**
  - `price_per_sqft = sale_price / finished_sqft`
  - Log price and log price per sqft (`log_price`, `log_ppsf`)
- **Location**
  - ZIP code dummies (`zip_94607`, `zip_94608`, …)  
  - Distance to nearest BART station (meters / miles)  
  - Indicators for “waterfront-adjacent”, “hillside”, “near major employer cluster”
- **Property characteristics**
  - Beds, baths, lot size, year built, age buckets  
  - Renovation indicators (e.g., “recently remodeled” flag if available)
- **School quality**
  - Average ELA and Math performance score for the nearest public school
- **Historic redlining**
  - Categorical HOLC grade (A–D) and/or dummy for historically redlined tracts

---

## Modeling Approach

All modeling is done in **R** using the tidyverse ecosystem. I built a simple three-step model ladder, each layer adding more structure to the pricing story.

1. **Model 1 – Size only**  
   - `price ~ square_footage`  
   - Baseline relationship between size and sale price; useful as a “naive” benchmark.

2. **Model 2 – Size + Beds/Baths**  
   - `price ~ square_footage + beds + baths`  
   - Controls for basic livability features and shows how much additional variation is explained once we add bedroom/bathroom mix.

3. **Model 3 – Full context model**  
   - `price ~ square_footage + beds + baths + amenities + school_quality + redline_zoning_or_zip + time_sold`  
   - Adds:
     - Amenity access (e.g., transit, parks, grocery, waterfront),
     - School quality indices,
     - Historic redlining / ZIP code controls, and
     - Time-sold indicators to account for market regime.
   - This is the main model used to:
     - Compare predicted vs. listed prices for current listings, and
     - Estimate value creation from improvements (e.g., upgrading a property in a high-demand ZIP).

For each model, I track changes in fit (e.g., adjusted \(R^2\)) and how key coefficients evolve as more neighborhood context is added.


### 2. Undervaluation Score

For any **current listing**:

1. Use the final model to generate a **predicted price per sqft**.  
2. Compare to the **observed list price per sqft**.  
3. Define an **“undervaluation score”**:

\[
\text{score} = \frac{\text{predicted\_ppsf} - \text{list\_ppsf}}{\text{predicted\_ppsf}}
\]

- Positive scores → potentially **undervalued** relative to fundamentals.  
- Negative scores → potentially **overpriced**.

### 3. Value Creation Calculator

For candidate properties, the repo includes simple helpers to:

- Simulate an **improved condition** (e.g., higher quality/amenity score, cosmetic renovation).  
- Re-run the model to estimate a **post-renovation value**.  
- Compare **value uplift** to **estimated renovation cost** to assess margin.

---

## Key Findings (for the sample analyzed)

- After controlling for property characteristics, amenities, school quality, and year effects, ZIP codes **94607** and **94608** consistently show **higher predicted price per sqft** and strong fundamentals.
- The model flags a subset of listings in these ZIPs where the **list price per sqft is below model-predicted value**, suggesting potential **undervalued opportunities**.
- Historic redlining remains visible in pricing patterns: properties in formerly redlined tracts (HOLC C/D) often trade at a discount even after controlling for structural features, highlighting both investment potential and equity considerations.
- The value-creation tool illustrates that modest interior upgrades in targeted ZIPs can create attractive margins when renovation cost is disciplined.

> These are model-based insights, not investment advice; they depend on sample selection, data quality, and modeling choices.

---

## Repository Structure



```text
oakland-real-estate/
├─ R/
│  ├─ 01_Redfin-BART-School-HOLC-Amenities-Data-Clean.ipynb
│  ├─ 02_Oakland Real Estate Assessment Group 11-Final Submission.ipynb
├─ notebooks/
│  ├─ 01_Redfin-BART-School-HOLC-Amenities-Data-Clean.ipynb
├─ data_sample/
│  ├─ Oakland Real Estate Assessment Group 11-Final Submission.html
├─ slides/
│  ├─ Oakland Real Estate Assessment Group 11-Final Submission.html
└─ README.md
