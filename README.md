# **Cyclone Shelter Accessibility & Population Risk Modeling – Teknaf Upazila, Bangladesh**

This project builds a geospatial and optimization framework to analyze **cyclone shelter accessibility**, **population distribution**, and **environmental exposure** in **Teknaf Upazila, Cox’s Bazar District**. Using high-resolution demographic, terrain, land-use, and hazard-risk layers, the goal is to identify underserved populations, assess vulnerability, and support potential shelter siting or resource allocation decisions.

The pipeline integrates spatial data engineering, risk-informed constraints, and prescriptive analytics, producing a clean gridded dataset with assigned population counts, environmental features, hazard indicators, and proximity to existing cyclone shelters.

---

## **Project Objectives**

- Assemble accurate geospatial layers for Teknaf (population, union boundaries, shelters, terrain, and hazards).  
- Construct a uniform analysis grid and allocate union-level population across inhabited cells using land-cover masks.  
- Compute accessibility metrics (e.g., distance to nearest shelter, travel cost, coverage radius).  
- Integrate hazard indicators (coastal flood depth, storm surge zones, landslide risk, elevation, slope).  
- Identify high-risk and underserved grid cells using rule-based and optimization-based approaches.  
- Provide a foundation for prescriptive decision models such as locating new shelters or allocating resources under disaster constraints.

---
# **Data Sources**

This project relies only on publicly available open datasets.  
All sources are listed below by category.

---

## **1. Administrative Boundaries & Population**

### **Union-level population data**
- Banglapedia: Teknaf Upazila  
  https://en.banglapedia.org/index.php/Teknaf_Upazila  
- Banglapedia: Upazilas of Bangladesh  
  https://en.banglapedia.org/index.php?title=Category:Upazilas_of_Bangladesh  
- Bangladesh Bureau of Statistics (BBS) – Population Census 2011 (Teknaf report, PDF)  
  https://bbs.portal.gov.bd/sites/default/files/files/bbs.portal.gov.bd/page/86b66335_4fa7_4143_a034_9abe3c537553/2020-05-15-16-23-ebfd02cda263d2cb76f4e75b1c7f91cc.pdf  

### **Union boundaries (GIS shapefiles)**
- GADM Global Administrative Boundaries  
  https://gadm.org/download_country.html  

---

## **2. Cyclone Shelter Locations**

### **Shelter coordinates & metadata**
- HDX: Teknaf & Ukhiya Cyclone Shelters  
  https://data.humdata.org/dataset/cox-s-bazar-district-teknaf-and-ukhiya-upazila-cyclone-shelter-locations  

### **Reference map**
- REACH/IMPACT Initiatives – Teknaf Cyclone Shelter Map (PDF)  
  https://repository.impact-initiatives.org/document/impact/5e981041/REACH_BGD_Map_Teknaf_Cyclone_Shelters_23Oct2019.pdf  

---

## **3. Land Cover, Terrain, and Environment**

### **Land Cover (built-up, shrubland, forest, cropland, etc.)**
- GADM auxiliary raster layers  
  https://gadm.org/download_country.html  

### **Elevation & Slope (DEM)**
- USGS EarthExplorer  
  https://earthexplorer.usgs.gov  
- USGS Data Catalog  
  https://data.usgs.gov/datacatalog/data/USGS:60a81500d34ea221ce4e5a6d  

---

## **4. Natural Hazard & Exposure Datasets**

### **Hazard layers**
- HDX Bangladesh Natural Hazard Datasets  
  https://data.humdata.org/dataset/bangladesh-natural-hazards-datasets  

### **Hazard reference documents**
- UNDRR (2020). *Bangladesh Disaster Risk Landscape.*  
  https://www.undrr.org/publication/bangladesh-disaster-risk-landscape  
- World Bank GFDRR (2017). *Bangladesh Disaster Risk Profile.*  
  https://www.gfdrr.org/en/publication/bangladesh-disaster-risk-profile  
- IPCC AR6 WG2 (2022). *Impacts, Adaptation and Vulnerability.*  
  https://www.ipcc.ch/report/ar6/wg2/  
- BWDB (2021). *Coastal Flood Risk Atlas of Bangladesh.*  
- UN OCHA (2018). *Cox’s Bazar Coastal Hazard Profile.*  
  https://data.humdata.org  
- UNDP (2019). *Coastal Vulnerability Assessment for Bangladesh.*  
- Geological Survey of Bangladesh (2019). *Landslide Hazard Zonation Map: Cox’s Bazar District.*  
- BMD (2017). *Cyclone Risk Map of Bangladesh.*  

---

# **Methodology Overview**

Below is a high-level description of the modeling pipeline.

## **1. Preprocessing & Standardization**
- Load union boundaries from GADM.  
- Clip all raster and vector datasets to the Teknaf Upazila extent.  
- Clean and standardize shelter coordinates, removing duplicates and invalid points.  
- Reproject all layers into a common CRS (EPSG:4326 for lat/lon, plus a metric CRS for analysis).
- - Output: **`grid_pop.csv`**, **`shelter_final.csv`**

## **2. Constructing the Analysis Grid**
- Generate a uniform analysis grid (e.g., 1 sq km resolution).  
- Plot union boundaries.  
- Mask out uninhabitable areas (water bodies, marshes, etc.) using land-cover data.

## **3. Population Allocation**
- Retrieve union-level total population.  
- Count the number of *inhabited* grid cells per union.  
- Distribute union population randomly across inhabited cells.  
- Output: **`grid_pop.csv`**

## **4. Environmental Feature Integration**
For each grid cell:
- Elevation (m)  
- Slope (degrees)  
- Land cover category  
- Distance to coastline     

## **5. Shelter Accessibility Analysis**
- Compute Euclidean or network distances from each grid cell to all shelters.  
- Identify nearest shelter and distance (m).  
- Flag underserved cells (e.g., > 2 km from nearest shelter).  
- Generate heatmaps and risk-access overlays.

## **6. Risk Scoring**
A composite vulnerability score considers:
- Hazard exposure (distance fom coast)  
- Terrain constraints (elevation, slope)  

This score supports prioritizing interventions or new shelter locations.

## **7. Optimizaton Formulation**
To obtain a Pareto-style frontier, we vary $K$, the maximum number of new shelters that may be opened.  
For each fixed K, we solve a lexicographically ordered three-stage problem:
- Stage 1 (Risk-coverage first): maximize total risk-weighted covered population, producing $S_r^\star(K)$ and $\alpha_r^\star(K)$.
- Stage 1b (Existing-use tie-break): among nearly Stage-1-optimal solutions, maximize use of existing shelters, producing $U_E^\star(K)$.
- Stage 2 (Distance tie-break): among solutions preserving near-optimal risk coverage and existing-use, minimize total travel distance.

# **Outputs**

### `grid_with_pop.csv`**
Contains:
- Assigned population  
- Land cover  
- Elevation & slope  
- Hazard indicators  
- Distance to nearest shelter  

### **`shelter_final.csv`**
Cleaned cyclone shelter list with:
- Coordinates  
- Capacity
- Union name  

### **Visualizations**
- Population density maps  
- Accessibility heatmaps  
- Hazard exposure overlays  
- Shelter assignment maps

---

# **Tools & Technologies**
- **Python:** geopandas, rasterio, shapely, numpy, pandas, matplotlib  
- **Julia:** JuMP for mathematical optimization modeling, HiGHS (MIP solver), CairoMakie for plotting
- **File formats:** GeoJSON, CSV, TIF  
- **Jupyter Notebook** for reproducible workflows  

---

# **Citation**
If you use or adapt this workflow, please cite the original data sources listed above and reference this repository.
