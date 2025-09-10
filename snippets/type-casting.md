## Type Casting

One of the major benefits of utilizing a pre-existing package like `REDCapR` or `redcapAPI` is the amount of reuseable code that has gone into it's design. In the heart of these packages something known as 'type casting' occurs and it is a central concept which contains a large amount of value added. 

Type casting is the conversion from one data type into another in a computer. All data coming from REDCap is a character based string, and R has a variety of data types utilized in analysis, e.g. numeric or factor. A naïve idea would be that when given a string "123" it going to be easy to convert into a numeric in R. For that specific string it is, but there are a huge number of concerns that lurk in those waters. We shall dive into a short aside on just how theoretically deep those waters are and a follow up of the practicalities of REDCap data.

*The following paragraph can be safely ignored.*

$f : A \rightarrow B$ denotes a function, $f$ in type theory is a function that converts something of type $A$ into something of type $B$. The value of automatically evaluating and checking this has become very important in computer science. However, the idea has become incredibly important in mathematics as well. Betrand Russell published *Principia Mathematica* in 1910 seeking to find a single foundation for all of mathematics. He settled on set theory and it had 18 axioms, small formal assumptions, which it was based on. In this system it takes about 300 pages of math to show that 1+1=2. Godel's famous incompleteness theorem in 1929 was a huge blow to the goals of finding a single system that all mathematics could be based upon--as any such system must forever remain incomplete. Type theory in computers began making theoretical advances in the 1960's, a system Russell had initially considered. Voevodsky in 2006 showed a homotopy between $\lambda$-calculus, algebraic topology and type theory which is homotopy type theory (HoTT). In this foundation only 2 axioms are required to bootstrap mathematics and a short few pages to show 1+1=2. Russell's vision from a century before has been realized. One of those axioms, is the existence of a path between two points $A$ and $B$ which is represented by $f : A \rightarrow B$, thus making type casting one of the deepest philosophical entities in mathematics! 

Fortunately, a REDCap user's needs are simpler--and the API developers have you covered. However, it is not as simple as it sounds. All data stored in REDCap is character or string. In the computer, an analyst wants dates, numbers, factors or potentially some other data type the authors of `REDCapR` and `redcapAPI` have never heard of. To further complicate matters data can be missing under a variety of definitions and it might be invalid as user input may have been changed from free form to date specified in the middle of data collection leading to a huge number of uninterpretable date values.

Type casting from REDCap concerns itself with 3 fundamental steps:

1. Is a value NA?
2. Is a value valid given it's specification in the REDCap data dictionary?
3. For values that are not NA and are valid, convert in memory to the computer's related type. 

Secondly, the inverse applies when writing data back via the API. Given a value in memory, type cast it down to a string that can be stored properly in REDCap. 

Both libraries both make choices for the user that are the usual for 95% of the cases, but that doesn't mean those choices are what ones project needs. Thus inversion of control is provided such that the user can override these operations with their own.

### REDCapR and readr Approach

As the automation of your scripts matures and institutional resources depend on its output, its output should be stable. One way to make it more predictable is to specify the column names and the column data types. In the previous example, notice that R (specifically `readr::read_csv()`) made its best guess and reported it in the "Column specification" section.

