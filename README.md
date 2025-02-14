
<!-- README.md is generated from README.Rmd. Please edit that file -->

# congress: A Tool for CongressData <img src="figures/congress.png" height="150" align="right"/>

`congress` is a package designed to allow a user with only basic
knowledge of R interact with **CongressData**, a dataset with nearly 800
variables that compiles information about all US congressional districts
across 1789-2021. Specifically, the dataset tracks characteristics of
congressional districts, the members of congress themselves, and the
behavior of those members in policymaking. Users can find variables
related to demographics, politics, and policy; subset the data across
multiple dimensions; create custom aggregations of the dataset; and
access citations in both plain text and BibTeX for every variable. An
associated web application is available
[here](https://cspp.ippsr.msu.edu/congress/) and the data-only package
is [here](https://github.com/IPPSR/congressData).

## Read the Codebook and Manual

The CongressData codebook is available
<a href="https://raw.githubusercontent.com/IPPSR/congressData/main/congress_codebook_1_1.pdf" target="_blank">here</a>.

The package’s manual contains information regarding each function and
its arguments. It is available here:
<a href="https://raw.githubusercontent.com/IPPSR/congress/main/congress_1.1_manual.pdf" target="_blank">congress
manual</a>.

## Installing this Package and the Data-only Companion Package

`congress` is a functional package that interacts with CongressData. We
maintain the dataset in another package called `congressData`. You can
use the data-only package if you simply want to access the data. Install
them from GitHub like so:

``` r

# use the devtools library to download the package from GitHub
library(devtools)

# this will download congressData as well (NOTE: installation can take several minutes)
install_github("ippsr/congress")

# if there are issues or you only want to download congressData (NOTE: installation can take several minutes)
install_github("ippsr/congressData")
```

## Finding Variables

`get_var_info`: Retrieve information regarding variables in CongressData
and identify variables of interest with `get_var_info`. The function
allows you to search to codebook to find the years each variable is
observed in the data; a short and long description of each variable; and
the source and citation/s for each variable. Citations are available in
both bibtex and plain text. Use the function to search for broad terms
like ‘tax’ with the `related_to` argument and/or partial-match variable
names with `var_names`.

``` r

suppressMessages(library(dplyr))
library(congress)
#> Please cite:
#> Grossmann, M., Lucas, C., McCrain, J, & Ostrander, I. (2022). CongressData.
#> East Lansing, MI: Institute for Public Policy and Social Research (IPPSR).
#> 
#> You are using the version of the Congress Data stored in your local copy of congressData. Running `congressData::get_congress_version()` will print your local version number.

# variables related to health insurance
h_ins_cong <- get_var_info(related_to = "health insurance")

cat("There are",nrow(h_ins_cong),"variables related to health insurance in CongressData")
#> There are 41 variables related to health insurance in CongressData

head(h_ins_cong$variable)
#> [1] "percent_under18_healthins" "percent_private_under18"  
#> [3] "percent_public_under18"    "percent_privpub_under18"  
#> [5] "percent_pop18_34"          "percent_private_18_34"

# variables with 'under18' in their name
under18_cong <- get_var_info(var_names = "under18")

head(under18_cong$variable)
#> [1] "percent_under18"           "percent_under18_healthins"
#> [3] "percent_private_under18"   "percent_public_under18"   
#> [5] "percent_privpub_under18"   "under18"
```

`get_var_info` returns the following information to simplify using
CongressData:

-   variable: Variable name
-   year: The precise years the variable is observed
-   short_desc: A short description of the variable
-   long_desc: A long description of the variable
-   source: The sources of the data
-   category: the variable’s category (not all are coded)
-   plaintext_cite (1-4): Plain text citations for the data
-   bibtext_cite (1-4): BibTeX citation for the data

## Accessing Member-Year Data

`get_cong_data`: Access all or a part of CongressData with
`get_cong_data`. Subset by state names with `state` and years with
`years` (either a single year or a two-year vector that represents the
min/max of what you want). You can also use the `related_to` argument to
search across variable names, short/long descriptions from the codebook,
and citations for non-exact matches of a supplied term. For example,
searching ‘tax’ will return variables with words like ‘taxes’ and
‘taxable’ in any of those columns.

``` r

# return the entire dataset
all_the_dat <- get_cong_data()

# subset by state, topic, and years
cong_subset <- get_cong_data(states = c("Kentucky","Michigan","Pennsylvania")
                             ,related_to = "tax"
                             ,years = c(1960,1980)
                             )
```

## Aggregate to Member-Session Data with Custom Schemes

`aggregate_cong_dat`: Choose how to aggregate the member-year data into
member-session data across subsets (e.g. data sources) of CongressData.
You can choose either `Mean` or `Sum` or `First` (meaning the value in
the first year of the session) to aggregate the following chunks of the
dataset:

-   census_nonperc_vars: Non percent Census Variables
-   census_perc_vars Percent Census Variables
-   bill_vars: Congressional Bills Project Variables
-   com_vars: Committee Assignment Variables
-   else_vars: All the Other Variables

The variable names in the resulting dataset will reflect how they were
aggregated (e.g. `nbills_major_topic_10` becomes
`nbills_major_topic_10_mean`). Note: while character variables will
reflect the chosen aggregation scheme (e.g. `character_var_sum`), they
are aggregated by pasting their unique values together.

``` r

# import the data using the default mean values
agg_cong <- aggregate_cong_dat()

# choose specific aggregation methods by subgroup
cong_custom <- aggregate_cong_dat(census_nonperc_vars = "Mean"
                                  ,census_perc_vars = "Sum"
                                  ,bill_vars = "First"
                                  ,com_vars = "Mean"
                                  ,else_vars = "Sum")

# default aggs for specific states/sessions and vars related to health
cong_subset <- aggregate_cong_dat(states = c("Kentucky","Michigan","Pennsylvania")
                                  ,related_to = "health"
                                  ,sessions = c(50:55)
                                  )
```

## Pulling Citations

`get_var_info`: Each variable in CongressData was collected from
external sources, please use `get_var_info` to obtain their citations
(plain text and BibTeX). We’ve made it easy to cite the source of each
variable you use with the `get_var_info` function described above.
Supply a vector of variable names to the function with the `var_names`
function and collect the citations provided in the plain text or BibTeX
columns. NOTE: Some variables have multiple citations, so do check you
have them all.

``` r

# bibtex is also available
get_var_info(var_names = "com_benghazi_299") %>%
  pull(plaintext_cite)
#> [1] "Charles Stewart III and Jonathan Woon. Congressional Committee Assignments, 103rd to 114th Congresses, 1993--2017: House of Representatives, 2017.\n"

# bibtex is also available
get_var_info(var_names = "percent_bus") %>%
  pull(plaintext_cite)
#> [1] "U.S. Census Bureau. (2022). 2009-2019 American Community Survey 1-year Estimates. Retrieved from the Census Bureau Data API."
```

## Citation

In addition to citing each variable’s source, we ask that you cite
CongressData if use this package or the dataset. A recommended citation
is below.

> Grossmann, M., Lucas, C., McCrain, J, & Ostrander, I. (2022).
> CongressData. East Lansing, MI: Institute for Public Policy and Social
> Research (IPPSR)
