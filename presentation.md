R and Python package for REDCap API (beginner level)
======

REDCapCon 2025

Shawn Garbett, Günther Rezniczek, Geneva Marshall, Thomas Wilson, Will Beasley

Follow along with us at
<https://github.com/OuhscBbmc/redcapcon-2025-r-api>

Agenda
------------

Total time: **70m**

1. Introductions 1m
1. Motivation (Günther) 5m
   1. vs Excel/csv
   1. vs writing your own R code with httr/RCurl
   1. Reproducibility!
1. Prereqs  (Günther)
    1. REDCap project exists
    1. user has access to
       1. project,
       1. codebook,
       1. api key
    1. dev machine has R & redcapAPI/REDCapR
1. Basic Introduction redcapAPI/REDCapR (sg) 10m (unstructured)
1. Security (sg) 5m (shelter)
1. redcapAPI (sg) -- Type Theory (type casting) REDCapR/redcapAPI
1. Longitudinal (Will)
   1. Roll your own with redcapAPI/REDCapR
   1. REDCapTideR
1. Writing (brief -Will)
1. Other packages (brief -Will)
1. REDCap API 2.0 (Günther)
   a. (Günther) 10m, Issues with API
   a. Mention Python and Ruby tools.
1. Closing
   1. Benjamin Nutter slide
   1. Q&A


Motivation
------------

{Insert from <snippets/motivation.md>}

Prereqs
------------

{Insert from <snippets/prereqs.md>}

Basic Introduction redcapAPI/REDCapR
------------

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


Security
------------

When using the API as we've seen one needs an API key to get access. The API key is a user name and password in one convenient package. All the recommendations around dealing with a password apply to it's treatment and usage. This presents a problem, as writing a password into a file to get the needed access creates a high security risk.

In 2023 it is estimated that 12 million API keys were leaked on GitHub, a public code sharing website.[Toulas 2023][1]. Private health information (PHI) in the United States carries a minimum \$141 per record leaked with a cap of $2,134,831 per incident as of August 2024[Federal Register][2]. The possibility of leakage from an API key in code is massive and should never be done. Leaking a key is a reportable security incident if PHI is involved and can have serious consequences.

**BAD**: Writing an API_KEY in your R code.

**NOT GOOD**: Putting your API_KEY in a file inside the project directory. Just one slip away from sharing, and if you're publishing your project to github that share is the world.

**BETTER**: Put your API_KEYs in a file outside the project directory. 

## Security Method 1: Token File (REDCapR)

The basic goals are (a) separate the secret values from the R file into a dedicated file and (b) secure the dedicated file. If using a git repository, prevent the file from being committed with an entry in `.gitignore`. Ask your institution’s IT security team for their recommendation.

The `retrieve_credential_local()` function in the `REDCapR` package loads relevant information from a csv into R. The plain-text file might look like this:

```csv
redcap_uri,username,project_id,token,comment
"https://redcap-dev-2.ouhsc.edu/redcap/api/","myusername","33","9A068C425B1341D69E83064A2D273A70","simple"
"https://redcap-dev-2.ouhsc.edu/redcap/api/","myusername","34","DA6F2BB23146BD5A7EA3408C1A44A556","longitudinal"
"https://redcap-dev-2.ouhsc.edu/redcap/api/","myusername","36","F9CBFFF78C3D78F641BAE9623F6B7E6A","simple-write"
```

To retrieve the credentials for the first project listed above, pass the value of “33” to `project_id`.

```r
path_credential <- system.file("../tokenstore/dev-2.credentials", package = "REDCapR")
credential  <- REDCapR::retrieve_credential_local(
  path_credential = path_credential,
  project_id      = 33
)

credential
#> $redcap_uri
#> [1] "https://redcap-dev-2.ouhsc.edu/redcap/api/"
#> 
#> $username
#> [1] "myusername"
#> 
#> $project_id
#> [1] 33
#> 
#> $token
#> [1] "9A068C425B1341D69E83064A2D273A70"
#> 
#> $comment
#> [1] "simple"
```

## Security Method 2: Encrypted Keyring (shelter)