In the following example, REDCapR passes col_types to [readr::read_csv()](https://readr.tidyverse.org/reference/read_delim.html) as it converts the plain-text output returned from REDCap into an R data frame. (To be precise, a tibble is returned.)

When readr sees a column with values like 1, 2, 3, and 4, it will make the reasonable guess that the column should be a double precision floating-point data type. However we recommend using the [simplest data type reasonable](https://ouhscbbmc.github.io/data-science-practices-1/coding.html#coding-simplify-types) because a simpler data type is less likely contain unintended values and it’s typically faster, consumes less memory, and translates more cleanly across platforms. As specifically for identifiers like record_id specify either an integer or character.

```r
# Specify the column types.
desired_fields <- c("record_id", "race")
col_types <- readr::cols(
  record_id  = readr::col_integer(),
  race___1   = readr::col_logical(),
  race___2   = readr::col_logical(),
  race___3   = readr::col_logical(),
  race___4   = readr::col_logical(),
  race___5   = readr::col_logical(),
  race___6   = readr::col_logical()
)
REDCapR::redcap_read(
  redcap_uri  = uri,
  token       = token,
  fields      = desired_fields,
  verbose     = FALSE,
  col_types   = col_types
)$data
#> # A tibble: 5 x 7
#>   record_id race___1 race___2 race___3 race___4 race___5 race___6
#>       <int> <lgl>    <lgl>    <lgl>    <lgl>    <lgl>    <lgl>   
#> 1         1 FALSE    FALSE    FALSE    FALSE    TRUE     FALSE   
#> 2         2 FALSE    FALSE    TRUE     FALSE    TRUE     FALSE   
#> 3         3 FALSE    FALSE    FALSE    TRUE     TRUE     FALSE   
#> 4         4 FALSE    TRUE     FALSE    FALSE    TRUE     FALSE   
#> 5         5 TRUE     FALSE    FALSE    FALSE    FALSE    TRUE
```

#### REDCapR Specify Everything is a Character

REDCap internally stores every value as a string. To accept full responsibility of the data types, tell readr::cols() to keep them as strings.

```r
# Specify the column types.
desired_fields <- c("record_id", "race")
col_types <- readr::cols(.default = readr::col_character())
REDCapR::redcap_read(
  redcap_uri  = credential$redcap_uri,
  token       = credential$token,
  fields      = desired_fields,
  verbose     = FALSE,
  col_types   = col_types
)$data
#> # A tibble: 5 x 7
#>   record_id race___1 race___2 race___3 race___4 race___5 race___6
#>   <chr>     <chr>    <chr>    <chr>    <chr>    <chr>    <chr>   
#> 1 1         0        0        0        0        1        0       
#> 2 2         0        0        1        0        1        0       
#> 3 3         0        0        0        1        1        0       
#> 4 4         0        1        0        0        1        0       
#> 5 5         1        0        0        0        0        1
```

### redcapAPI and specifying conversion

The package `redcapAPI` provides 3 sets of callback lists: `na`, `validation` and `cast` to split the process into much finer granular pieces. This allows far deeper control over the conversion steps by the user, but comes at a cost of greater complexity. The default conversion steps are the most widely used as observed by the authors. 

Let's dive in with an example of the normal output. 

```r
redcapAPI::exportRecordsTyped(
  conn,
  fields = desired_fields
)
#>   record_id  race___1  race___2  race___3  race___4  race___5  race___6
#> 1         1 Unchecked Unchecked Unchecked Unchecked   Checked Unchecked
#> 2         2 Unchecked Unchecked   Checked Unchecked   Checked Unchecked
#> 3         3 Unchecked Unchecked Unchecked   Checked   Checked Unchecked
#> 4         4 Unchecked   Checked Unchecked Unchecked   Checked Unchecked
#> 5         5   Checked Unchecked Unchecked Unchecked Unchecked   Checked
```

The default choice the library made was to make the checkboxes an R factor and label them as 'Checked' or 'Unchecked'. Let's let the user make a choice and specify logical. 

```r
redcapAPI::exportRecordsTyped(
  conn,
  fields = desired_fields,
  cast   = list(checkbox=castLogical)
)
#>   record_id race___1 race___2 race___3 race___4 race___5 race___6
#> 1         1    FALSE    FALSE    FALSE    FALSE     TRUE    FALSE
#> 2         2    FALSE    FALSE     TRUE    FALSE     TRUE    FALSE
#> 3         3    FALSE    FALSE    FALSE     TRUE     TRUE    FALSE
#> 4         4    FALSE     TRUE    FALSE    FALSE     TRUE    FALSE
#> 5         5     TRUE    FALSE    FALSE    FALSE    FALSE     TRUE
```

The viewpoint from `readr` was column (field) based, but here it's field description based. One can write custom functions that switch based on name if desired. It is possible to do the same functionality as `readr` but requires more coding.

### redcapAPI everything character

If one desired the absolute raw character values stored, with `redcapAPI` there is an additional need to override validations--as these values will get dropped by the engine.

```r
redcapAPI::exportRecordsTyped(
  conn,
  fields     = desired_fields,
  validation = skip_validation,
  cast       = raw_cast
)
#>   record_id race___1 race___2 race___3 race___4 race___5 race___6
#> 1         1        0        0        0        0        1        0
#> 2         2        0        0        1        0        1        0
#> 3         3        0        0        0        1        1        0
#> 4         4        0        1        0        0        1        0
#> 5         5        1        0        0        0        0        1
```

One thing that is quite common as an analyst is locating those pesky values that do not cast properly. This is part of the fruit of having validation as a separate step. 

```r
> attr(redcapAPI::exportRecordsTyped(conn), "invalid")
#> # Failed Validations from REDCap project 'REDCapR: simple'
#> 
#> Tue 09 Sep 2025 12:00:00 AM UTC  
#> Package redcapAPI version 2.11.2  
#> REDCap version 15.2.0 
```

There's no data validations to report, all is good. The print for this is context sensitive and will render to markdown for embedding in a report. If there had been validation failures--it would provide a direct html link to the form to edit that record. 

## Sparse Block Matrix Form

![](./images/block_sparse.png)

## Checkboxes

With so many concerns handled in a consistent, repeatable manner by the libraries there still remains one difficult issue--that of the checkbox. 

The checkbox from a user experience using a form is a wonderful thing. A user can click/unclick a box and move on quickly. The root of the problem is that a checkbox *always* has a value--checked or unchecked. There is no idea of an NA. Just by including a checkbox on a form--it now by definition always has a value on that form. Couple this with the issue of detecting forms via sparse block matrices and whether a checkbox is NA is now dependent on the context in which it exists and it is not uncommon to define it as NA when all the other fields in the same form are NA or unchecked if a checkbox. This presents contextual interpretation of a checkbox's value which is not a desirable thing.

Checkboxes are best avoided and a dropdown that has a yes/no in which a user explicitly enters a value is prefered. Thus a user has explicitly set the value and the interpretation is clear.