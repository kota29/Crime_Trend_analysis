# Crime Trend Analysis & Hotspot Detection in Ireland

Machine-learning pipeline to analyze Irish crime trends (2003–2023) and detect geographic crime hotspots using clustering (HDBSCAN) and classification (Random Forest). The project combines official CSO datasets with geocoded Garda station locations to enable spatial analytics and interactive hotspot maps.  

## Project Highlights

* **Datasets**

  * **CJA01**: crime trends by year and type of offence
  * **CJA07**: Garda station-level incidents (used for hotspot detection), later enriched with latitude/longitude via geocoding  
* **EDA**

  * Interactive trend exploration by offence type and year range (Plotly + ipywidgets) 
* **Hotspot Detection**

  * **HDBSCAN** clustering:

    * Geo-based clustering using **Haversine** distance
    * Crime-intensity clustering using **Euclidean** distance over `[lat, lon, log(crimes)]` 
  * Reported Silhouette Scores:

    * **0.842** (Geo-based / Haversine)
    * **0.536** (Intensity-based / Euclidean)  
* **Prediction**

  * Random Forest classification of “high-crime hotspots” with **SMOTE** for imbalance handling  
  * Reported performance:

    * ~**85.5% accuracy**
    * **AUC-ROC ~0.92**
    * F1-score ~**0.85**
    * Cross-validated AUC ~**0.924**  
* **Visualization**

  * Interactive maps per year exported as `crime_hotspots_<year>.html` (Folium) 



## Installation

### 1) Create environment

```bash
python -m venv .venv
source .venv/bin/activate   # (Windows: .venv\Scripts\activate)
pip install -U pip
```

### 2) Install dependencies

Core packages observed in the code: 

* pandas, numpy
* scikit-learn
* hdbscan
* matplotlib, seaborn
* folium
* plotly
* ipywidgets
* imbalanced-learn (SMOTE)
* googlemaps (for geocoding)

Example:

```bash
pip install pandas numpy scikit-learn hdbscan matplotlib seaborn folium plotly ipywidgets imbalanced-learn googlemaps
```

If running in Jupyter:

```bash
pip install notebook
```

Enable widgets in classic Jupyter if needed (environment-dependent).

---

## Data Requirements

Place the CSO files in `data/`:

* `CJA01.csv` → crime trends
* `CJA07.csv` → Garda station-level crime records

The code expects the filenames exactly as above at load time. 

### What the geocoding step does

1. Extract unique Garda station names from CJA07
2. Geocode each station to get `(lat, lon)`
3. Manually patch a few failed geocodes
4. Merge coordinates back into the main dataset and save `crime_dataset.csv` 

---

## How to Run (Notebook Flow)

### Step 1 — Load datasets

* Read `CJA01.csv` (crime trends) and `CJA07.csv` (hotspots input). 

### Step 2 — Geocode Garda stations (creates spatial dataset)

Outputs:

* `unique_garda_stations.csv`
* `garda_stations_with_coordinates_google.csv`
* `crime_dataset.csv` (CJA07 enriched with latitude/longitude) 

### Step 3 — Data cleaning & preparation

* Fill missing `VALUE` with 0
* Ensure `Latitude/Longitude` numeric and drop missing
* Aggregate duplicates if present
* Build station-year totals (`TotalCrimes`)
* Pivot crime trends by offence type per year for EDA 

### Step 4 — EDA

* Interactive Plotly line charts by offence type + year range
* Bar chart: total crimes by year 

### Step 5 — Hotspot detection (HDBSCAN)

Two clustering runs: 

1. **Geo-based** (Haversine metric):

   * Clusters purely from latitude/longitude (in radians)
2. **Crime-based** (Euclidean metric):

   * Clusters using latitude/longitude + log-transformed crimes

Stores labels:

* `Cluster_Geo`
* `Cluster_Crime`

### Step 6 — Hotspot map generation (Folium)

* Interactive year slider
* Saves: `crime_hotspots_<year>.html` 

### Step 7 — Classification (Random Forest + SMOTE)

* Compute average crimes per geo-cluster
* Mark clusters above global average as `HotspotCluster = 1`
* Train RandomForest on `[Latitude, Longitude, Year]` after SMOTE balancing
* Evaluate using:

  * classification report
  * confusion matrix
  * ROC/AUC
  * cross-validated AUC 

---

## Results Summary (as reported)

* HDBSCAN Silhouette:

  * **0.842** geo-based
  * **0.536** intensity-based  
* Random Forest:

  * **~85.5% accuracy**
  * **AUC-ROC ~0.92**
  * **F1 ~0.85**
  * **CV AUC ~0.924**  

---

## Ethical Considerations

* Uses aggregated/public statistics (no individual-level targeting)
* Applies SMOTE to reduce imbalance bias toward low-crime class
* Uses interpretable modeling (feature importance from Random Forest)
* Avoid “predictive policing on individuals”: focus is on **place-based patterns** 

---

## Known Limitations / Notes

* Geocoding quality depends on station naming consistency; manual coordinate patches were needed for some entries. 
* Current classifier features are limited to `[lat, lon, year]` in the code; offense type/station name are discussed in the presentation as influential signals, but they are not included as model features in the shown training block.  
* Seaborn is used for plots in the notebook; Folium maps are saved as HTML for exploration. 

