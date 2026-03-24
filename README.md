# MPR Infrastructure Site Analysis

An interactive tool for overlaying Census demographic data on Minnesota Public Radio transmitter coverage areas. Built to support the FY2027 Congressionally Directed Funding application to Senators Klobuchar and Smith.

## What it does

Select one or more MPR transmitter sites, set a coverage buffer radius (10–75 miles), and click **Analyze**. The tool fetches live data from the U.S. Census Bureau ACS 2022 5-Year Estimates and shows:

- A choropleth map of Minnesota counties colored by any demographic metric
- A sortable data table with all metrics per county
- Summary cards (total population, weighted average income, poverty rate)
- One-click TSV export for pasting into Excel or Google Sheets

## Current demographic variables

All data from ACS 2022 5-Year Estimates, Table B03002 (race/ethnicity) and related tables:

| Variable | Census Table | Notes |
|---|---|---|
| Total population | B01003 | |
| Median household income | B19013 | |
| Poverty rate | B17001 | % below federal poverty line |
| Unemployment rate | B23025 | Civilian labor force |
| No internet access | B28002 | % of households |
| Renter-occupied housing | B25003 | % of occupied units |
| Age 65+ | B01001 | % of total population |
| Non-white population | B03002 | 100% minus White non-Hispanic |
| American Indian / Alaska Native | B03002_005E | Non-Hispanic alone |
| Hispanic or Latino | B03002_012E | Any race |
| Black / African American | B03002_004E | Non-Hispanic alone |
| Asian | B03002_006E | Non-Hispanic alone |

## Census API usage

The tool makes **one API call** per analysis using the ACS 5-Year Estimates endpoint:

```
https://api.census.gov/data/2022/acs/acs5
  ?get=NAME,[VARS]
  &for=county:*
  &in=state:27
  &key=[YOUR_KEY]
```

Currently fetches **all 87 Minnesota counties** and filters client-side by buffer proximity. This is efficient for the county geography level.

### The 50-variable limit

The Census API caps each request at 50 variables (including NAME, state, and county). The current call uses **30 variables total** — leaving 20 slots available for expansion.

**Variables that can be added in a future spec:**

| Variable | Census Code | Slots needed |
|---|---|---|
| No high school diploma | B15003_002–016 | 16 (or use S1501 subject table: 2) |
| Disability rate | B18101 (summary) | 4 |
| Uninsured rate | B27010_017,033,050,066 | 5 |
| Median age | B01002_001E | 1 |
| Households with children | B11003 | 2 |
| Veterans | B21001 | 2 |
| Foreign born | B05002 | 2 |
| Households with no vehicle | B08201 | 2 |

**Recommended approach for the final grant submission:** Once we confirm the exact upgrade target sites, run a focused query at the **census tract level** rather than county level. This gives much higher precision — a county like Beltrami (Bemidji) contains both urban and rural tracts with very different demographics. Tract-level data better captures who actually lives within the signal footprint.

Tract-level call structure:
```
https://api.census.gov/data/2022/acs/acs5
  ?get=NAME,[VARS]
  &for=tract:*
  &in=state:27%20county:[COUNTY_FIPS]
  &key=[YOUR_KEY]
```

You would loop this call per county within the buffer, then aggregate weighted by tract population.

## Setup

1. Get a free Census API key at [api.census.gov/sign-up](https://api.census.gov/sign-up)
2. Open `index.html` in a text editor
3. Find `YOUR_CENSUS_API_KEY` and replace with your key
4. Deploy to any static host (GitHub Pages, Netlify, etc.)

## Deployment (GitHub Pages)

This repo is configured to serve from the `main` branch root. The site is live at:
`https://civicoperations.org/mpr-overlay/`

To update: edit `index.html`, commit, and push. GitHub Pages deploys within ~2 minutes.

## Data source

U.S. Census Bureau, American Community Survey 5-Year Estimates (2018–2022).  
API documentation: [census.gov/data/developers/data-sets/acs-5year.html](https://www.census.gov/data/developers/data-sets/acs-5year.html)

## Context

Built as a supporting tool for Minnesota Public Radio's FY2027 Congressionally Directed Funding request for Infrastructure & Technology Upgrades, submitted to the offices of Senators Amy Klobuchar and Tina Smith. Deadline: March 27, 2026. Source of infrastructure data is mpr.org/listen/stations 

The demographic overlay supports the "Community Impact" and "Project Justification" sections of the application by documenting the underserved populations within MPR's broadcast coverage areas.
