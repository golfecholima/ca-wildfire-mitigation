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

### Analysis
* Run `Overlap Analysis` on each `fx-` layer using TP boundary as the overlay layer.

  * Save output as `overlap-fx-haz-2002` and `overlap-fx-th-2002` using .gpkg extension

* Filter both overlap layers by `"TrestleProjectBoundary_pc">=25`

  * This should narrow the data to all projects that were at least 25% within the Trestle Project boundaries and completed after Jan 1, 2002.

#### Activity Acres
* Calculate **activity acres** (Clarke Knight's term for including treatments that overlap) using `Show statistical summary` in QGIS and selecting the following layer and column combos:

* Results:

  * `overlap-fx-haz-2002`

    * `GIS_ACRES` = 12799 

    * `NBR_UNITS_ACCOMPLISHED` = 11901

  * `overlap-fx-th-2002`

    * `GIS_ACRES` = 5883

    * `NBR_UNITS_ACCOMPLISHED` = 5725

* Repeat the above calculations by duplicating each overlap layer twice and filtering for projects completed on or after Jan 1, 2018 on one pair and projects completed on or after Jan 1, 2009 on the second pair.

* The filters should look like this: `"DATE_COMPLETED">='2009-01-01T00:00:00.000' AND " TrestleProjectBoundary_pc">=25` and `"DATE_COMPLETED">='2018-01-01T00:00:00.000' AND " TrestleProjectBoundary_pc">=25`

* Results:

  * From Jan 1, 2009

    * `overlap-fx-haz-2009`

      * `GIS_ACRES` = 9881 

      * `NBR_UNITS_ACCOMPLISHED` = 9464

    * `overlap-fx-th-2009`

      * `GIS_ACRES` = 3545

      * `NBR_UNITS_ACCOMPLISHED` = 3563

  * From Jan 1, 2018

    * `overlap-fx-haz-2018`

      * `GIS_ACRES` = 9272 

      * `NBR_UNITS_ACCOMPLISHED` = 8847

    * `overlap-fx-th-2018`

      * `GIS_ACRES` = 3476

      * `NBR_UNITS_ACCOMPLISHED` = 3495

#### Footprint Acres
* Flatten then merge the layers to calculate total **footprint acres** (only the total space treated, not including repeat treatments on the same area) by using QGIS `Dissolve` and `Merge vector layers`.

* Run the Dissolve function on each of the overlap layers (six layers in all, a `th` and `haz` for start dates in 2002, 2009, and 2018) saving each layer as a .gpkg using the naming convention `d-overlap-fx-[th/haz]-[YYYY]`

* Delete all fields except `OBJECTID` and `fid` from the dissolved layers using `Layer properties > Fields` dialog. (This is because some of the fields have conflicts when being merged and they're not being used for anything anyway because the dissolve function mashes all the shape components into one big shape.)

  * Merge the `th` and `haz` layers, grouping by year using the Merge Vector Layers function. Save using the naming convention `m-d-overlap-fx-thhaz-[YYYY].gpkg`

**Note:** merged layers have two rows. This means we need to dissolve again on each merged layer using the naming convention. `d-m-d-overlap-fx-thhaz-2002.gpkg`

* Calculate the total area of each `d-m-d-` layer using `Open attribute table > Open field calculator` 

  * Enter the output field name `QGIS_ACRES`

  * In the Expression box enter `$area / 4046.86` (4046.86 is the number of square meters in an acres, QGIS defaults to square meters)

* Results
  * `d-m-d-overlap-fx-thhaz-2018` = 4021 total footprint acres

  * `d-m-d-overlap-fx-thhaz-2009` = 4397 total footprint acres

  * `d-m-d-overlap-fx-thhaz-2002` = 7119 total footprint acres
 