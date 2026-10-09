# Geocode Benchmark Notes  

The process and results of a simple geocoding benchmark experiment to evaluate geocoding precision, accuracy, and geocoder characteristics for four geocoding tools: the ESRI ArcGIS geocoder in ArcGIS Pro with a paid subscription to the ESRI North America street database, the oepn source Nominatim  geocoder based on the Open Street Map database, the DeGauss geocoding tool based on the US Census TIGER line files, and the PostGIS TIGER Line geocoding tool also baed on the US Census TIGER line files. 

I utilized 5000 random address drawn from the [Miami-Dade County Open Data Hu](https://gis-mdc.opendata.arcgis.com/) - [Property Point View](https://gis-mdc.opendata.arcgis.com/datasets/bf92e51f90a8426cae904ebc15018067_0/explore?location=25.727426%2C-80.236112%2C10) likely downloaded in early September of 2023. Note that Miami-Dade County GIS office runs the Open Data Hub using the ESRI Enterprise Suite.

### Randomly select 5000 addresses

```sql
-- Get a random set of 5000 addresses from MDC
-- the result is mdc_5000_addresses_normalized.csv
-- 5000 addresses from:
SELECT * FROM public.mdc_property_point_view
WHERE true_site_addr IS NOT null
ORDER BY random()
LIMIT 5000
```

NOTE: these were then run through an address normalizer found in the jupyter notebook.  

### Geocoding process

This varied for each tool. 
- ArcGIS - I used the geocoding tool in ArcGIS Pro linked to an institutional geocoding account. The geocoding account  provides access to the paid version of the ESRI North American street database.
- Nominatim/DeGauss/PostGIS - I used the benchmarking.ipynb notebook in conjunction with three docker containers that provided the services for the three geocoders, one each for Nominatim, Degauss and PostGIS TIGER geocoders. I believe I can find the manifest for these containers to get the software versions and dates that geocoding the databases were pulled.

### Geocode runtimes

Windows PC - Intel i9-9900 @ 3.10GHz and 32GB installed RAM

-   degauss: 21m 13s first try, 48 internal server errors but all recovered  ... 
    21m 18s second time, 61 internal errors all recovered -- 5000 rows
    4960 joined to ArcGIS table
-   nominatim: ~18 min first try, 82 skipped ... 
    2 min 3 sec second time through ... 82 skipped -- 4919 rows
    4918 joined to ArcGIS table
-   postgis: 1m 59 s first time - no obvios errors ... 
    1m 58s second time through with no obvuis errors -- 5000 rows
    4999 joined to ArcGIS table
-   ArcGIS: 31s, 4865 Matched (97.30%), 132 Unmatched (2.64%), 3 Tied (0.06%) - cost 

### Normalize geocoding results and reproject to EPSG:2236 for measurement

Create geoms for table mdc_5000_addresses_normalized in EPSG 2236 (NAD83 State Plane Florida East)

```sql
-- degauss lat_1, lon_1 (WGS84) -> degauss_local_geom
-- postgis lat, lon (WGS84) -> postgis_local_geom
-- nominatim lat12, lon12 (WGS84) -> nominatim_local_geom
-- arcgis displayx, displayy (WGS84) -> argis_local_geom

ALTER TABLE mdc_5000_addresses_geocode_join ADD COLUMN nominatim_local_geom geometry(Point, 2236);
UPDATE mdc_5000_addresses_geocode_join SET nominatim_local_geom = ST_Transform(
	ST_SetSRID(
		ST_MakePoint(nominatim_lon::numeric,nominatim_lat::numeric), 4326
	),
	2236
)
WHERE nominatim_lon is not Null AND nominatim_lat is not Null;

-- repeat for other geocoders ...
```

### Join results check on lat long columns for geom creation

-   nominatim 4918 with geom (98.4%)
-   degauss 4960 with geom
-   postgis 4999 with geom
-   arcgis 5000 with geom (must delete unmatched)

### Calculate distances (for each of arcgis, nominatim, degauss, postgis)

```sql
ALTER TABLE mdc_5000_addresses_geocode_join ADD COLUMN arcgis_distance numeric;
UPDATE mdc_5000_addresses_geocode_join 
	SET postgis_distance = ST_Distance(geom_local,arcgis_local_geom)
WHERE arcgis_local_geom is not Null;
-- repeat for other geocoders
```

### Calculate basic stats of distance to original point (feet), note argis is likely a source of the county location data ....

postgis
| min | max | mean | median | std |
| --- | --- | ---- | ------ | --- |
| 5.17303098168929 | 2700504.02641769 | 3933.9542682726007494 | 155.451591162212 | 66545.94 |
3325 locations off by more than 100 feet
924 locations off by more than 500 feet
213 over a mile

arcgis
| min | max | mean | median | std |
| --- | --- | ---- | ------ | --- |
| 0.0513386001141654 | 212686.604964339 | 284.0539801346072805 | 85.48755964817765 | 4724.06 |
1883 locations off by more than 100 feet
94 locations off by more than 500 feet
7 over a mile

nominatim
| min | max | mean | median | std |
| --- | --- | ---- | ------ | --- |
| 0.00439270357252232 | 57095.9856198431 | 302.62381417966379066 | 18.41571941586185 | 1906.00 |
1200 locations off by more than 100 feet
301 locations off by more than 500 feet (6%)
71 over a mile

degauss
| min | max | mean | median | std |
| --- | --- | ---- | ------ | --- |
| 13.1386247623692 | 57697.9031000326 | 817.7409081769163258 |164.61697558673097 | 2419.51 |
3540 locations off by more than 100 feet
957 locations off by more than 500 feet
191 over a mile

### Contact

Timothy Norris
University of Miami Libraries
Frost Institute for Data Science and Computing