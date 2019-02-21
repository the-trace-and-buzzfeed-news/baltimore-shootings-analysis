# Analysis of Baltimore Shootings

This repository includes methodologies, data, and code supporting the following articles, published by The Trace and BuzzFeed News:

- "Shoot Someone In A Major US City, And Odds Are You’ll Get Away With It" (January 24, 2019) — [The Trace](https://www.thetrace.org/features/murder-solve-rate-gun-violence-baltimore-shootings) / [BuzzFeed News](https://www.buzzfeednews.com/article/sarahryley/police-unsolved-shootings)
- "5 Things To Know About Cities’ Failure To Arrest Shooters" (January 24, 2019) — [The Trace](https://www.thetrace.org/2019/01/gun-murder-solve-rate-understaffed-police-data-analysis) / [BuzzFeed News](https://www.buzzfeednews.com/article/sarahryley/5-things-to-know-about-cities-failure-to-arrest-shooters)

[*Click here for additional data and code from The Trace and BuzzFeed News's collaboration.*](https://github.com/the-trace-and-buzzfeed-news/introduction)

## Data sources

The data used in these analysis came from three sources:

- Baltimore PD's incident database
- Baltimore PD's police-district-boundary data
- Google Maps' Geocoding API

### Incident database

In response to a public records request submitted by Sarah Ryley of The Trace, the Baltimore Police Department provided a series of spreadsheets detailing the victims and suspects for the following:

- All homicides between 2012 and mid-2017
- All non-fatal shootings between 2010 and mid-2017

The records were provided on August 2, 2017, and contained the following columns:

| Column         | Description
|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DATABASE`     | Either "Non Fatal Shooting" or "Homicide". |
| `CASE_NUMBER`  | The unique identifier assigned to the incident, e.g., "17V0351". |
| `CC_NUMBER`    | The "criminal complaint" number associated with the case. Not present for suspect files. |
| `CASE_DATE`    | The date associated with the case, "2017-07-26". |
| `CASE_ADDRESS` | The address associated with the case, e.g, "1700 SHERWOOD AV". Not present for suspect files. |
| `LAST_NAME`    | The person's last name. |
| `FIRST_NAME`   | The person's first name. |
| `DESCRIPTION`  | Either "Victim", "Suspect", or "Person of Interest". |
| `VICT_TYPE`    | Either "Non Fatal Shooting" or "Homicide {x}", where {x} is "Shooting", "Stab/Cut", "Blunt Force", "Beating", "Asphyxiation", "Arson", "Auto", "Poison", or "Other". Not present  |for suspect files.
| `RACE`         | The person's race. |
| `SEX`          | The person's gender. |
| `DOB`          | The person's date of birth. \* |
| `CASE_STATUS`  | Either "Open," "Closed," or "Open/Warrant". |

\* The earliest date of birth in the database is from 1950, despite many people in the database being verifiably older than that. Conversely, about 0.9% of victims have DOBs in 2018 or later — impossible, given that no incident in the data occurred after 2017. What appears to have happened is that dates of birth before 1950 were pushed forward by 100 years, so that (for example) 1949 became 2049.

### Police district shapefiles

The Baltimore Police Department publishes geospatial data, in the form of "shapefiles", [detailing the boundaries of each of its nine police districts](http://gis-baltimore.opendata.arcgis.com/datasets/police-districts). The Trace and BuzzFeed News used these boundaries to determine the district in which the incidents above occurred.


### Google Maps geocoding

The incident data did not include geocoordinates (latitude/longitude) for the incidents. It did, however, include street addresses for each incident. The Trace and BuzzFeed News submitted those addresses to [Google Maps' Geocoding API](https://developers.google.com/maps/documentation/geocoding/start) to obtain geocoordinates. The file `inputs/baltimore-case-addresses.csv` contains the results of that geocoding. For each incident, the file contains the following columns:


| Column                          | Description
|---------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `agency_ori`                    | Always "MDBPD0000" |
| `agency_incident_id`            | The unique identifier assigned to the incident, e.g., "17H0198" |
| `location_address`              | The address provided in Baltimore PD's data, e.g., "1200 GREENMOUNT AV" |
| `location_address_adjusted`     | The address submitted to the Geocoding API, e.g., "1200 GREENMOUNT AV, Baltimore, Maryland, USA" |
| `geocoded_location_interpreted` | The Geocoding API's interpretation of the address, e.g., "1200 Greenmount Ave, Baltimore, MD 21202, USA" |
| `geocoded_location_type`        | The Geocoding API's determination of what type of address this is, e.g., "street_address", "premise", "route". If the API returned multiple values, they are  |concatenated with the "\|" character.
| `geocoding_imprecise`           | A True/False indicator of whether the API returned a precise result (rather than, e.g., the centroid of the city). |
| `lat`                           | The latitude returned by the Geocoding API |
| `lng`                           | The longitude returned by the Geocoding API |
| `tract_2010`                    | The Census tract ID corresponding to the latitude/longitude in the 2010 Census |
| `tract_state_2010`              | The state in which that Census tract resides |
| `tract_2000`                    | The Census tract ID corresponding to the latitude/longitude in the 2000 Census |
| `tract_state_2000`              | The state in which that Census tract resides |


## Data standardization and anonymization

To aid the analysis, The Trace and BuzzFeed News standardized raw data, taking the following steps:

- Selected only cases that occurred in 2012 or later, so that the analysis operates on the same timeframe for both fatal and non-fatal shootings
- Excluded non-shooting homicides
- Fixed one incorrect case number
- Identified names that were vague/non-specific (e.g., "SUSPECT", "UNKNOWN", etc.).
- Combined the victim and suspect spreadsheets into a single "persons" table
- Constructed a `person_uid` based on the person's listed name and date of birth
- Manually adjusted the `person_uid`s for people whose listed names appear to involve typos

The file in `inputs/baltimore-shootings-persons.csv` contains the results of that data-standardization, with the following addition steps taken for the purposes of anonymization:

- Removes the `FIRST_NAME` and `LAST_NAME` columns
- Replaces the `DOB` column with an `age_at_case` column
- Replaces the `person_uid` values with a cryptographic hash of the values (first 10 characters of SHA-256 with salt)

## Data Analysis

The `notebooks/analyze-baltimore-shootings.ipynb` notebook, written in Python, takes the data output in the previous step, and does the following:

- Calculates basic case trends and victim demographics
- Identifies groups of shootings connected connected by a common suspect or victim
- Calculates the number of people listed as a victim in one case and a suspect in another, and similar statistics
- Identifies, whether, for persons shot in multiple incidents, their final recorded victimization was fatal or non-fatal
- Counts the number of people listed as suspects in multiple cases, and the number of victims for those cases
- Calculates the demographic groupings of mulitply-involved persons
- Maps and spatially analyzes cases connected by a common victim/suspect

## Data Disclaimer

The data in this repository is based on raw data that the Baltimore Police Department provided to The Trace and BuzzFeed News in response to a public records request. We have carefully checked the accuracy of our analysis, and shared our findings with the Baltimore Police Department and numerous experts in the law enforcement field prior to publication. We are sharing our data, methodology, and code in order to support further research and reporting on gun violence. However, users of this data may wish to independently verify the accuracy of their findings prior to making them public, as The Trace and Buzzfeed make no representations or warranties as to any third party use of this data.

## Licensing

All code in this repository is available under the [MIT License](https://opensource.org/licenses/MIT). All the incident and address data files are available under the [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0) license. The police district shapefile data is [published by the City of Baltimore](http://gis-baltimore.opendata.arcgis.com/datasets/police-districts) with "no license specified".

## Questions / Comments?

Please contact Jeremy Singer-Vine at jeremy.singer-vine@buzzfeed.com.
