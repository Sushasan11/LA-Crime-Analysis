# LA Crime Analysis

Exploratory analysis of reported crime in Los Angeles from 2020 to 2024, covering how counts change over time and where incidents cluster. The dataset has 991,360 rows and 28 columns.

This repo holds the **exported results** of a Google Colab notebook. It doesn't include the source notebook or the dataset, so the analysis can't be re-run from here.

## Contents

| File | What it is |
| --- | --- |
| `Crime.html` | Notebook export with the code, outputs and two trend charts |
| `crime_hotspot_map.html` | Interactive heatmap of every incident (21.5 MB) |

## Viewing the results

Clone the repo and open either file in a browser:

```bash
git clone https://github.com/Sushasan11/LA-Crime-Analysis.git
cd LA-Crime-Analysis
```

No installation is needed, but the heatmap needs an internet connection to load its map library and street tiles. It may take a few seconds to open because all the points are embedded in the page.

## What the notebook does

1. Loads the CSV and inspects column types, missing values and duplicates.
2. Creates a cleaned copy, `df_cleaned`, by dropping sparse columns, filling missing values and parsing dates.
3. Plots report counts by year and by month with Matplotlib and Seaborn.
4. Builds the heatmap with Folium.
5. Defines a small Dash dashboard with the yearly trend and the map.

Approximate reports per year, read from the yearly chart:

| Year | Reports |
| --- | --- |
| 2020 | 267,000 |
| 2021 | 130,000 |
| 2022 | 230,000 |
| 2023 | 234,000 |
| 2024 | 130,000 |

## Known limitations

Don't read these numbers as real changes in crime. The analysis is exploratory and has these known issues:

- **Cleaning isn't applied:** the charts and map use the raw `df`, not `df_cleaned`.
- **Duplicates are counted:** 80,912 fully duplicated rows (about 8%) are included in the charts and map.
- **2021 looks incomplete:** every month is about half the level of neighbouring years, which suggests missing data rather than a real drop. This hasn't been confirmed.
- **2024 falls sharply:** monthly counts drop from about 20,000 in January to about 8,000 by mid-year. LAPD adopted a new records system on March 7, 2024, and has since stopped updating this dataset, so the drop probably reflects that change rather than less crime.
- **Dates are report dates:** trends use `Date Rptd`, not the date the crime occurred (`DATE OCC`).
- **Missing locations are plotted:** 2,031 points sit at (0, 0), which the dataset uses for missing locations. They pull the heatmap's starting view too far east.
- **Victim fields aren't validated:** ages include negative values, 0 and 120, and sex and descent include `-`.
- **Months are out of order:** the monthly chart's x-axis is alphabetical, which hides seasonal patterns.
- **The dashboard map path is wrong:** the Dash iframe expects `/assets/crime_hotspot_map.html`, but the map is saved to the working directory.

## Reproducing the analysis

To rebuild the analysis, you'd need to recreate the notebook from the code in `Crime.html`:

1. Download the CSV from the [LAPD crime dataset](https://data.lacity.org/Public-Safety/Crime-Data-from-2020-to-Present/2nrs-mtv8) on the LA Open Data portal, now titled "Crime Data from 2020 to 2024". It may have changed since this export, so your counts may differ. For current data, LAPD now publishes separate NIBRS datasets on the same portal.
2. Update the file path. The notebook reads from Colab's `/content/Crime_Data_from_2020_to_Present.csv`.
3. Install the dependencies:

   ```bash
   pip install pandas matplotlib seaborn folium dash plotly
   ```

   The export shows Python 3.11, pandas 2.2.2, Dash 3.0.4 and Plotly 5.24.1. It doesn't show the other versions.

## Map tiles

The heatmap uses Esri's World Street Map tiles, because OpenStreetMap's servers returned a `403 Access blocked` error when the page was viewed. `folium.Map` uses OpenStreetMap by default, so keep the tile settings if you regenerate the map:

```python
crime_map = folium.Map(
    location=map_center,
    zoom_start=12,
    tiles="https://server.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer/tile/{z}/{y}/{x}",
    attr="Tiles &copy; Esri",
)
```

The saved notebook export still shows the original `folium.Map` call without these settings.
