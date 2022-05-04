
<!-- README.md is generated from README.Rmd. Please edit that file -->

# augmentCongress

A package to extract named members of the U.S. Congress in messy text
with typos and inconsistent name formats

Install this package with

    devtools::install_github("judgelord/augmentCongress")

    library(augmentCongress)

Currently, functions are in two scripts:

``` r
# name matching function 
source(here::here("R", "nameMethods.R"))

# common typos and known permutations and nicknames 
source(here::here("R", "MemberNameTypos.R"))
```

## Data

This package relies on a data frame of permutations of the names of
members of congress. This dataframe is build on the basic structure of
voteview.com data, especially the `bioname` field.

``` r
data("members")

members[c("chamber", "congress", "bioname", "pattern")] |> head()
```

    #> # A tibble: 6 × 4
    #>   chamber   congress bioname                        pattern                                                                                                                                             
    #>   <chr>        <int> <chr>                          <chr>                                                                                                                                               
    #> 1 President      108 BUSH, George Walker            "george bush|george walker bush|\\bg bush|george w bush|\\bna bush|(^|senator |representative )bush\\b|bush, george|bush george|bush, g\\b|presiden…
    #> 2 House          108 DEAL, John Nathan              "john deal|john nathan deal|\\bj deal|john n deal|nathan deal|nathan nathan deal|nathan n deal|\\bn deal|(^|senator |representative )deal\\b|deal, …
    #> 3 Senate         108 CAMPBELL, Ben Nighthorse       "ben campbell|ben nighthorse campbell|\\bb campbell|ben n campbell|benjamin campbell|benjamin nighthorse campbell|benjamin n campbell|(^|senator |r…
    #> 4 House          108 HALL, Ralph Moody              "ralph hall|ralph moody hall|\\br hall|ralph m hall|\\bna hall|(^|senator |representative )hall\\b|hall, ralph|hall ralph|hall, r\\b|representative…
    #> 5 House          108 TAUZIN, Wilbert Joseph (Billy) "wilbert tauzin|wilbert joseph tauzin|\\bw tauzin|wilbert j tauzin|billy tauzin|billy joseph tauzin|billy j tauzin|\\bb tauzin|(^|senator |represen…
    #> 6 Senate         108 SHELBY, Richard C.             "richard shelby|richard c shelby|\\br shelby|rich shelby|rich c shelby|(^|senator |representative )shelby\\b|shelby, rich|shelby, richard|shelby ri…

## Basic Usage

There are two basic functions

1.  `extractMemberName()` returns a list of the names of members of
    Congress in a supplied vector of text
2.  `augmentCongress()` augments a dataframe that includes at least one
    unique identifier to include a suite of other common identifiers

