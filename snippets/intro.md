Simple Introduction
===========

One has a choice of `REDCapR` or `redcapAPI` and both offer a rich set of features. The `REDCapR` package has better `tibble` integration with the more modern `tidyverse` style of operations. The `redcapAPI` package is targeted towards base R and includes additional support for automated environments. This introduction will cover examples using both libraries. 

We begin with loading libraries and a token and a uri. NOTE: writing a token in R code is poor security practice. More on proper security in the next section.

```{r}
uri      <- "https://redcap-dev-2.ouhsc.edu/redcap/api/"
token    <- "9A068C425B1341D69E83064A2D273A70" 
```

## Read Data (Unstructured)

### REDCapR

```{r}
rcr_1 <- REDCapR::redcap_read(redcap_uri=uri, token=token)
#> 24 variable metadata records were read from REDCap in 0.3 seconds.  The http status code was 200.
#> The data dictionary describing 17 fields was read from REDCap in 0.2 seconds.  The http status code was 200.
#> 3 instrument metadata records were read from REDCap in 0.1 seconds.  The http status code was 200.
#> 1 rows were read from REDCap in 0.1 seconds.  The http status code was 200.
#> 2 data access groups were read from REDCap in 0.1 seconds.  The http status code was 200.
#> 5 records and 1 columns were read from REDCap in 0.5 seconds.  The http status code was 200.
#> Starting to read 5 records  at 2025-09-09 18:08:59.403296.
#> Reading batch 1 of 1, with subjects 1 through 5 (ie, 5 unique subject records).
#> 5 records and 25 columns were read from REDCap in 0.3 seconds.  The http status code was 200.
#>
rcr_1 <- rcr_1$data # Extract just the data
```

At this point, the data.frame `rcr_1` has everything one needs to start analyzing the project.

```{r}
rcr_1
#> # A tibble: 5 � 25
#>   record_id name_first name_last address  telephone email dob          age   sex
#>       <dbl> <chr>      <chr>     <chr>    <chr>     <chr> <date>     <dbl> <dbl>
#> 1         1 Nutmeg     Nutmouse  "14 Ros\u2026 (405) 32\u2026 nutt\u2026 2003-08-30    11     0
#> 2         2 Tumtum     Nutmouse  "14 Ros\u2026 (405) 32\u2026 tumm\u2026 2003-03-10    11     1
#> 3         3 Marcus     Wood      "243 Hi\u2026 (405) 32\u2026 mw@m\u2026 1934-04-09    80     1
#> 4         4 Trudy      DAG       "342 El\u2026 (405) 32\u2026 pero\u2026 1952-11-02    61     0
#> 5         5 John Lee   Walker    "Hotel \u2026 (405) 32\u2026 left\u2026 1955-04-15    59     1
#> # \u2139 16 more variables: demographics_complete <dbl>, height <dbl>, weight <dbl>,
#> #   bmi <dbl>, comments <chr>, mugshot <chr>, health_complete <dbl>,
#> #   race___1 <dbl>, race___2 <dbl>, race___3 <dbl>, race___4 <dbl>,
#> #   race___5 <dbl>, race___6 <dbl>, ethnicity <dbl>, interpreter_needed <dbl>,
#> #   race_and_ethnicity_complete <dbl>

hist(rcr_1$weight)
```
![](./images/histogram-weight.png)

