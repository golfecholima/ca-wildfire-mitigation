# Grizzly Flats Trestle Analysis Data Diary 2

_Revisions based on feedback from USFS GIS specialist._

### Prep

* Set QGIS project CRS to `WGS 84 / Pseudo-Mercator EPSG:3857`
* Import via drag and drop, accepting default CRS transformations:
  * `CaldorArea_FACTS_20220128.gdb`: USFS Eldorado National Forest provided Jan. 28, 2022. Completed treatments within the forest and the Trestle Project specifically.
    * Layers:
      * `FACTS_CompletedProjects_MinusTrestle_20220128`
      * `FACTS_TrestleProject_20220128`
  * `tl_2021_06_place`: 2021 Census California place data retrieved Aug. 5, 2022. Includes Grizzly Flats boundary.
  * `TrestleTreatmentUnits.gdb`: USFS Eldorado National Forest provided Feb. 17, 2022. Planned treatments in the Trestle Project.
    * Layers:
      * `Trestle_Decision_Units_201709`
      * `Trestle_DraftEIS_Alt5_BurnOnly`
  * `TrestleProjectBoundary.gdb`: USFS Eldorado National Forest provided April 29, 2022.

* Naming abbreviations to be followed throughout:
  * tpb = TrestleProjectBoundary
  * gf = tl_2021_06_place
  * ctp = FACTS_TrestleProject_20220128
  * cop = FACTS_CompletedProjects_MinusTrestle_20220128
  * tdu = Trestle_Decision_Units_201709
  * tbo = Trestle_DraftEIS_Alt5_BurnOnly

* Filtering:
  * tpb: Not applicable, layer shows Trestle Project boundary.
  * gf: `"NAME"='Grizzly Flats'`
    * Include only the town of Grizzly Flats.
  * ctp: `"DATE_COMPLETED">='2018-01-01T00:00:00.000'`
    * Include only treatments that were completed on or after Jan. 1, 2018, the year Trestle broke ground.
  * cop: `"DATE_COMPLETED">='2007-01-01T00:00:00.000'`
    * Include only treatments that were completed on or after Jan. 1, 2007, the maximum amount of time subject matter experts say treatments in an Eldorado-type forest are viable.
  * tdu: `"Treatment" NOT IN ('Rx Burn','Rx Burn - High Priority','No Understory Burn')`
    * Include only planned treatments other than Rx burn from this dataset per USFS Eldorado National Forest GIS specialist. 'No understory burn' indicates areas not planned for treatment in the Trestle Project and are therefore also filtered out.
  * tbo: Not applicable, dataset shows all planned understory prescribed burns. Use this set for understory prescribed fire instead of `tdu`.
  * cp819: Not applicable, this data includes only the fire perimeter on Aug. 19 as reported by the U.S. Forest Service.

* Run QGIS function `Fix geometries` on all layers saving as `.gpkg` files with naming convention `fx-[abbreviation]`.

* Calculate the area of every shape of every `fx-` layer using QGIS:
  * Right click layer > Open Attribute Table
    * Open field calculator (Ctrl/Cmd + i)
      * `Output field name` = QGIS_ACRES
      * `Output field type` = Decimal number (real)
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
  * fx-ctp = 4399[^1]
  * fx-cop = 68176
  * fx-tdu = 5082[^2]
  * fx-tbo = 10127[^2]

#### Footprint Acres

_Flatten and merge the layers to calculate total **footprint acres** (again the term used by Knight et al to describe only the total space treated, not including repeat treatments on the same acreage) by using QGIS `Dissolve` and `Merge vector layers`._

* Run the Dissolve function on each of fx-ctp, fx-cop, fx-tdu, fx-tbo saving each new layer as a .gpkg using the naming convention `d-fx-[abbreviation]`.
* Using the same method for calculating area as in the **Prep** section, calculate the area of each dissolved layer naming the column `d_QGIS_ACRES`.

* Results:
  * d-fx-ctp = 2137[^3]
  * d-fx-cop = 30743
  * d-fx-tdu = 5080
  * d-fx-tbo = 10119

