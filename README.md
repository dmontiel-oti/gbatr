
<!-- README.md is generated from README.Rmd. Please edit that file -->

# gbatr

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
<!-- badges: end -->

**gbatr** is an interface to [Geosupport Dektop
Edition](https://www1.nyc.gov/site/planning/data-maps/open-data/dwn-gde-home.page),
a batch geocoder for New York City created by the NYC Department of City
Planning. It enables very fast local geocoding of NYC addresses. This is
a fork of the [rGBAT16AB pacakge](https://github.com/gmculp/rGBAT16AB)
developed by GM Culp.

*NOTE: This is very much a work-in-progress and is very likely to
change. If you would like to help with development, please send me a
message or open an issue\!*

## Example

To use gbatr to geocode addresses, first prepare a data frame that
contains a column of street addresses and a column with either zip codes
or boroughs. Note, this only works for addresses in New York City\! The
zip or borough column must all zip codes or all boroughs, not a mix of
both.

Then, call the `gbat()` function and specify the input data frame, the
name of the the address column, the name of the zip or borough column,
whether you are using zip codes or boroughs, and which geocoded fields
you want to get back from the GBAT geocoder.

``` r
library(gbatr)

za <- tibble::tribble(
  ~name,                 ~address,              ~borough,
  "Roberta's",           "261 Moore Street",    "Brooklyn",
  "L'Industrie",         "254 S 2nd Stret",     "Brooklyn",
  "Emmy Squared",        "364 Grand Street",    "Brooklyn",
  "Di Fara",             "1424 Avenue J",       "Brooklyn",
  "L&B Spumoni Gardens", "2725 86th Street",    "Brooklyn",
  "Totonno's",           "1524 Neptune Avenue", "Brooklyn"
  )

za_geo <- gbat(
  za,
  address = "address",
  zip_boro = "borough",
  zip_boro_type = "boro",
  geo_colnames = "lot_centroid_latlon"
  )

za_geo
#> # A tibble: 6 x 6
#>   name              address           borough F1E_GRC F1A_Latitude F1A_Longitude
#>   <chr>             <chr>             <chr>   <chr>   <chr>        <chr>        
#> 1 Roberta's         261 Moore Street  Brookl~ 00      40.705171    -73.934116   
#> 2 L'Industrie       254 S 2nd Stret   Brookl~ 00      40.711481    -73.957848   
#> 3 Emmy Squared      364 Grand Street  Brookl~ 00      40.712164    -73.955708   
#> 4 Di Fara           1424 Avenue J     Brookl~ 00      40.624923    -73.961487   
#> 5 L&B Spumoni Gard~ 2725 86th Street  Brookl~ 00      40.594672    -73.981269   
#> 6 Totonno's         1524 Neptune Ave~ Brookl~ 00      40.578802    -73.983811
```

## Why gbatr?

Geocoding is the process of converting places or addresses to geographic
points, often latitudes and longitudes or x and y coordinates. It is a
common process for geographers and data scientists because most often
data sets contain street addresses of points of interest. In order to
make maps and perform spatial analysis, these addresses must be first
converted to geographic coordinates.

There are many tools to turn addresses into points. Some R packages that
have geocoding functionality include
[**ggmap**](https://github.com/dkahle/ggmap),
[**censusxy**](https://slu-opengis.github.io/censusxy/index.html), and
[**tidygeocoder**](https://jessecambon.github.io/tidygeocoder/). These
are all good options, but one major drawback of all of them is that they
use web APIs to perform the geocoding. This means that you send your
addresses (either one by one or in a batch) to someone’s webserver and
you get back the geocoded points. Two consequences of this are 1) that
large requests may be quite slow and 2) you may not be able to ensure
the privacy of the addresses you send out (if they are sensitive). A
third concern, depending on which geocoding service you use, is cost. In
particular, Google has recently changed the cost structure of their
[Geocoding API
service](https://developers.google.com/maps/documentation/geocoding/usage-and-billing).

Given these challenges, it is fortunate that a fast, free batch geocoder
that can be run locally exists. It is called [**Geosupport Desktop
Edition**](https://www1.nyc.gov/site/planning/data-maps/open-data/dwn-gde-home.page)
and was created by the [New York City Department of City
Planning](https://www1.nyc.gov/site/planning/index.page). The major
downside of this geocoder is that it only works for New York City
address.

City Planning makes the geocoder available as a Windows and Linux
desktop application (there are server version as well), in which you
specify an Excel workbook (or other file type) of address, set some
parameters, and the geocoder adds the geocoded information to a new
sheet in the workbook. One issue with this method of using Geosupport is
that you can’t script or automate this process. If you are trying to
develop a data pipeline in which one step is geocoding addresses, you
would have to export your data, manually perform the geocoding, import
the data, and continue your processing. So, if geocoding is a common
task, gbatr can greatly simplify the task by using Geosupport directly
from R with a data frame input and output.

## Installation

Before you install gbatr, you must have Geosupport Desktop Edition
installed. Geosupport is a free geocoding application developed by the
NYC Department of City Planning. It is available for Windows and Linux
and can be downloaded from the [City Planning
website](https://www1.nyc.gov/site/planning/data-maps/open-data/dwn-gde-home.page).
gmculp's R package (https://github.com/gmculp/rGBAT16AB) from which this package
was written for use with Windows. The source has code has been updated to work
with Linux. 

When installing Geosupport Desktop, it is strongly recommended to use
the path '/opt/version-22a_22.11 . If you install it in this location, building gbatr from
source should just work. If you can’t install it in this location, you
may have to do some additional work to build gbatr.

Once you have Geosupport Desktop Edition installed, the easiest way to
install gbatr is by using the [remotes
package](https://remotes.r-lib.org/). In order to install using remotes,
you will likely need to set an [environment variable to allow warnings
during
installation](https://github.com/r-lib/remotes#environment-variables)
before you try to install:


Download and install the latest version of the linux desktop geosupport version.
```
sudo cd /opt 
sudo wget https://www1.nyc.gov/assets/planning/download/zip/data-maps/open-data/linux_geo22a1_22_11.zip
sudo unzip linux_geo22a1_22_11.zip 
sudo rm linux_geo22a1_22_11.zip
sudo ln -s /opt/version-22a_22.11/lib/* /usr/lib/R/lib/
sudo chmod a+rx /usr/lib/R/lib/*
sudo R -e 'remotes::install_github("dmontiel-oti/gbatr")'
```

## Using `gbat()`

### Specify return columns

By default, all 203 fields from GBAT functions 1A, 1E, and AP are
returned when you call `gbat()`. You probably do not want all these
columns in your geocoded data frame, so there are a few approaches to
specifying which fields to return.

#### Presets

First, there are a handful of “presets” you can specify that return the
most frequently used GBAT fields.

| Preset                  | GBAT Fields                        |
| :---------------------- | :--------------------------------- |
| `"lot_centroid_xy"`     | F1A\_Xcoordinate, F1A\_Ycoordinate |
| `"lot_centroid_latlon"` | F1A\_Latitude, F1A\_Longitude      |
| `"block_face_xy"`       | F1E\_XCoordinate, F1E\_YCoordinate |
| `"block_face_latlon"`   | F1E\_Latitude, F1E\_Longitude      |
| `"cd"`                  | F1E\_CommunityDistrict             |
| `"nta"`                 | F1E\_NTA, F1E\_NTAName             |
| `"census_tract"`        | F1E\_2010CensusTractGEOID          |
| `"census_block"`        | F1E\_2010CensusBlockGEOID          |

You can use one of more of the presets by specifying them in the
`geo_colnames` argument.

``` r
gbat(
  za,
  address = "address",
  zip_boro = "borough",
  zip_boro_type = "boro",
  geo_colnames = c("cd", "nta", "census_tract")
  )
#> # A tibble: 6 x 8
#>   name  address borough F1E_GRC F1E_CommunityDi~ F1E_NTA F1E_NTAName
#>   <chr> <chr>   <chr>   <chr>   <chr>            <chr>   <chr>      
#> 1 Robe~ 261 Mo~ Brookl~ 00      301              BK78    Bushwick S~
#> 2 L'In~ 254 S ~ Brookl~ 00      301              BK73    North Side~
#> 3 Emmy~ 364 Gr~ Brookl~ 00      301              BK73    North Side~
#> 4 Di F~ 1424 A~ Brookl~ 00      314              BK43    Midwood    
#> 5 L&B ~ 2725 8~ Brookl~ 00      315              BK29    Bensonhurs~
#> 6 Toto~ 1524 N~ Brookl~ 00      313              BK21    Seagate-Co~
#> # ... with 1 more variable: F1E_2010CensusTractGEOID <chr>
```

#### Column names

`gbat_fields` is a data set built into gbatr that contains all the
possible GBAT fields you can request. If you want to specify one or more
of these fields to be included in the `gbat()` output, use the name of
that field from the `col_names` column in the `gbat_fields` data set.

``` r
head(gbat_fields)
#> # A tibble: 6 x 6
#>   field              func  col_name               length start   end
#>   <chr>              <chr> <chr>                   <int> <int> <int>
#> 1 BoroName           F1E   F1E_BoroName                9   361   369
#> 2 HouseNumberDisplay F1E   F1E_HouseNumberDisplay     16   370   385
#> 3 HouseNumberSort    F1E   F1E_HouseNumberSort        11   386   396
#> 4 Street_B10SC       F1E   F1E_Street_B10SC           11   397   407
#> 5 Street_Name        F1E   F1E_Street_Name            32   408   439
#> 6 BBL                F1E   F1E_BBL                    10   526   535
```

You can combine these column names with the presets discussed above.

``` r
gbat(
  za,
  address = "address",
  zip_boro = "borough",
  zip_boro_type = "boro",
  geo_colnames = c("lot_centroid_latlon", "F1E_BBL")
  )
#> # A tibble: 6 x 7
#>   name         address       borough F1E_GRC F1E_BBL  F1A_Latitude F1A_Longitude
#>   <chr>        <chr>         <chr>   <chr>   <chr>    <chr>        <chr>        
#> 1 Roberta's    261 Moore St~ Brookl~ 00      3031010~ 40.705171    -73.934116   
#> 2 L'Industrie  254 S 2nd St~ Brookl~ 00      3024200~ 40.711481    -73.957848   
#> 3 Emmy Squared 364 Grand St~ Brookl~ 00      3023960~ 40.712164    -73.955708   
#> 4 Di Fara      1424 Avenue J Brookl~ 00      3067160~ 40.624923    -73.961487   
#> 5 L&B Spumoni~ 2725 86th St~ Brookl~ 00      3071160~ 40.594672    -73.981269   
#> 6 Totonno's    1524 Neptune~ Brookl~ 00      3070220~ 40.578802    -73.983811
```

#### Geosupport functions

A third option for choosing return columns is to use the `func`
argument. You can read all about Geosupport Functions in the [Geosupport
doucmentation](https://nycplanning.github.io/Geosupport-UPG/appendices/appendix01/#introduction).Function
1A and Function 1E return slightly different versions of similar
information. For instance, you might have noticed that there are both 1A
and 1E latitude and longitude fields. There is lots of detail in the
Geosupport manual, but the important thing to know is that Function 1A
returns tax lot- and building-level data while Function 1E returns block
face-level data. In practice this means that when you geocode an
address, the coordinates returned by F1A will generally be the centroid
of the tax lot, while the coordinates returned by F1E will be a point on
the street. For smaller, residenital buildings, this generally does not
make a big difference, but if you are geocoding an address for a large
housing development with multiple buildings, there may be larger
differences between the points returned by 1A and 1E.

Here, for example, we get latitude and longitude for our pizza places
from both 1A and 1E. Note the small differences between the latidues and
longitudes returned.

``` r
gbat(
  za,
  address = "address",
  zip_boro = "borough",
  zip_boro_type = "boro",
  geo_colnames = c("lot_centroid_latlon", "block_face_latlon")
  )
#> # A tibble: 6 x 8
#>   name  address borough F1E_GRC F1E_Latitude F1E_Longitude F1A_Latitude
#>   <chr> <chr>   <chr>   <chr>   <chr>        <chr>         <chr>       
#> 1 Robe~ 261 Mo~ Brookl~ 00      40.704825    -73.934015    40.705171   
#> 2 L'In~ 254 S ~ Brookl~ 00      40.711728    -73.957887    40.711481   
#> 3 Emmy~ 364 Gr~ Brookl~ 00      40.712331    -73.955679    40.712164   
#> 4 Di F~ 1424 A~ Brookl~ 00      40.625105    -73.962020    40.624923   
#> 5 L&B ~ 2725 8~ Brookl~ 00      40.594401    -73.981359    40.594672   
#> 6 Toto~ 1524 N~ Brookl~ 00      40.579118    -73.983430    40.578802   
#> # ... with 1 more variable: F1A_Longitude <chr>
```

### Dealing with failure

### Converting to `sf` objects and mapping

``` r
za_geo %>%
  sf::st_as_sf(
    coords = c("F1A_Longitude", "F1A_Latitude"),
    crs = 4326
    ) %>%
  mapview::mapview()
```