```{r}
summary(rcr_1)
#>    record_id  name_first         name_last           address         
#>  Min.   :1   Length:5           Length:5           Length:5          
#>  1st Qu.:2   Class :character   Class :character   Class :character  
#>  Median :3   Mode  :character   Mode  :character   Mode  :character  
#>  Mean   :3                                                           
#>  3rd Qu.:4                                                           
#>  Max.   :5                                                           
#>                                                                      
#>   telephone            email                dob                  age      
#>  Length:5           Length:5           Min.   :1934-04-09   Min.   :11.0  
#>  Class :character   Class :character   1st Qu.:1952-11-02   1st Qu.:11.0  
#>  Mode  :character   Mode  :character   Median :1955-04-15   Median :59.0  
#>                                        Mean   :1969-11-06   Mean   :44.4  
#>                                        3rd Qu.:2003-03-10   3rd Qu.:61.0  
#>                                        Max.   :2003-08-30   Max.   :80.0  
#>                                                                           
#>       sex      demographics_complete     height          weight   
#>  Min.   :0.0   Min.   :2             Min.   :  6.0   Min.   :  1  
#>  1st Qu.:0.0   1st Qu.:2             1st Qu.:  7.0   1st Qu.:  1  
#>  Median :1.0   Median :2             Median :165.0   Median : 54  
#>  Mean   :0.6   Mean   :2             Mean   :110.2   Mean   : 48  
#>  3rd Qu.:1.0   3rd Qu.:2             3rd Qu.:180.0   3rd Qu.: 80  
#>  Max.   :1.0   Max.   :2             Max.   :193.0   Max.   :104  
#>                                                                   
#>       bmi          comments           mugshot          health_complete
#>  Min.   : 19.8   Length:5           Length:5           Min.   :0      
#>  1st Qu.: 24.7   Class :character   Class :character   1st Qu.:0      
#>  Median : 27.9   Mode  :character   Mode  :character   Median :1      
#>  Mean   :110.9                                         Mean   :1      
#>  3rd Qu.:204.1                                         3rd Qu.:2      
#>  Max.   :277.8                                         Max.   :2      
#>                                                                       
#>     race___1      race___2      race___3      race___4      race___5  
#>  Min.   :0.0   Min.   :0.0   Min.   :0.0   Min.   :0.0   Min.   :0.0  
#>  1st Qu.:0.0   1st Qu.:0.0   1st Qu.:0.0   1st Qu.:0.0   1st Qu.:1.0  
#>  Median :0.0   Median :0.0   Median :0.0   Median :0.0   Median :1.0  
#>  Mean   :0.2   Mean   :0.2   Mean   :0.2   Mean   :0.2   Mean   :0.8  
#>  3rd Qu.:0.0   3rd Qu.:0.0   3rd Qu.:0.0   3rd Qu.:0.0   3rd Qu.:1.0  
#>  Max.   :1.0   Max.   :1.0   Max.   :1.0   Max.   :1.0   Max.   :1.0  
#>                                                                       
#>     race___6     ethnicity interpreter_needed race_and_ethnicity_complete
#>  Min.   :0.0   Min.   :0   Min.   :0.00       Min.   :0.0                
#>  1st Qu.:0.0   1st Qu.:1   1st Qu.:0.00       1st Qu.:2.0                
#>  Median :0.0   Median :1   Median :0.00       Median :2.0                
#>  Mean   :0.2   Mean   :1   Mean   :0.25       Mean   :1.6                
#>  3rd Qu.:0.0   3rd Qu.:1   3rd Qu.:0.25       3rd Qu.:2.0                
#>  Max.   :1.0   Max.   :2   Max.   :1.00       Max.   :2.0                
#>                            NA's   :1

summary(lm(age ~ 1 + sex + bmi, data = rcr_1))
#> 
#> Call:
#> lm(formula = age ~ 1 + sex + bmi, data = rcr_1)
#> 
#> Residuals:
#>       1       2       3       4       5 
#>  -2.491   1.954   9.132   2.491 -11.086 
#> 
#> Coefficients:
#>             Estimate Std. Error t value Pr(>|t|)  
#> (Intercept) 63.34496    8.89980   7.118   0.0192 *
#> sex         13.55626    9.62958   1.408   0.2945  
#> bmi         -0.24426    0.04337  -5.632   0.0301 *
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 10.55 on 2 degrees of freedom
#> Multiple R-squared:  0.9442, Adjusted R-squared:  0.8884 
#> F-statistic: 16.92 on 2 and 2 DF,  p-value: 0.05581
```

