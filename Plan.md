I am trying to answer the question, "From the year 2010 to 2024, how does the increase in levels of bachelor degrees or higher in San Francisco County track with unemployment rates in that area?" I want you to use the FRED API to retrieve the following data sets: "Unemployment Rate in San Francisco County/City, CA", "Bachelor's Degree or Higher (5-year estimate) in San Francisco County/city, CA (HC01ESTVC1706075)".

The final output is a Jupyter notebook (`analysis.ipynb`) in this repository (Week-6-Assignment).

## Expectation (hypothesis)

As the share of adults with a bachelor's degree or higher rises, the unemployment rate should fall. I expect a **negative** relationship between the two series.

## 1. Data retrieval

- [ ] Store the FRED API key in an environment variable (`FRED_API_KEY`), not in the notebook. Add any `.env` file to `.gitignore`.
- [ ] Use `fredapi` to pull:
  - `CASANF0URN`: Unemployment Rate in San Francisco County/City, CA (monthly, not seasonally adjusted)
  - `HC01ESTVC1706075`: Bachelor's Degree or Higher, 5-year estimate (annual)
- [ ] Save the raw pulls to `data/raw/` as CSV so the notebook can run again without the API.
- [ ] Record each series' units, frequency, and last-updated date from the FRED metadata.

## 2. Cleaning steps

**Date ranges**
- [ ] Filter both series to 2010-01-01 through 2024-12-31.
- [ ] Note that each ACS 5-year value is labeled by its end year (for example, 2010 = the 2006–2010 average). Confirm which years are available. The most recent year may not be published yet.

**Frequency alignment / transformations**
- [ ] Convert monthly unemployment to annual by taking the calendar-year mean, which gives one row per year.
- [ ] Also build a 5-year rolling mean of annual unemployment. This matches the ACS 5-year window, so the two series compare like with like.
- [ ] Create derived columns:
  - Year-over-year change in each series (first differences)
  - An index for each series with 2010 = 100, so both can be compared on one scale

**Merges**
- [ ] Inner-join the annual unemployment and bachelor's series on `year`.
- [ ] Check that the merged table has the expected 15 rows (2010–2024), or note which years are missing.

**Missing values**
- [ ] Count NaNs in each series before and after the merge.
- [ ] Do not interpolate the ACS series. Drop years where it is missing, and note them.
- [ ] If any unemployment month is missing, compute the annual mean from the available months and flag that year.

**Sanity checks**
- [ ] Check that values fall in plausible ranges (rates between 0 and 100%).
- [ ] Print `df.describe()` and the first and last rows of the merged table.
- [ ] Flag 2020 (COVID) as a known outlier and add a `covid` dummy column (1 for 2020–2021).

## 3. Charts

1. **Dual-axis line chart:** bachelor's share on the left axis and unemployment rate on the right, by year, 2010–2024.
2. **Indexed line chart:** both series indexed to 2010 = 100 on a single axis.
3. **Scatter plot with regression line:** bachelor's share (x) against unemployment (y), with each point labeled by year.
4. **First-difference scatter:** year-over-year change in bachelor's share against year-over-year change in unemployment. This separates the shared time trend from real co-movement.
5. (Optional) Repeat chart 3 using the 5-year rolling mean of unemployment.

## 4. Statistical tests

- [ ] **Pearson correlation** on levels, with r and p-value.
- [ ] **Spearman rank correlation** on levels. It holds up better with the 2020 outlier and a small sample.
- [ ] **Correlation on first differences.** This tests whether the series move together year to year, beyond both trending over time.
- [ ] **OLS regression** (`statsmodels`): `unemployment ~ bachelors + covid`. Report the coefficient, standard error, p-value, and R².
- [ ] **Robustness:** rerun the correlation and regression without 2020–2021, and again using the 5-year rolling unemployment.
- [ ] **Caveats to state in the notebook:**
  - The sample is small (n ≈ 15).
  - Overlapping ACS windows make the education series smooth and autocorrelated.
  - Two trending series can correlate without any causal link (spurious correlation).
  - County-level correlation says nothing about individuals (ecological fallacy).

## 5. Interpreting the results

**Supports the expectation if:**
- The levels correlation is negative and statistically significant (p < 0.05).
- The OLS coefficient on bachelor's share stays negative and significant after controlling for COVID.
- The first-difference correlation is also negative, meaning the relationship shows up year to year as well as in the long-run trend.

**Contradicts the expectation if:**
- The correlation is positive, or close to zero and not significant.
- The levels relationship is negative but disappears in first differences or once 2020–2021 is excluded. That would suggest the link comes from a shared time trend and the business cycle (the post-2010 recovery and the COVID spike), not from education.

**Inconclusive if:**
- The results flip between Pearson and Spearman, or between the full sample and the sample without 2020–2021. If so, note that n is too small to draw a firm conclusion.

## 6. Deliverable

- [ ] Write a markdown conclusion cell that answers the research question in 3–5 sentences and cites the test results.
- [ ] Restart the kernel and run all cells top to bottom before committing.
- [ ] Commit the notebook, `data/raw/` CSVs, and `requirements.txt` (`pandas`, `fredapi`, `matplotlib`, `scipy`, `statsmodels`), then push to the Week-6-Assignment GitHub repo.
