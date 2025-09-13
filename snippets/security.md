## Security

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

## Final Thoughts

Good security around sensitive data doesn't have to be difficult, and now one is equipped with practical and easy means to prevent key leakage. Utilizing `shelter::unlockKeys` or `redcapAPI::unlockREDCap` has an additional advantage that it works well with automated environments. A sys admin may utilize overriding the interactive bits of `shelter` by providing system level ENV variables or a local yaml key file. Thus with a simple call one is now enhancing their security *and* preparing for automated reporting. 

If you have automation needs have your sys admin email on of the authors of the `shelter` package. Details will be provided.

### References

* [1]: Toulas B (2023). "Over 12 million auth secrets and keys leaked on GitHub in 2023." https://www.bleepingcomputer.com/news/security/
over-12-million-auth-secrets-and-keys-leaked-on-github-in-2023/
* [2]: (2024). Federal Register, (2024-17446), 64815\u201364832.
* [3]: [CRAN: shelter](https://cran.r-project.org/package=shelter)