### redcapAPI

`redcapAPI` has a different approach in that a connection object is created. This object maintains state locally for things like the data dictionary to minimize round trips to the server.

```{r}
conn <- redcapAPI::redcapConnection(uri, token)
rca_1 <- exportRecordsTyped(conn)
```

```{r}
head(rca_1)
#>    record_id name_first name_last                                 address      telephone               email
#>  1         1     Nutmeg  Nutmouse 14 Rose Cottage St.\nKenning UK, 323232 (405) 321-1111     nutty@mouse.com
#>  2         2     Tumtum  Nutmouse 14 Rose Cottage Blvd.\nKenning UK 34243 (405) 321-2222    tummy@mouse.comm
#>  3         3     Marcus      Wood          243 Hill St.\nGuthrie OK 73402 (405) 321-3333        mw@mwood.net
#>  4         4      Trudy       DAG          342 Elm\nDuncanville TX, 75116 (405) 321-4444 peroxide@blonde.com
#>  5         5   John Lee    Walker      Hotel Suite\nNew Orleans LA, 70115 (405) 321-5555  left@hippocket.com

hist(rca_1$weight)
```

![](./images/histogram-weight.png)

```{r}
summary(rca_1)
> summary(rca_1)
#>  record_id          name_first         name_last           address         
#> Length:5           Length:5           Length:5           Length:5          
#> Class :character   Class :character   Class :character   Class :character  
#> Mode  :character   Mode  :character   Mode  :character   Mode  :character  
#>                                                                            
#>                                                                            
#>                                                                            
#>  telephone            email                dob                     
#> Length:5           Length:5           Min.   :1934-04-09 00:00:00  
#> Class :character   Class :character   1st Qu.:1952-11-02 00:00:00  
#> Mode  :character   Mode  :character   Median :1955-04-15 00:00:00  
#>                                       Mean   :1969-11-05 23:48:00  
#>                                       3rd Qu.:2003-03-10 00:00:00  
#>                                       Max.   :2003-08-30 00:00:00  
#>     age                sex    demographics_complete     height          weight   
#> Length:5           Female:2   Incomplete:0          Min.   :  6.0   Min.   :  1  
#> Class :character   Male  :3   Unverified:0          1st Qu.:  7.0   1st Qu.:  1  
#> Mode  :character              Complete  :5          Median :165.0   Median : 54  
#>                                                     Mean   :110.2   Mean   : 48  
#>                                                     3rd Qu.:180.0   3rd Qu.: 80  
#>                                                     Max.   :193.0   Max.   :104  
#>      bmi          comments           mugshot            health_complete
#> Min.   : 19.8   Length:5           Length:5           Incomplete:2     
#> 1st Qu.: 24.7   Class :character   Class :character   Unverified:1     
#> Median : 27.9   Mode  :character   Mode  :character   Complete  :2     
#> Mean   :110.9                                                          
#> 3rd Qu.:204.1                                                          
#> Max.   :277.8                                                          
#>      race___1      race___2      race___3      race___4      race___5
#> Unchecked:4   Unchecked:4   Unchecked:4   Unchecked:4   Unchecked:1  
#> Checked  :1   Checked  :1   Checked  :1   Checked  :1   Checked  :4  
#>                                                                      
#>                                                                      
#>                                                                      
#>                                                                      
#>      race___6                  ethnicity interpreter_needed
#> Unchecked:4   Unknown / Not Reported:1   Mode :logical     
#> Checked  :1   NOT Hispanic or Latino:3   FALSE:3           
#>               Hispanic or Latino    :1   TRUE :1           
#>                                          NA's :1           
#>                                                            
#>                                                            
#> race_and_ethnicity_complete
#> Incomplete:1               
#> Unverified:0               
#> Complete  :4 

summary(lm(age ~ 1 + sex + bmi, data = rca_1))
#>
#> Call:
#> lm(formula = age ~ 1 + sex + bmi, data = rca_1)
#>
#> Residuals:
#>       1       2       3       4       5 
#>  -2.491   1.954   9.132   2.491 -11.086 
#> attr(,"label")
#> [1] "Age (years)"
#> 
#> Coefficients:
#>             Estimate Std. Error t value Pr(>|t|)  
#> (Intercept) 63.34496    8.89980   7.118   0.0192 *
#> sexMale     13.55626    9.62958   1.408   0.2945  
#> bmi         -0.24426    0.04337  -5.632   0.0301 *
#> ---
#> Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
#> 
#> Residual standard error: 10.55 on 2 degrees of freedom
#> Multiple R-squared:  0.9442,	Adjusted R-squared:  0.8884 
#> F-statistic: 16.92 on 2 and 2 DF,  p-value: 0.05581
```

