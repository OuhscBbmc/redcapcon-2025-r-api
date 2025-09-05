## Security

When using the API as we've seen one needs an API key to get access. The API key is a user name and password in one convenient package. All the recommendations around dealing with a password apply to it's usage. This presents a problem, as writing a password into a file to get the needed access creates a high security risk.

In 2023 it is estimated that 12 million API keys were leaked on GitHub, a public code sharing website.[Toulas 2023][1]. Private health information (PHI) in the United States carries a minimum \$141 per record leaked with a cap of $2,134,831 per incident as of August 2024[Federal Register][2]. The possibility of leakage from an API key in code is massive and should never be done. Leaking a key (e.g. via stolen laptop) is a reportable security incident if PHI is involved and can have serious consequences.

It is insufficient to say what not to do, a positive solution is required. That solution should have the following properties:

* Follows best security practices possible.
* Usage is easier than self management of a key.
* Works in automated environments with _no code changes_.
* Works for Windows, Mac and Linux.
* Works inside RStudio and command line.

This is a tall bill of goods to accomplish, but it's available in the R package [`shelter`][3]. It took three years of user feedback and design to hit these goals, but they are now available with a simple interface for any API_KEY desired. It utilizes an encrypted key ring to store API keys to disk using encryption. Thus API keys only exist in memory unencrypted and go away when a session is closed (*assuming R isn't saving session to disk*!!!)

## shelter and redcapAPI 

Let's first look at an example using `redcapAPI`. This has a helper function (calling `shelter`) that hides a part we'll explore later. This would be near the top of an R function to open the connection. 

```r
library(redcapAPI)
uri <- "https://redcap-dev-2.ouhsc.edu/redcap/api/"
unlockREDCap(c(rcc = 'REDCapCon2025'), 
             keyring='API_KEYS',
             envir=1,
             url=uri)
```

And with that small bit of code, the chance of leakage of API_KEY is greatly reduced.

The first argument `c(rcc = 'REDCapCon2025')` is a named list of connections to make. In this example it says to write to the current envir `envir=1` the variable `rcc` to hold a `redcapConnection` object. The name 'REDCapCon2025' is a user defined name that will be used in interaction. In general it's best of this is the name of the project in REDCap, but as that can change there is no constraint on the name. It is simply how the user wishes it to be known.

The second argument `keyring='API_KEYS'` says to store the requested keys in a key ring named 'API_KEYS'. One could use the same key ring on different projects, or have just one key ring for everything. This is once again up to how the user wishes to organize their key rings. We recommend that you have a key ring for each group of related REDCap projects.

The `envir` argument just allows for writing the connections directly into the current environment as variables. 

The `url` argument specifies the remote url utilized for the connection.

The first time this is run, it will ask for a password for the key ring. Later executions of the command will ask for the password to unlock the key ring. If a given API key is not found in the key ring, the user will be prompted for the named API key. This will attempt to open an connection and read the version of the REDCap server back. If it succeeds it will return that object to the environment, if it fails it will ask again. If an API key for a project changes, it will detect this as a failure when it tries to open and ask for that key again. 

## shelter and REDCapR

`REDCapR` doesn't have a utility function like `redcapAPI` for using `shelter` yet (stay tuned). This gives a great opportunity to show how one can use `shelter` with any package needing API keys. 

What's needed is a "connect and check" function. A function that given an API key, returns either a connection object or NULL upon failure. 

```r
library(shelter)
library(REDCapR)
checkToken <- function(key, ...)
{
  ver <- redcap_version(token=key, verbose=FALSE, ...)
  if(ver == '0.0.0') NULL else ver
}
```

Let's see how that works:

```r
uri <- "https://redcap-dev-2.ouhsc.edu/redcap/api/"

# THIS IS EXACTLY WHAT SHOULD NEVER BE IN ONES CODE
#        vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
token <- "9A068C425B1341D69E83064A2D273A70" # Public DEMO Project
#        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
# This key is to a project provided for public demo use.

bad <- paste(rep('0',32)

checkToken(bad, collapse=''), redcap_uri = uri)

checkToken(token, redcap_uri = uri)

checkToken(token, redcap_uri="https://google.com")

```

Thus one can use this function to store the token in a key ring instead of writing it in code via `shelter`.

```r
unlockKeys(c(token='OUHSC_DEV2'),
           keyring='API_KEYs',
           connectFUN=checkToken,
           envir=1,
           redcap_uri = uri) # Passed to checkToken
           
redcap_version(token=token, verbose=FALSE, redcap_uri = uri)
```

Thus this token is now hidden behind the name "OUHSC_DEV2" and sits in memory as `token`.

## Important Note

Use of `shelter` alone is insufficient for good security. One must make sure to never save workspace to .Rdata on exit and never set `cache=TRUE` in quarto or rmarkdown files for a block that contains private health information (PHI) or private identifying information (PII).

In RStudio, look under Tools -> Global Options -> General and make sure save is set to NEVER and load is not clicked. This should be done for history as well.

Further making sure if one needs to store PHI/PII locally that is always stored to an encrypted volume in some form is important.

## Final Thoughts

Good security around sensitive data doesn't have to be difficult, and now one is equipped with practical means to prevent key leakage. 

If you have automation needs have your sys admin email on of the authors of the `shelter` package. Details will be provided. These instructions are hidden from developers as there are overrides that should only be used in a controlled production environment.

### References

* [1]: Toulas B (2023). "Over 12 million auth secrets and keys leaked on GitHub in 2023." https://www.bleepingcomputer.com/news/security/
over-12-million-auth-secrets-and-keys-leaked-on-github-in-2023/
* [2]: (2024). Federal Register, (2024-17446), 64815\u201364832.
* [3]: [CRAN: shelter](https://cran.r-project.org/package=shelter)

