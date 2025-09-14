writing
================

### Writing to the server is more error prone than reading

Mentality: start small & simple

Test the water:

- Start w/ a single patient & a few columns (eg, `record_id` & `gender`)
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

- Client encoding & problematic non-ASCII characters

- Recode Variables where necessary

- Plumbing Variables (especially for repeating structures)

  ```r
  ds_daily <-
    ds_daily |>
    dplyr::group_by(id_code) |>
    dplyr::mutate(
      redcap_repeat_instrument  = "daily",
      redcap_repeat_instance    = dplyr::row_number(da_date),
      daily_complete            = REDCapR::constant("form_complete"),
    ) |>
    dplyr::ungroup() |>
    dplyr::select(
      id_code,                        # Or `record_id`, if you didn't rename it
      # redcap_event_name,            # If the project is longitudinal or has arms
      redcap_repeat_instrument,       # The name of the repeating instrument/form
      redcap_repeat_instance,         # The sequence of the repeating instrument
      tidyselect::everything(),       # All columns not explicitly passed to `dplyr::select()`
      daily_complete,                 # Indicates incomplete, unverified, or complete
    )
  ```

- Utilize Helper functions, eg

  ```r
  # Check for potential problems.  (Remember zero rows are good.)
  REDCapR::validate_for_write(ds_daily, convert_logical_to_integer = TRUE)
  ```

### Next Steps

- More Complexity
  - eg, arms, longitudinal events

- Batching

- Manual vs API

  If you have trouble uploading, consider adding a few fake patients & measurements and then download the csv. It might reveal something you didn’t anticipate. But be aware that it will be in the block matrix format (i.e., everything jammed into one rectangle.)

- REDCap’s CDIS & Other interoperability

  - See Stephany Duda et al's talk in 15 min: "REDCap + NIH: A Recipe for Innovation"
  - See Adam's Birds of a Feather session 3:50pm today: "REDCap SHARE (Sharing Health Access for Research & Empowerment)"
  - See Alex, Francesco, & Adam's talk 8:50am tomorrow: "EHR Integration: What's New & What's Next"

  > The Clinical Data Interoperability Services (CDIS) use FHIR to move data from your institution’s EMR/EHR (eg, Epic, Cerner) to REDCap. Research staff have control over which patient records are selected or eligible. Conceptually it’s similar to writing to REDCap’s with the API, but at much bigger scale. Realistically, it takes months to get through your institution’s human layers. Once established, a project would be populated with EMR data in much less development time –assuming the desired data models corresponds with FHIR endpoints.

### Resources for Writing/Importing

- Language Agnostic Troubleshooter: <https://ouhscbbmc.github.io/REDCapR/articles/TroubleshootingApiCalls.html#writing>

- REDCapR Writing Vignette:
  <https://ouhscbbmc.github.io/REDCapR/articles/workflow-write.html>
