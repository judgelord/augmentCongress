# augmentCongress
A package to extract named members of the U.S. Congress in messy text with typos and inconsistent name formats

Install this package with 
```
devtools::install_github("judgelord/augmentCongress")
```

## Basic Usage

There are two basic functions
1. `extractMemberName()` returns a list of the names of members of Congress in a supplied vector of text
2. `augmentCongress()` augments a dataframe that includes at least one unique identifier to include a suite of other common identifiers

```
library(augmentCongress)

data("congressional_record")

# extract legislator names and match to voteview ICPSR numbers
cr <- extractMemberName(congressional_record, col_name = "speaker")

cr

# add other common unique identifiers
cr_augmented <- augmentCongress(cr)

cr_augmented
```
