Data for the REDCapCon Presentation  2025
=============

"https://redcap-dev-2.ouhsc.edu/redcap/api/","myusername","64","800AB9D9D6D4C4F397EC1E2C8B9F3542","multilevel-model-1"

<https://github.com/OuhscBbmc/REDCapR/blob/main/inst/misc/dev-2.credentials>


```r
library(lme4)


lmer(
  cog_1 ~ 1 + phys_1 + phys_2 + phys_3 + county_id + (1 | subject_id),
  data = ds
) |>
  summary()
```