For example, we can use `extractMemberName()` to detect the names of
members of congress in the text of the congressional record. I
demonstrate this with text of the congressional record from 3/1/2007,
scraped and parsed using methods described
[here](https://github.com/judgelord/cr)

``` r
data("cr2007_03_01")

head(cr2007_03_01)
```

    #> # A tibble: 6 × 9
    #>   file                             date       url_txt                                                                                speaker    header             section   url          year  chamber 
    #>   <chr>                            <date>     <chr>                                                                                  <chr>      <chr>              <chr>     <chr>        <chr> <chr>   
    #> 1 CREC-2007-03-01-pt1-PgE431-2.htm 2007-03-01 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-PgE431-2.htm HON. SAM … RECOGNIZING JARRE… extensio… https://www… 2007  Extensi…
    #> 2 CREC-2007-03-01-pt1-PgE431-3.htm 2007-03-01 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-PgE431-3.htm HON. MARK… INTRODUCING A CON… extensio… https://www… 2007  Extensi…
    #> 3 CREC-2007-03-01-pt1-PgE431-4.htm 2007-03-01 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-PgE431-4.htm HON. JAME… BIOSURVEILLANCE E… extensio… https://www… 2007  Extensi…
    #> 4 CREC-2007-03-01-pt1-PgE431-5.htm 2007-03-01 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-PgE431-5.htm HON. JIM … A TRIBUTE TO THE … extensio… https://www… 2007  Extensi…
    #> 5 CREC-2007-03-01-pt1-PgE431.htm   2007-03-01 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-PgE431.htm   HON. SAM … RECOGNIZING JARRE… extensio… https://www… 2007  Extensi…
    #> 6 CREC-2007-03-01-pt1-PgE432-2.htm 2007-03-01 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-PgE432-2.htm HON. SANF… IN HONOR OF SYNOV… extensio… https://www… 2007  Extensi…

``` r
# to better match member names, this method currently requires a column, "congress" with the number of the congress (this can be created from a date)
# in this example, all observations are in the 110th congress 

cr2007_03_01$congress <- 110

# extract legislator names and match to voteview ICPSR numbers
cr <- extractMemberName(data = cr2007_03_01, 
                        col_name = "speaker", 
                        members = members)
```

    #> Typos fixed in 5 seconds

    #> Searching  data for members of the 110th, n = 171 (125 distinct strings).

    #> Names matched in 7 seconds

    #> Joining, by = c("congress", "pattern", "first_name", "last_name")

``` r
head(cr)
```

    #> # A tibble: 6 × 19
    #>   data_row_id match_id icpsr bioname                    string   pattern         chamber congress file    date       url_txt     speaker   header      section url      year  first_name last_name state
    #>   <chr>       <chr>    <dbl> <chr>                      <chr>    <chr>           <chr>      <dbl> <chr>   <date>     <chr>       <chr>     <chr>       <chr>   <chr>    <chr> <chr>      <chr>     <chr>
    #> 1 000002      000001   20124 GRAVES, Samuel             extensi… "samuel graves… House        110 CREC-2… 2007-03-01 https://ww… HON. SAM… RECOGNIZIN… extens… https:/… 2007  Samuel     GRAVES    miss…
    #> 2 000003      000002   29906 UDALL, Mark                extensi… "mark udall|\\… House        110 CREC-2… 2007-03-01 https://ww… HON. MAR… INTRODUCIN… extens… https:/… 2007  Mark       UDALL     colo…
    #> 3 000004      000003   20136 LANGEVIN, James            extensi… "james langevi… House        110 CREC-2… 2007-03-01 https://ww… HON. JAM… BIOSURVEIL… extens… https:/… 2007  James      LANGEVIN  rhod…
    #> 4 000005      000004   20501 COSTA, Jim                 extensi… "jim costa|\\b… House        110 CREC-2… 2007-03-01 https://ww… HON. JIM… A TRIBUTE … extens… https:/… 2007  Jim        COSTA     cali…
    #> 5 000006      000005   20124 GRAVES, Samuel             extensi… "samuel graves… House        110 CREC-2… 2007-03-01 https://ww… HON. SAM… RECOGNIZIN… extens… https:/… 2007  Samuel     GRAVES    miss…
    #> 6 000007      000006   29339 BISHOP, Sanford Dixon, Jr. extensi… "sanford bisho… House        110 CREC-2… 2007-03-01 https://ww… HON. SAN… IN HONOR O… extens… https:/… 2007  Sanford    BISHOP    geor…

Because `augmentCongress` links each detected name to ICPSR IDs from
voteview.com, we already have some information, like party
(`party_name`), state, district, and ideology scores (`nominate.dim1`)
for each match in the text of the congressional record.

``` r
full_join(cr, members) |> select(data_row_id, match_id, bioname, icpsr, congress, state, district_code, party_name, nominate.dim1, url_txt)
```

    #> Joining, by = c("icpsr", "bioname", "pattern", "chamber", "congress", "first_name", "last_name", "state")

    #> # A tibble: 6,684 × 10
    #>    data_row_id match_id bioname                    icpsr congress state        district_code party_name       nominate.dim1 url_txt                                                                     
    #>    <chr>       <chr>    <chr>                      <dbl>    <dbl> <chr>                <int> <chr>                    <dbl> <chr>                                                                       
    #>  1 000002      000001   GRAVES, Samuel             20124      110 missouri                 6 Republican Party         0.442 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-P…
    #>  2 000003      000002   UDALL, Mark                29906      110 colorado                 2 Democratic Party        -0.353 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-P…
    #>  3 000004      000003   LANGEVIN, James            20136      110 rhode island             2 Democratic Party        -0.375 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-P…
    #>  4 000005      000004   COSTA, Jim                 20501      110 california              20 Democratic Party        -0.191 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-P…
    #>  5 000006      000005   GRAVES, Samuel             20124      110 missouri                 6 Republican Party         0.442 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-P…
    #>  6 000007      000006   BISHOP, Sanford Dixon, Jr. 29339      110 georgia                  2 Democratic Party        -0.282 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-P…
    #>  7 000008      000007   TOWNS, Edolphus            15072      110 new york                10 Democratic Party        -0.519 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-P…
    #>  8 000009      000008   DAVIS, Thomas M., III      29576      110 virginia                11 Republican Party         0.282 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-P…
    #>  9 000010      000009   GRAVES, Samuel             20124      110 missouri                 6 Republican Party         0.442 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-P…
    #> 10 000011      000010   UDALL, Mark                29906      110 colorado                 2 Democratic Party        -0.353 https://www.congress.gov/117/crec/2007/03/01/modified/CREC-2007-03-01-pt1-P…
    #> # … with 6,674 more rows

The function `cr_augmentCongress` is not not yet built, but it will
simply join in other datasets on IDs like ICPSR numbers.

    # add other common unique identifiers
    cr_augmented <- augmentCongress(cr)