Note at this point there are already differences in the data, but not the fitted model parameters. The variable for sex in the REDCapR is presented as the numeric code, whereas the redcapAPI version converted it to a factor utilizing the defined metadata. How these choices of data type conversion are made and options for specifying them are covered later.

```{r}
summary(rcr_1$sex) # REDCapR data in tibble
#>    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
#>     0.0     0.0     1.0     0.6     1.0     1.0 
```

```{r}
summary(rca_1$sex) # redcapAPI data in base R
#> Female   Male 
#>      2      3 
```

Both libraries have advantages and disadvantages when compared. The biggest consideration at this point is a user writing code to analyze data must chose a path as the downstream code working on the data is not compatible and cannot be easily swapped out. What support does one have with their department and what code are is shared for analysis is a strong consideration.  

*Pause for questions.*

## Be Nice to Your Server

When you read a dataset for the first time, you probably haven’t decided which columns are needed so it makes sense to retrieve everything. As you gain familiarity with the data and with the analytic objectives, consider being more selective with the variables and rows transported from the remote server to your local machine.

Advantages include:

1. A server is almost always more efficient filtering than a language like R or Python.
1. REDCap’s PHP code retrieves less data from REDCap’s database and translates less to a text format (like csv or json).
1. Fewer bytes are transmitted across your network.
1. Your local machine will have better performance, because R has a smaller dataset to manage.
1. Your brain doesn’t have to look past unnecessary columns.
1. Your R code doesn’t have filter what the server already removed.
1. Highly-sensitive PHI columns that are unnecessary for an analysis remain on the server.

### Specify Rows and Columns

As a basic demo of both libraries capacity to limit a query, let's make our query only pull records 1 and 4, and only the fields (columns): 'record_id', 'name_first', and 'age'. 

```r
records <- c(1, 4)
fields  <- c("record_id", "name_first", "age")
REDCapR::redcap_read(
  redcap_uri  = uri,
  token       = token,
  fields      = fields,
  records     = records,
  verbose     = FALSE
)$data
#> # A tibble: 2 × 3
#>   record_id name_first   age
#>       <dbl> <chr>      <dbl>
#> 1         1 Nutmeg        11
#> 2         4 Trudy         61
```

```r
exportRecordsTyped(
  conn,
  fields  = fields,
  records = records)
#>  record_id name_first age
#> 1         1     Nutmeg  11
#> 2         4      Trudy  61
```

#### Row Filtering

REDCapR offers a useful filter command. It limit rows via an expression before returning. See your server's documentation for the syntax rules of the filter statements. Remember to enclose your variable names in square brackets. Also be aware of differences between strings and numbers.

```r
REDCapR::redcap_read(
  redcap_uri    = uri,
  token         = token,
  filter_logic  = "'2003-01-01' < [dob]",
  verbose       = FALSE,
  fields        = fields
)$data
#> # A tibble: 2 × 3
#>   record_id name_first   age
#>       <dbl> <chr>      <dbl>
#> 1         1 Nutmeg        11
#> 2         2 Tumtum        11
```

Similar limitations of a query exist for `events` and `forms`. 
