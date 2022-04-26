# Source
This is where our source data lives. It is raw and shall never be touched other than to be read or copied.

In our case these files will likely live locally or in the cloud somewhere other than GH, and we will list reference URLs with descriptions in this file.

## USFS
- FACTS: https://data.fs.usda.gov/geodata/edw/datasets.php?xmlKeyword
  - TH: https://data.fs.usda.gov/geodata/edw/edw_resources/fc/S_USA.Activity_TimberHarvest.gdb.zip
  - HFT: https://data.fs.usda.gov/geodata/edw/edw_resources/fc/S_USA.Activity_HazFuelTrt_PL.gdb.zip (rxfire_facts_hf in R code is just a subset of facts_hft filtered on method = 'Prescribed Burn')

## CalFire
- THP: https://map.dfg.ca.gov/metadata/ds0816.html
- NTMP: https://gis.data.cnra.ca.gov/maps/CALFIRE-Forestry::cal-fire-nonindustrial-timber-management-plans-and-notices-ta83
  - From: https://data.ca.gov/dataset/cal-fire-nonindustrial-timber-management-plans-and-notices-ta832
- Prescribed burns: https://data.ca.gov/dataset/prescribed-burns1/resource/4c7c8d9a-4f47-4175-a5d5-04bb72d1c9d2
- CalMapper: File `FuelTreatments_CALFIRE22_1_public.gdb` from Mark Rosenberg who helped Clarke Knight extract her CalMapper data. This is a pull from their ESRI REST API.

## Census

- States: https://www.census.gov/cgi-bin/geo/shapefiles/index.php?year=2020&layergroup=States+%28and+equivalent%29
- Counties: https://www.census.gov/cgi-bin/geo/shapefiles/index.php?year=2021&layergroup=Counties+%28and+equivalent%29