* Dissolving the `fx-ctp` layer yields footprint acres for complete Trestle Project treatments right away. However, since planned acreage is divided into a prescribed burn set `fx-tbo` and  other treatments `fx-tdu`, we want to combine those sets. We'll do this by merging the two sets and dissolving again.
* Delete all fields except `fid` and `d_QGIS_ACRES` from the dissolved layers by right clicking on each layer and selecting `Layer properties > Fields`. (This is because some of the fields have conflicts when being merged and they're not being used for anything anyway because the dissolve function mashes all the shape components into one big shape. So if anything they're wrong and could cause confusion.)
* Merge the `fx-tbo` and `fx-tdu` layers using the Merge vector layers function. Save using the naming convention `m-d-fx-tbotdu.gpkg`.
* This produces a layer with two shapes. Dissolve again, combining those two shapes. Save as `d-m-d-fx-tbotdu.gpkg`.
* Calculate the acreage again using the method in the **Prep** section.

* Result:
  * d-m-d-fx-tbotdu = 15194[^4]

#### Buffers, clips and slicing the data.

_Create a five mile perimeter from the border of Grizzly Flats to evaluate treatments beyond the Trestle Project perimeter. Use the QGIS function `Buffer` to create the five mile perimeter and the `Clip` function to limit the various treatments to within that buffer._

**Create buffers**
* Right click the `fx-gf` layer and select `Export > Save Features As`.
* Enter the following:
  * File name: `epsg-3857-fx-gf`
  * Layer name: `epsg-3857-fx-gf`
  * CRS: `EPSG:3857 - WGS 84 / Pseudo-Mercator`
  * Leave the rest to defaults
  * Check the `Add saved file to map` box
  * Click `OK`
  * This adds a layer that changes the coordinate reference system of the `fx-gf` layer to be able to do calculations in meters rather than degrees, which matters for the upcoming buffer calculation.
* Use the `Buffer` function with the following parameters:
  * Input layer: `epsg-3857-fx-gf`
  * Distance: `8046.72` (The number of meters in five miles.)
  * Segments: `20`
  * End cap style: `Round`
  * Join style: `Round`
  * Miter limit: `2`
  * Save as: `b5-epsg-3857-fx-gf`
  * Click `Run`
  * Result is the Grizzly Flats town border extended in all directions by five miles.

**Clip using the buffer**
* Run the `Clip` function on the `fx-cop` and the `fx-ctp` layer using the `fx-` layers as the input and the `b5-epsg-3857-fx-gf` layer as the overlay. Save each as `cb5-[INPUT LAYER NAME]`. This provides completed treatments that fall within a five mile buffer of Grizzly Flats.
* Calculate the activity acres using the field calculator and show statistical summary, as above.
  
* Results:
  * cb5-fx-cop = 10735.7 activity acres
  * cb5-fx-ctp = 4399.14 activity acres
  * Combined = 15135 activity acres

* Use `Merge`, `Dissolve`, and the field calculator as above to calculate footprint acres. Save using the same `d-` and `m-` naming conventions.

  * Result:
    * d-m-cb5-fx-ctpcop = 5845

_Note: This same buffer and clip method was used to calculate various figures in the story and as a gut check various source claims and our own assumptions. We also frequently exported the tabular data from these GIS files and used pivot tables to evaluate the prevalence and breakdown of different treatment types._ 

### Findings:

#### As seen in: Stalled U.S. Forest Service project could have protected California town from Caldor Fire destruction
* The Trestle Project planned to treat 15209 acres (total planned treatment acres including overlapping treatments)
* The Trestle Project planned footprint acres = 15194 (total planned treatment acres less any overlapping treatment areas)
* The Forest Service completed 2137 footprint acres of the Trestle Project or 14% of the total planned footprint acres.[^5]
* The Forest Service completed 1182 footprint acres from 2003 to 2007 within 1 mile buffer off southern border. (Includes the entirety of any treatment that was at least partially within that border).
* From Jan. 1, 2007 to the Caldor Fire ignition date, the Forest Service completed 15135 activity acres within a five mile Grizzly Flats buffer. The majority of that acreage happened beyond the a three mile buffer.

[^1]: Trestle Project completed activity acres.
[^2]: Trestle Project planned activity acres `fx-tdu + fx-tbo` = `5082 + 10127` = 15209.
[^3]: Trestle Project completed footprint acres.
[^4]: Trestle Project planned footprint acres.
[^5]: The same methodology was used on the Forest Service's public-facing database "FACTS". The result was 4021 acres and 24% of the planned footprint, which prompted the agency to realize that several treatments in FACTS had been erroneously marked complete. That process is [available here](https://github.com/golfecholima/ca-wildfire-mitigation/blob/main/qgis/grizzly-flats-trestle-qc/datadiary.md) NOTE: DO NOT PUBLISH ANY FIGURES FROM THAT DOCUMENT, THERE IS A HIGH LIKELIHOOD THEY ARE WRONG.