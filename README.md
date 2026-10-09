# Geocoding Benchmark - 2024-09-23  

The process and results of a simple geocoding benchmark experiment to evaluate geocoding precision, accuracy, and geocoder characteristics for four geocoding tools (see below).

I utilized 5000 random address drawn from the [Miami-Dade County Open Data Hu](https://gis-mdc.opendata.arcgis.com/) - [Property Point View](https://gis-mdc.opendata.arcgis.com/datasets/bf92e51f90a8426cae904ebc15018067_0/explore?location=25.727426%2C-80.236112%2C10) likely downloaded in early September of 2023. Note that Miami-Dade County GIS office runs the Open Data Hub using the ESRI Enterprise Suite.

See the [NOTES.md](NOTES.md) for more information.

### geocoders evaluated

|                                       | Addresses remain in-house | Local installation | HIPAA Compliance * | License Fees |
|---------------------------------------|:-------------------------:|:------------------:|:------------------:|:------------:|
| ArcGIS Enterprise / Streetmap Premium |             X             |          X         |          X         |      $$$     |
| Nominatim                             |             X             |          X         |          X         |              |
| DeGAUSS **                            |             X             |          X         |          X         |              |
| PostGIS Tiger **                      |             X             |          X         |          X         |              |

### Summary Statistics

|            |    Hit rate   |   | Error thresholds |               |                |          |   | Geocoding performance statistics |            |             |               |                    |
|------------|:-------------:|---|:----------------:|:-------------:|:--------------:|:--------:|---|:--------------------------------:|:----------:|:-----------:|:-------------:|:------------------:|
|            | Out of n=5000 |   |    0 to 100 ft   | 100 to 499 ft | 500 ft to 1 mi | > 1 mile |   |            Min (ft) *            | Max (ft) * | Mean (ft) * | Median (ft) * | Standard dev (ft)* |
| PostGIS *  |     99.98%    |   |        537       |     3,325     |       924      |    213   |   |                 5                |  2,700,504 |       3,934 |      155      |             66,546 |
| DeGauss *  |     99.20%    |   |        272       |     3,540     |       957      |    191   |   |                13                |     57,697 |         818 |      165      |              2,420 |
| ArcGIS Pro |     100.0%    |   |       3016       |     1,883     |       94       |     7    |   |                ~0                |    212,687 |         284 |       85      |              4,724 |
| Nominatim  |     98.36%    |   |       3346       |     1,200     |       301      |    71    |   |                ~0                |     57,096 |         303 |       18      |              1,906 |
 
### Contents

```
+-- data
|   benchmarking.ipynb                          # Jupyter notebook used for parts of the geocoding
|   geocode_analyze.gqs                         # QGIS project for visualization and analysis *
|   +-- data  
|   |   ArcGIS_geocode.gdb                      # ArcGIS geodatabase used for geocoding with ArcGIS Pro
|   |   arcgis.dbf                              # note sure
|   |   degauss.csv                             # degauss geocde results
|   |   geocode.sql                             # PostGIS dump of tables used for visualization and analaysis
|   |   mdc_5000_addresses.csv                  # random drawing of 5000 addresses from the MDC open data hub
|   |   mdc_5000_addresses_normalized.csv       # the 5000 addresses run through the postGIS address normalizer 
|   |   nominatim.csv                           # nominatim geocoding results
|   |   postgis.csv                             # the postgis TIGER geocding results
```

> \* The QGIS project needs all files in the data directory _and_ geocode.sql needs to be ingested into a local version of Postgres with the postGIS extension running on port 5432. If you need to change the port, open the QGIS project in a text editor and search/replace all instances of 5432 with your desired port.

### Contact:

Timothy Norris
University of Miami Libraries
Frost Institute for Data Science and Computing

### License:
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/) [![License: AGPL v3](https://img.shields.io/badge/License-AGPL_v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)