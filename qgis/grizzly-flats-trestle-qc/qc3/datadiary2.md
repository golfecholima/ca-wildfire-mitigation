# Grizzly Flats Trestle Analysis Data Diary 2

_Revisions based on feedback from USFS GIS specialist._

### Prep
* Set QGIS project CRS to `WGS 84 / Pseudo-Mercator EPSG:3857`
* Import via drag and drop, accepting default CRS transformations: **CHECK DATES W/ ZENTER**
  * `CaldorArea_FACTS_20220128.gdb`: USFS Eldorado National Forest provided January 2022. Completed treatments within the forest and the Trestle Project specifically.
    * Layers:
      * `FACTS_CompletedProjects_MinusTrestle_20220128`
      * `FACTS_TrestleProject_20220128`
  * `tl_2021_06_place`: 2021 Census California place data retrieved Aug. 5, 2022. Includes Grizzly Flats boundary.
  * `TrestleTreatmentUnits.gdb`: USFS Eldorado National Forest provided February 2022. Planned treatments in the Trestle Project.
    * Layers:
      * `Trestle_Decision_Units_201709`
      * `Trestle_DraftEIS_Alt5_BurnOnly`
  * `TrestleProjectBoundary.gdb`: USFS Eldorado National Forest provided April 2022.
* Make a copy of `FACTS_TrestleProject_20220128`, yields `FACTS_TrestleProject_20220128 copy` 

* Naming abbreviations to be followed throughout:
  * tpb = TrestleProjectBoundary
  * gf = tl_2021_06_place
  * ctp = FACTS_TrestleProject_20220128
  * ptp = FACTS_TrestleProject_20220128 copy
  * cop = FACTS_CompletedProjects_MinusTrestle_20220128
  * tdu = Trestle_Decision_Units_201709
  * tbo = Trestle_DraftEIS_Alt5_BurnOnly

* Filtering:
  * tpb: Not applicable, layer shows Trestle Project boundary.
  * gf: `"NAME"='Grizzly Flats'`
    * Include only the town of Grizzly Flats.
  * ctp: `"DATE_COMPLETED">='2018-01-01T00:00:00.000'`
    * Include only treatments that were completed on or after Jan. 1, 2018, the year Trestle broke ground.
  * ptp: `"STATUS" IN ('Completed','Under Contract and In Progress')`
    * Include only treatments that have the status Completed or Under Contract and In Progress. **DOUBLE CHECK IF THIS IS THE CORRECT WAY TO FILTER FOR THESE**
  * cop: `"DATE_COMPLETED">='2007-01-01T00:00:00.000'`
    * Include only treatments that were completed on or after Jan. 1, 2007, the maximum amount of time subject matter experts say treatments in an Eldorado-type forest are viable.
  * tdu: `"Treatment" NOT IN ('Rx Burn','Rx Burn - High Priority','No Understory Burn')`
    * Include only planned treatments other than Rx burn from this dataset per USFS Eldorado National Forest GIS specialist. 'No understory burn' indicates areas not planned for treatment in the Trestle Project and are therefore also filtered out. **< DOUBLE CHECK**
  * tbo: Not applicable, dataset shows all planned understory prescribed burns. Use this set for understory prescribed fire instead of `tdu`.

* Run QGIS function `Fix geometries` on all layers saving as `.gpkg` files with naming convention `fx-[abbreviation]`.

* Calculate the area of every shape of every `fx-` layer using QGIS: **CHECK COLUMNS BEING USED FOR AREA/ACREAGE**
  * Right click layer > Open Attribute Table
    * Open field calculator (Ctrl/Cmd + i)
      * `Output field name` = QGIS_ACRES
      * `Expression` = $area/4046.86
      * Click `OK`
      * Toggle editing mode (Ctrl/Cmd + E)
      * Click `Save`

### Analysis

#### Activity Acres
_Calculate **activity acres** (the term used by Clarke Knight, et al [in this paper](https://static1.squarespace.com/static/545a90ede4b026480c02c5c7/t/61a52861dfd0aa6784b117bb/1638213736182/Knight+et+al.+2022.pdf) for including treatments that overlap) using `View > Show statistical summary` function in QGIS._

* Click `View > Show statistical summary`
* Select each layer and the QGIS_ACRES column in the `Statistics` box in the bottom left corner. Record the sum.

* Results:
  * fx-tpb = 20455
  * fx-gf = 4242
  * fx-ctp = 4396[^1]
  * fx-ptp = 8983[^2]
  * fx-cop = 68134
  * fx-tdu = 5081[^3]
  * fx-tbo = 10131[^3]

#### Footprint Acres
_Flatten and merge the layers to calculate total **footprint acres** (again the term used by Knight et al to describe only the total space treated, not including repeat treatments on the same acreage) by using QGIS `Dissolve` and `Merge vector layers`._

* Run the Dissolve function on each of fx-ctp, fx-cop, fx-tdu, fx-tbo saving each new layer as a .gpkg using the naming convention `d-fx-[abbreviation]`.
* Using the same method for calculating area as in the **Prep** section, calculate the area of each dissolved layer naming the column `d_QGIS_ACRES`.

* Results:
  * d-fx-ctp = 2137[^4]
  * d-fx-ptp = 4740[^5]
  * d-fx-cop = 30743
  * d-fx-tdu = 5080
  * d-fx-tbo = 10119

* Dissolving the `fx-ctp` layer yields footprint acres for complete Trestle Project treatments right away. However, since planned acreage is divided into a prescribed burn set `fx-tbo` and  other treatments `fx-tdu`, we want to combine those sets. We'll do this by merging the two sets and dissolving again.
* Delete all fields except `fid` and `d_QGIS_ACRES` from the dissolved layers by right clicking on each layer and selecting `Layer properties > Fields`. (This is because some of the fields have conflicts when being merged and they're not being used for anything anyway because the dissolve function mashes all the shape components into one big shape. So if anything they're wrong and could cause confusion.)
* Merge the `fx-tbo` and `fx-tdu` layers using the Merge vector layers function. Save using the naming convention `m-d-fx-tbotdu.gpkg`.
* This produces a layer with two shapes. Dissolve again, combining those two shapes. Save as `d-m-d-fx-tbotdu.gpkg`.
* Calculate the acreage again using the method in the **Prep** section.

* Result:
  * d-m-d-fx-tbotdu = 15194[^6]

### Findings:
* Trestle Project completed activity acres = 4396
* Trestle Project completed and in progress footprint acres = 8983
* Trestle Project planned activity acres = 15212
* Trestle project completed footprint acres = 2137
* Trestle Project completed and in progress footprint acres = 4740
* Trestle project planned footprint acres = 15194

[^1]: Trestle Project completed activity acres.
[^2]: Trestle Project completed and in progress activity acres.
[^3]: Trestle Project planned activity acres `fx-tdu + fx-tbo` = `5081 + 10131` = 15212.
[^4]: Trestle project completed footprint acres.
[^5]: Trestle Project completed and in progress footprint acres.
[^6]: Trestle project planned footprint acres. _Note: This is only five acres less than the combined footprint acres of `fx-tbo` and `fx-tdu` (10119 + 5080 = 15199) before any dissolving or merging; that's very little overlap._