A plain text file with ones password on a laptop is a risk if that laptop is stolen or hacked. Storing the tokens in an encrypted manner is even better security.

**GREAT**: Store tokens using encryption.

Working with cryptography and security is in general more difficult than just writing the API_KEY into a file. The knowledge required, and coding, much less knowledge of best practices can be challenging. What's needed is a solution that does the following:

* Follows best security practices possible.
* Usage is easier than self management of a key.
* Works in automated environments with _no code changes_.
* Works for Windows, Mac and Linux.
* Works inside RStudio and command line.

This is a tall bill of goods to accomplish, but it's available in the R package [`shelter`][3]. It took three years of user feedback and design to hit these goals, and this is now available with a simple interface for any API_KEY desired. It utilizes an encrypted key ring to store API keys to disk using encryption. Thus API keys only exist in memory unencrypted and go away when a session is closed (*assuming R isn't saving session to disk which defeats the point of encryption*!!!).

## shelter and redcapAPI 

Let's first look at an example using `redcapAPI`. This has a helper function (calling `shelter`) that hides a part we'll explore later. This would be near the top of an R function to open the connection. 

```r
uri <- "https://redcap-dev-2.ouhsc.edu/redcap/api/"
redcapAPI::unlockREDCap(
  c(rcc = 'REDCapCon2025'), 
    keyring='API_KEYs',
    envir=1,
    url=uri)
```

And with that small bit of code, the chance of leakage of API_KEY is greatly reduced.

The first argument `c(rcc = 'REDCapCon2025')` is a named list of connections to make. In this example it says to write to the current envir `envir=1` the variable `rcc` to hold a `redcapConnection` object. The name 'REDCapCon2025' is a user defined name that will be used in interaction. In general it's best of this is the name of the project in REDCap, but as that can change there is no constraint on the name. It is simply how the user wishes it to be known.

The second argument `keyring='API_KEYS'` says to store the requested keys in a key ring named 'API_KEYS'. One could use the same key ring on different projects, or have just one key ring for everything. This is once again up to how the user wishes to organize their key rings. We recommend that you have a key ring for each group of related REDCap projects.

The `envir` argument just allows for writing the connections directly into the current environment as variables. 

The `url` argument specifies the remote url/uri utilized for the connection.

The first time this is run, it will ask for a password for the key ring. Later executions of the command will ask for the password to unlock the key ring.

![](./images/shelter-password.png)

If a given API key is not found in the key ring, the user will be prompted for the named API key. Then it will test that key to see if it is valid.

![](./images/shelter-apikey.png)

If validation succeeds it will return that object to the environment, if it fails it will ask again. If an API key for a project changes, it will detect this as a failure when it tries to open and ask for that key again. 

## shelter and REDCapR

`REDCapR` doesn't have a utility function like `redcapAPI` for using `shelter` yet (stay tuned). This gives a great opportunity to show how one can use `shelter` with any package needing API keys. I.e., `shelter` is a security package for any secret storage retrieval needs.

What's needed is a "connect and check" function. A function that given an API key, returns either a connection object or NULL upon failure. The redcap version is the lowest overhead on the remote server, so that call is a great check to see if a an API_KEY is valid.

```r
library(shelter)
library(REDCapR)
checkToken <- function(key, ...)
{
  ver <- redcap_version(token=key, verbose=FALSE, ...)
  if(ver == '0.0.0') NULL else key
}
```

Let's see that in action:

```r
uri <- "https://redcap-dev-2.ouhsc.edu/redcap/api/"

# THIS IS EXACTLY WHAT SHOULD NEVER BE IN ONES CODE
#        vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
token <- "9A068C425B1341D69E83064A2D273A70" # Public DEMO Project
#        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
# This key is to a project provided for public demo use.

bad <- paste(rep('0',32), collapse='')

checkToken(bad, redcap_uri = uri)
#> NULL
#> 
checkToken(token, redcap_uri = uri)
#> [1] '15.2.0'
#>
checkToken(token, redcap_uri="https://google.com")
#> NULL
```

