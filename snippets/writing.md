writing
================

### More error prone than reading

Mentality: start small & simple

Test the water:

- Start w/ a single patient & single cell
- Slowly add more rows & columns

And intermediate snippet might be:

```r
ds_patient |>
  dplyr::select(              # First three columns
    id_code,
    date,
    is_mobile,
  ) |>
  dplyr::slice(1:2) |>        # First two rows
  REDCapR::redcap_write(
    ds_to_write = _,
    redcap_uri  = credential$redcap_uri,
    token       = credential$token,
    convert_logical_to_integer = TRUE
  )
```

Strategy:

- Upload the patient-level instrument(s)
- Upload the each repeating instrument separately.


### Gotchas

- Recode Variables where Necessary
- Plumbing Variables
-
### Resources for Writing/Importing

- Language Agnostic Troubleshooter: <https://ouhscbbmc.github.io/REDCapR/articles/TroubleshootingApiCalls.html#writing>

- REDCapR Writing Vignette:
<https://ouhscbbmc.github.io/REDCapR/articles/workflow-write.html>