# Grizzly Flats Trestle Analysis Data Diary

### Prep
* Download USFS data from: https://data.fs.usda.gov/geodata/edw/datasets.php

    * [Hazardous Fuel Treatment Reduction: Polygon — ESRI geodatabase](https://data.fs.usda.gov/geodata/edw/edw_resources/fc/S_USA.Activity_HazFuelTrt_PL.gdb.zip)

    * [Timber Harvests — ESRI geodatabase](https://data.fs.usda.gov/geodata/edw/edw_resources/fc/S_USA.Activity_TimberHarvest.gdb.zip)

    * Both downloaded on 7/6/22

* Drag and drop import to new QGIS document

* Accept default QGIS CRS transformation

* Filter both sets: `"STATE_ABBR" = 'CA' AND "DATE_COMPLETED" >= '2002-01-01T00:00:00.000'`

* Run `Fix Geometry` on both sets save output as .gpkg `fx-haz-2002` and `fx-th-2002`

* Drag and drop import Trestle Project (TP) boundary file, sourced from USFS Eldorado National Forest, accept QGIS default CRS transform. `TrestleProjectBoundary.gdb.zip`
  
* Drag and drop import burn severity data provided by USFS: `121721_CaldorTrestleDataRequest > Caldor20211215.gdb` using only the `Caldor_SoilBurnSeverity` layer. More information about what this type of burn severity analysis indicates is [available here](https://inciweb.nwcg.gov/incident/article/7842/66601/).

* Repair the layer using `Fix geometries` saving the layer as `fx-burn-severity` using the .gpkg extension.

### Analysis
* Run `Overlap Analysis` on each `fx-[th/haz]` layer using TP boundary as the overlay layer.

  * Save output as `overlap-fx-haz-2002` and `overlap-fx-th-2002` using .gpkg extension

* Filter both overlap layers by `"TrestleProjectBoundary_pc">=25`

  * This should narrow the data to all projects that were at least 25% within the Trestle Project boundaries and completed after Jan 1, 2002.


#### Activity Acres
_Calculate **activity acres** (Clarke Knight's term for including treatments that overlap) using the `Show statistical summary` function in QGIS._

* Select the following layer and column combos in the statistical summary box in the bottom left corner. Record the sum:

* Results:

  * `overlap-fx-haz-2002`

    * `GIS_ACRES` = 12799 

  * `overlap-fx-th-2002`

    * `GIS_ACRES` = 5883

* Repeat the above calculations by duplicating each overlap layer thrice and filtering for projects completed on or after Jan 1, 2018 on one pair, projects completed on or after Jan 1, 2009 on the second pair, and projects completed on or after Jan 1, 2007 for the third pair.

  * The filters should look like this: `"DATE_COMPLETED">='2007-01-01T00:00:00.000' AND " TrestleProjectBoundary_pc">=25` and `"DATE_COMPLETED">='2009-01-01T00:00:00.000' AND " TrestleProjectBoundary_pc">=25` and `"DATE_COMPLETED">='2018-01-01T00:00:00.000' AND " TrestleProjectBoundary_pc">=25`

  * Rename each layer with the corresponding year: `overlap-fx-haz-2007`, `overlap-fx-th-2009`, etc.

* Results:

  * From Jan 1, 2007

    * `overlap-fx-haz-2007`

      * `GIS_ACRES` = 10500

    * `overlap-fx-th-2007`
  
      * `GIS_ACRES` = 3894

      * `NBR_ACRES_ACCOMPLISHED` = 3865

  * From Jan 1, 2009

    * `overlap-fx-haz-2009`

      * `GIS_ACRES` = 9881 

    * `overlap-fx-th-2009`

      * `GIS_ACRES` = 3545

  * From Jan 1, 2018

    * `overlap-fx-haz-2018`

      * `GIS_ACRES` = 9272 

    * `overlap-fx-th-2018`

      * `GIS_ACRES` = 3476


#### Footprint Acres
_Flatten and merge the layers to calculate total **footprint acres** (only the total space treated, not including repeat treatments on the same area) by using QGIS `Dissolve` and `Merge vector layers`._

* Run the Dissolve function on each of the overlap layers (eight layers in all, a `th` and `haz` for start dates in 2002, 2007, 2009, and 2018) saving each layer as a .gpkg using the naming convention `d-overlap-fx-[th/haz]-[YYYY]`

* Delete all fields except `OBJECTID` and `fid` from the dissolved layers using `Layer properties > Fields` dialog. (This is because some of the fields have conflicts when being merged and they're not being used for anything anyway because the dissolve function mashes all the shape components into one big shape.)

* For each year, merge the `th` and `haz` layers the Merge Vector Layers function. Save using the naming convention `m-d-overlap-fx-thhaz-[YYYY].gpkg`

**Note:** Merged layers have two rows. This means we need to dissolve again on each merged layer using the naming convention. `d-m-d-overlap-fx-thhaz-[YYYY].gpkg`

* Calculate the total area of each `d-m-d-` layer using `Open attribute table > Open field calculator` 

  * Enter the output field name `QGIS_ACRES`

  * In the Expression box enter `$area/4046.86` (4046.86 is the number of square meters in an acres — QGIS defaults to square meters)

* Results
  * `d-m-d-overlap-fx-thhaz-2018` = 4021 total footprint acres

  * `d-m-d-overlap-fx-thhaz-2009` = 4397 total footprint acres

  * `d-m-d-overlap-fx-thhaz-2007` = 4760 total footprint acres

  * `d-m-d-overlap-fx-thhaz-2002` = 7119 total footprint acres


#### Soil Burn Severity
_Calculate the soil burn severity in the treated areas and compare with burn severity in untreated areas using the `Split vector layer`, `Clip` and `Intersection` functions._

* Filter the `fx-burn-severity` layer to just the Trestle Project Boundary using the Clip function with the burn severity layer as the Input and the Trestle Project boundary layer as the overlay. Save it as `fx-burn-severity-clip-trestle` using the .gpkg extension.

* Split that layer into separate layers using the `gridcode` column within the Split vector layer QGIS function, saving to a folder called `sbs-split`.

* From that folder, import the four `gridcode_[#].gpkg` files using drag and drop. Rename in ascending order to `Unburned/Very Low`, `Low`, `Medium`, `High`.

* Create a baseline burn severity index for the Trestle Project area by calculating the acreage of each burn severity category and dividing it by the acreage of the whole Trestle Project, each time using the same field calculator method as in the last section to calculate the acres of each of the five layers with `$area/4046.86`.

**Baseline**
* Results:
  * `Trestle Project Boundary`: 20455 acres, 100 pct
  * `High`: 8366 acres, 41 pct
  * `Med`: 9285 acres, 45 pct
  * `Low`: 2552 acres, 12 pct
  * `Unburned/Very Low`: 197 acres, 1 pct

**Note**: Percentages and acres do not add up to 100 and 20455 respectively because the Caldor Fire spared a small piece of the Western tip of the Trestle Project boundary. 

* Use the Clip function on each `d-m-d-` layer by using the `d-m-d-` layer as the Input and one of the four split burn severity layers as the overlay. This should create 16 new layers in total: four burn severity categories times four year layers is 16 burn year-severity pairings. Save each new layer using the `.gpkg` extension and the file naming convention `[YYYY]-[high/med/low/uvl]`.

* As in the Footprint Acres section above, find the acres of each of the 16 new layers using the field calculator and the `$area/4046.86` formula naming the column `QGIS_ACRES_[HIGH/MED/LOW/UVL]` each time. 

* Also add the percentage of the total footprint acres for that year grouping by using the field calculator again, this time creating a new column called `PCT_QGIS_ACRES_[HIGH/MED/LOW/UVL]` and using the formula `$area/4046.86/[total footprint acres figure from above for the corresponding year]*100`.

**Treated Acres**
* Results
  * `2018-high` = 1068 acres, 27 pct
  * `2018-med` = 2487 acres, 62 pct
  * `2018-low` = 452 acres, 11 pct
  * `2018-uvl` = 12 acres, 00 pct

  * `2009-high` = 1104 acres, 25 pct
  * `2009-med` =  2673 acres, 61 pct
  * `2009-low` =  558 acres, 13 pct
  * `2009-uvl` =  31 acres, 01 pct

  * `2007-high` = 1150 acres, 24 pct
  * `2007-med` =  2814 acres, 59 pct
  * `2007-low` =  650 acres, 14 pct
  * `2007-uvl` =  43 acres, 01 pct

  * `2002-high` = 2029 acres, 29 pct
  * `2002-med` = 3923 acres, 55 pct
  * `2002-low` = 748 acres, 11 pct
  * `2002-uvl` = 55 acres, 01 pct

**Note**: Some percentages do not add up to 100 because portions of some older treatments fall outside of the Trestle Project boundary or the Caldor Fire perimeter.

* Calculate the same figures for the untreated areas using the formula: `([HIGH/MED/LOW/UVL] - [YYYY-HIGH/MED/LOW/UVL]) / ([TRESTLE PROJECT TOTAL ACREAGE] - [FOOTPRINTACRES])`.

  * In other words to calculate the percent of high soil burn severity in untreated acres from 2018 onwards: `(High - 2018-high) / (Trestle Project Boundary - d-m-d-overlap-fx-thhaz-2018)` OR `(8366 - 1068) / (20455 - 4021) = 0.44`

_For the next part, I switched to a spreadsheet and let it do the figurin' using all the above QGIS generated numbers._

**Untreated Acres**
* Results:
  * `2018-high (UNTREATED)` = 7298 acres, 44 pct
  * `2018-med (UNTREATED)` = 6798 acres, 41 pct
  * `2018-low (UNTREATED)` = 2100 acres, 13 pct
  * `2018-uvl (UNTREATED)` = 185 acres, 01 pct

  * `2009-high (UNTREATED)` = 7262 acres, 45 pct
  * `2009-med (UNTREATED)` = 6612 acres, 41 pct
  * `2009-low (UNTREATED)` = 1994 acres, 12 pct
  * `2009-uvl (UNTREATED)` = 166 acres, 01 pct

  * `2007-high (UNTREATED)` = 7216 acres, 46 pct
  * `2007-med (UNTREATED)` = 6471 acres, 41 pct
  * `2007-low (UNTREATED)` = 1902 acres, 12 pct
  * `2007-uvl (UNTREATED)` = 154 acres, 01 pct

  * `2002-high (UNTREATED)` = 6337 acres, 48 pct
  * `2002-med (UNTREATED)` = 5362 acres, 40 pct
  * `2002-low (UNTREATED)` = 1804 acres, 14 pct
  * `2002-uvl (UNTREATED)` = 142 acres, 01 pct

**Note**: Some percentages do not add up to 100 because portions of some older treatments fall outside of the Trestle Project boundary or the Caldor Fire perimeter.

* Calculate the relative ratios by divding the untreated percentages by the treated percentages.

* Results:
  * `2018-high (RELATIVE RATIO)` = 1.67
  * `2018-med (RELATIVE RATIO)` = 0.67
  * `2018-low (RELATIVE RATIO)` = 1.14
  * `2018-uvl (RELATIVE RATIO)` = 3.77

  * `2009-high (RELATIVE RATIO)` = 1.80
  * `2009-med (RELATIVE RATIO)` = 0.68
  * `2009-low (RELATIVE RATIO)` = 0.98
  * `2009-uvl (RELATIVE RATIO)` = 1.47

  * `2007-high (RELATIVE RATIO)` = 1.90
  * `2007-med (RELATIVE RATIO)` = 0.70
  * `2007-low (RELATIVE RATIO)` = 0.89
  * `2007-uvl (RELATIVE RATIO)` = 1.09

  * `2002-high (RELATIVE RATIO)` = 1.67
  * `2002-med (RELATIVE RATIO)` = 0.73
  * `2002-low (RELATIVE RATIO)` = 1.29
  * `2002-uvl (RELATIVE RATIO)` = 1.38