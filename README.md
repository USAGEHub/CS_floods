# Description

The tool elaborates the data about flood-impacted locations collected with speditive surveys in order to derive their spatial extensione and estimate their water volume. 
Flooded locations are surveyed as lines: features made of 3 or more vertexes that end up within a custom distance from where they began are considered encircling a flooded area, while the others are considered "linear floods" with a lateral size derived from the field "Larghezza", valued during the survey. A datased describing building footprints is required in order to avoid considering building parts as part of the flooded areas.
Once the surface extension of every surveyed feature is established, features surveyed on the same day and having a spatial overlap are merged, to avoid counting the same area more than once. The depth of flooded areas and ultimately their volume is determined using a local DTM.


# Input data requirements

* Flood-impacted locations, vector dataset with linestring geometry and a decimal field named "Larghezza"
* Building footprints
* Digital Terrain Model (DTM)
* Model parameters:
   * spatial reference system for the surveyed locations (default 4326)
   * maximum vertex distance for a line encircling a flooded area (default 10 meters)  


# Output data requirements

* Flood-impacted locations (elaborated), with polygon geometry and an additional boolean field named "Lineare"
* Estimation of flooded volumes with a polygon geometry and the following fields:
   * "Giorno": the survey date
   * "Area_m2": the extension of the flooded surface
   * "Quota_min_m": minimum DTM elevation within the extension of the flooded surface
   * "Quota_max_m": maximum DTM elevation within the extension of the flooded surface
   * "Diff_quota_cm": the estimated depth of the flooded volume
   * "Stima_volume_m3": the estimated value of the flooded volume


# Objective

Support the impact evaluation of floods by the local municipalities by providing a quantitative estimate of flooded volumes, their location, extent and recurrence.


# Use of resource

**First part**
* Assigns a spatial reference system to the surveyed dataset
* Converts it to a known projected system 
* Determines for each surveyed linestring if it is "almost close", measuring if it ends up within a custom distance from where it begins
* Flags as "linear" all features that are NOT "almost close" OR that are made of 2 or less vertexes
* Bufferizes linear features osing the value in field "Larghezza" as size
* Transforms non-linear features in poligons by "closing" their linestring geometry ad applying a buffer of 0.2 meters.
* Merge linear and non-linear features
* Intersect the poligonized features with the building footprints and remove from the first all overlapping areas 

**Second part**
* Extract day from timestamp of survey
* Dissolve overlapping polygons related to the same day of survey
* Computes minumun (h_min) and maximum (h_max) DEM elevation for each dissolved polygon
* Computes the area (A) of each dissolved polygon
* Estimates the depth (h) of the flooded volume corresponding to each dissolved polygon by assuming it corresponds to the difference between max and min DEM heigh: h = h_max - h_min
* Estimates the flooded volume (V) corresponding to each dissolved polygon by assimilating the volume to a cone of base (A) and height (h): V = A * h / 3