Thus one can use this function to store the token in a key ring instead of writing it in code via `shelter`.

```r
shelter::unlockKeys(
  c(token='REDCapCon2025'),
    keyring='API_KEYs',
    connectFUN=checkToken,
    envir=1,
    redcap_uri = uri) # NOTE: ... is passed to checkToken
#>
token
#> [1] "9A068C425B1341D69E83064A2D273A70"
redcap_version(token=token, verbose=FALSE, redcap_uri = uri)
#> [1] '15.2.0'
```
Thus this token is now hidden behind the name "REDCapCon2025" and sits in memory as `token`. It is stored encrypted and is decrypted using the user supplied password. The key ring is named 'API_KEYs'. 

## Important Note

Use of `shelter` alone is insufficient for good security. One must make sure to never save workspace to .Rdata on exit and never set `cache=TRUE` in quarto or rmarkdown files for a block that contains private health information (PHI) or private identifying information (PII).

In RStudio, look under Tools -> Global Options -> General and make sure save is set to NEVER and load is not clicked. This should be done for history as well.

Further making sure if one needs to store PHI/PII locally that is always stored to an encrypted volume in some form is important.

## Takeaway on Security

Good security around sensitive data doesn't have to be difficult, and now one is equipped with practical and easy means to prevent key leakage. Utilizing `shelter::unlockKeys` or `redcapAPI::unlockREDCap` has an additional advantage that it works well with automated environments creating a seemless transition to an automated report system. A sys admin may utilize overriding the interactive bits of `shelter` by providing system level ENV variables or a local yaml key file. Thus with a simple call one is now enhancing security *and* preparing for automated reporting. 

If you have automation needs have your sys admin email on of the authors of the `shelter` package. Details will be provided.

### References

