# Big Bear Complete and Planned Hazardous Fuels Treatment Analysis Data Diary

### Prep
* Download USFS data from: https://data.fs.usda.gov/geodata/edw/datasets.php

    * [Hazardous Fuel Treatment Reduction: Polygon — ESRI geodatabase](https://data.fs.usda.gov/geodata/edw/edw_resources/fc/S_USA.Activity_HazFuelTrt_PL.gdb.zip)
    * [NEPA Project Area Boundaries — ESRI geodatabase](https://data.fs.usda.gov/geodata/edw/edw_resources/fc/S_USA.Actv_ProjectArea_NEPA.gdb.zip)
    * [North Big Bear Fuels Reduction Project](https://ftp.wildfire.gov/public/incident_specific_data/Fuels/CA_BDF/Shapefiles/NBB_PROJECT_BOUNDARY_20200805.zip)
    * [Santa Ana Watershed Hazardous Fuels Reduction Project](https://ftp.wildfire.gov/public/incident_specific_data/Fuels/CA_BDF/Shapefiles/SouthForkPerimeter_south38andBartonFlats_07062022.zip)
    * [Ranger District Boundaries — ESRI geodatabase](https://data.fs.usda.gov/geodata/edw/edw_resources/fc/S_USA.RangerDistrict.gdb.zip)
    _Note: Exclusion of timber harvest data is based on feedback from San Bernardino National Forest FACTS specialist that all timber harvest treatments intended to address fuels reduction are duplicated in the Hazardous Fuel Treatment dataset._
    * Hazardous fuel downloaded on 7/6/22, NEPA projects on 7/15/22, North Big Bear and Santa Ana projects on 7/29/22, and Ranger District on 7/25/22.

* Drag and drop import to new QGIS document

* Accept default QGIS CRS transformation

* Abbreviated naming convention:
  * haz = Hazardous Fuel Treatment Reduction
  * nepa = NEPA Project Area Boundaries
  * nbb = North Big Bear Fuels Reduction Project
  * saw = Santa Ana Watershed Hazardous Fuels Reduction Project
  * rd = Ranger District Boundaries

* Filtering:
  * haz: `"PROC_REGION_CODE"='05' AND "PROC_FOREST_CODE"='12' AND "ADMIN_DISTRICT_CODE"='52' AND "DATE_COMPLETED">='2007-01-01T00:00:00.000'`
  * nepa: `"ADMIN_DISTRICT_CODE"='52' AND "ADMIN_FOREST_CODE"='12' AND "NAME" LIKE '%FUEL%'`
  * nbb: na
  * saw: na
  * rd: `"DISTRICTNUMBER"='52' AND "FORESTNAME" = 'San Bernardino National Forest'`
  _Note: NEPA data includes projects that are beyond the scope of Hazardous Fuels Treatment, hence filtering for the word 'FUEL'._

* Run `Fix geometries` on all datasets save output as .gpkg with naming convention `fx-[ABBREVIATED NAME]`

### Analysis

#### Activity Acres
_Calculate **activity acres** (Clarke Knight's term for including treatments that overlap) using the `Show statistical summary` function in QGIS._

* Select the following layer and column combos in the statistical summary box in the bottom left corner. Record the sum:

* Results:
  * `fx-haz` > `GIS_ACRES` = 28432 
  * `fx-nepa` > `GIS_ACRES` = 8856
  * `fx-nbb` > `GIS_ACRES` = 12919
  * `fx-saw` > `acres` = 5887

#### Footprint Acres
_'Flatten' layers to calculate total **footprint acres** (only the total space treated, not including repeat treatments on the same area) by using the QGIS `Dissolve` function._

* Run the Dissolve function on each of the `fx` layers saving each layer as a .gpkg using the naming convention `d-fx-[ABBREVIATED NAME]`.

* Calculate the total area of each `d-fx-` layer using `Open attribute table > Open field calculator` 

  * Enter the output field name `QGIS_ACRES`

  * In the Expression box enter `$area/4046.86` (4046.86 is the number of square meters in an acres — QGIS defaults to square meters.)

* Results:
  * `fx-haz` = 8024 
  * `fx-nepa` = 8731
  * `fx-nbb` = 12929
  * `fx-saw` = 5891

* Create a layer that subtracts the `d-fx-haz` layer from the `d-fx-nepa` layer using the QGIS function `Difference` with the NEPA layer as the input and the HAZ layer as the overlay. Label it `d-fx-nepa-less-d-fx-haz`.
  * This will enable the calculation of the footprint acres of NEPA projects that have not had recent treatments and/or are not part of ongoing maintenance treatments.
* Calculate the footprint acres as above.

* Results:
  * `d-fx-nepa-less-d-fx-haz` = 4909