* [1]: Toulas B (2023). "Over 12 million auth secrets and keys leaked on GitHub in 2023." https://www.bleepingcomputer.com/news/security/
over-12-million-auth-secrets-and-keys-leaked-on-github-in-2023/
* [2]: (2024). Federal Register, (2024-17446), 64815\u201364832.
* [3]: [CRAN: shelter](https://cran.r-project.org/package=shelter)



Type Theory (type casting)
------------

One of the major benefits of utilizing a pre-existing package like `REDCapR` or `redcapAPI` is the amount of reuseable code that has gone into it's design. In the heart of these packages something known as 'type casting' occurs and it is a central concept which contains a large amount of value added. 

Type casting is the conversion from one data type into another in a computer. All data coming from REDCap is a character based string, and R has a variety of data types utilized in analysis, e.g. numeric or factor. A naïve idea would be that when given a string "123" it going to be easy to convert into a numeric in R. For that specific string it is, but there are a huge number of concerns that lurk in those waters. We shall dive into a short aside on just how theoretically deep those waters are and a follow up of the practicalities of REDCap data.

*The following paragraph can be safely ignored.*

$f : A \rightarrow B$ denotes a function, $f$ in type theory is a function that converts something of type $A$ into something of type $B$. The value of automatically evaluating and checking this has become very important in computer science. However, the idea has become incredibly important in mathematics as well. Betrand Russell published *Principia Mathematica* in 1910 seeking to find a single foundation for all of mathematics. He settled on set theory and it had 18 axioms, small formal assumptions, which it was based on. In this system it takes about 300 pages of math to show that 1+1=2. Godel's famous incompleteness theorem in 1929 was a huge blow to the goals of finding a single system that all mathematics could be based upon--as any such system must forever remain incomplete. Type theory in computers began making theoretical advances in the 1960's, a system Russell had initially considered. Voevodsky in 2006 showed a homotopy between $\lambda$-calculus, algebraic topology and type theory which is homotopy type theory (HoTT). In this foundation only 2 axioms are required to bootstrap mathematics and a short few pages to show 1+1=2. Russell's vision from a century before has been realized. One of those axioms, is the existence of a path between two points $A$ and $B$ which is represented by $f : A \rightarrow B$, thus making type casting one of the deepest philosophical entities in mathematics! 

![](./images/type-theory.png)

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

One thing that is quite common as an analyst is locating those pesky values that do not cast properly. This is payoff from having validation as a separate step. 

```r
> reviewInvalidRecords(redcapAPI::exportRecordsTyped(conn))
#> # Failed Validations from REDCap project 'REDCapR: simple'
#> 
#> Tue 09 Sep 2025 12:00:00 AM UTC  
#> Package redcapAPI version 2.11.2  
#> REDCap version 15.2.0 
```

There's no data validations to report, all is good. The print for this renders to markdown for embedding in a report. If there had been validation failures--it would provide a direct html link to the form to edit that record. 

## Sparse Block Matrix Form

![](./images/block_sparse.png)

Another issue in play is the context in which exporting data from REDCap without restriction to a form means that every column for every form has a value. This results in a block sparse format with the majority of the data being NULL and potentially thousands of columns wide. It's best to filter and extract these down to the relevant forms. 

### redcapAPI export forms

One method of dealing with this is a wrapper function in `redcapAPI::exportBulkRecords`. This is a loop function that queries the meta data, finds all the defined forms, and then requests the data for each form with only the appropriate columns. Empty rows are dropped. This can greatly reduce server and network load while reducing time to produce analytically ready data.

```r
conn$instruments()
#>      instrument_name   instrument_label
#> 1       demographics       demographics
#> 2             health             health
#> 3 race_and_ethnicity race_and_ethnicity

exportBulkRecords(list(simple=conn))$simple_demographics
#>   record_id name_first name_last                                 address
#> 1         1     Nutmeg  Nutmouse 14 Rose Cottage St.\nKenning UK, 323232
#> 2         2     Tumtum  Nutmouse 14 Rose Cottage Blvd.\nKenning UK 34243
#> 3         3     Marcus      Wood          243 Hill St.\nGuthrie OK 73402
#> 4         4      Trudy       DAG          342 Elm\nDuncanville TX, 75116
#> 5         5   John Lee    Walker      Hotel Suite\nNew Orleans LA, 70115
#>        telephone               email        dob age    sex demographics_complete
#> 1 (405) 321-1111     nutty@mouse.com 2003-08-30  11 Female              Complete
#> 2 (405) 321-2222    tummy@mouse.comm 2003-03-10  11   Male              Complete
#> 3 (405) 321-3333        mw@mwood.net 1934-04-09  80   Male              Complete
#> 4 (405) 321-4444 peroxide@blonde.com 1952-11-02  61 Female              Complete
#> 5 (405) 321-5555  left@hippocket.com 1955-04-15  59   Male              Complete
```

Coupled with `unlockKeys`'s ability to open multiple project, `exportBulkRecords` can loop over and pull all the data in forms directly into memory making ready to analyze data organized and type cast in two carefully constructed calls!

## Checkboxes

With so many concerns handled in a consistent, repeatable manner by the libraries there still remains one difficult issue--that of the checkbox. 

The checkbox from a user experience using a form is a wonderful thing. A user can click/unclick a box and move on quickly. The root of the problem is that a checkbox *always* has a value--checked or unchecked. There is no possibility of an NA by *definition*. Just by including a checkbox on a form--it now always has a value on that form. Couple this with the issue of detecting forms via sparse block matrices and whether a checkbox is NA is now dependent on the context in which it exists and it is not uncommon to define it as NA when all the other fields in the same form are NA or unchecked if a checkbox. This presents contextual interpretation of a checkbox's value which is not a desirable thing.

Checkboxes are best avoided and a dropdown that has a yes/no in which a user explicitly enters a value is a preferable alternative. Thus a user has explicitly set the value and the interpretation is clear.

Be aware that checkboxes will cause difficulties in getting data clean for analysis and custom filtering code to remove empty rows based on an analysts interpretation is required.

Longitudinal
------------

{Insert from <snippets/longitudinal.md>}

Writing
------------

{Insert from <snippets/writing.md>}

Other packages
------------

{Insert from <snippets/other-packages.md>}

REDCap API 2.0
------------

{Insert from <snippets/redcap-api-2.md>}

Closing
------------

{Insert from <snippets/closing.md>}
