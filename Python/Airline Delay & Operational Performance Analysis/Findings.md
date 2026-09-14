# Phase 2 — Dataset Understanding

## 1. Dataset Overview

| Finding             | Result                                                      |
| ------------------- | ----------------------------------------------------------- |
| Rows                | 3,000,000                                                   |
| Columns             | 32                                                          |
| Date range          | 2019-01-01 to 2023-08-31                                    |
| Unique flight dates | 1,704                                                       |
| Date coverage       | Continuous; every calendar date in the range is represented |
| Duplicate rows      | 0                                                           |
| Memory usage        | ~732 MB                                                     |

---

## 2. Dataset Grain

**One row represents one scheduled flight record.**

The dataset contains information about the flight's:

`date → airline → flight → origin → destination → scheduled/actual operation → delays → cancellation/diversion`

---

## 3. Column Groups

### Flight / Airline Identification

| Column         | Understanding                                        |
| -------------- | ---------------------------------------------------- |
| `AIRLINE`      | Airline name                                         |
| `AIRLINE_DOT`  | Airline name combined with DOT/IATA identifier       |
| `AIRLINE_CODE` | Airline code, e.g. `UA`, `DL`, `NK`                  |
| `DOT_CODE`     | U.S. Department of Transportation carrier identifier |
| `FL_NUMBER`    | Flight number                                        |

`AIRLINE_DOT` is not a location.

---

### Route

| Column        | Understanding            |
| ------------- | ------------------------ |
| `ORIGIN`      | Origin airport code      |
| `ORIGIN_CITY` | Origin airport city      |
| `DEST`        | Destination airport code |
| `DEST_CITY`   | Destination airport city |
| `DISTANCE`    | Flight distance          |

---

### Date

| Column    | Understanding |
| --------- | ------------- |
| `FL_DATE` | Flight date   |

`FL_DATE` was converted from `object` to Pandas datetime.

---

### Scheduled Clock Times

| Column         | Understanding            |
| -------------- | ------------------------ |
| `CRS_DEP_TIME` | Scheduled departure time |
| `CRS_ARR_TIME` | Scheduled arrival time   |

Times are stored numerically in an `HHMM`-style representation, not as timestamps.

---

### Actual Clock Times

| Column       | Understanding               |
| ------------ | --------------------------- |
| `DEP_TIME`   | Actual departure time       |
| `WHEELS_OFF` | Time aircraft leaves runway |
| `WHEELS_ON`  | Time aircraft lands         |
| `ARR_TIME`   | Actual arrival time         |

These are also stored numerically rather than as timestamp/datetime values.

---

### Durations

| Column             | Understanding                                     |
| ------------------ | ------------------------------------------------- |
| `DEP_DELAY`        | Difference between actual and scheduled departure |
| `TAXI_OUT`         | Gate departure → wheels off                       |
| `TAXI_IN`          | Wheels on → gate arrival                          |
| `ARR_DELAY`        | Difference between actual and scheduled arrival   |
| `CRS_ELAPSED_TIME` | Scheduled flight duration                         |
| `ELAPSED_TIME`     | Actual elapsed flight duration                    |
| `AIR_TIME`         | Time spent airborne                               |

Durations are generally measured in minutes.

---

## 4. Negative Delays

Negative delay values are valid.

Example:

`DEP_DELAY = -6`

means the flight departed **6 minutes earlier than scheduled**.

The same concept applies to `ARR_DELAY`.

---

## 5. Operational Status

| Field               | Finding                       |
| ------------------- | ----------------------------- |
| `CANCELLED`         | Binary cancellation indicator |
| `DIVERTED`          | Binary diversion indicator    |
| `CANCELLATION_CODE` | Cancellation reason category  |

Observed status combinations:

| Cancelled | Diverted |   Flights |
| --------: | -------: | --------: |
|         0 |        0 | 2,913,804 |
|         0 |        1 |     7,056 |
|         1 |        0 |    79,140 |
|         1 |        1 |         0 |

Cancellation and diversion are mutually exclusive in this dataset.

---

## 6. Cancellation Findings

Total cancelled flights:

**79,140**

`CANCELLATION_CODE` has four observed categories:

`A, B, C, D`

Counts:

| Code |   Flights |
| ---- | --------: |
| A    |    19,476 |
| B    |    28,772 |
| C    |     6,475 |
| D    |    24,417 |
| NaN  | 2,920,860 |

The number of non-null cancellation codes exactly equals the number of cancelled flights.

**Interpretation:** `CANCELLATION_CODE` appears to be structurally applicable only to cancelled flights.

---

## 7. Cancelled Flight Findings

For cancelled flights:

| Field       | Observation                                     |
| ----------- | ----------------------------------------------- |
| `DEP_TIME`  | Usually missing, but 1,525 records have a value |
| `ARR_TIME`  | All missing                                     |
| `ARR_DELAY` | All missing                                     |

We observed cancelled flights with a departure time, but the reason for this should **not be assumed** without further evidence.

---

## 8. Diverted Flight Findings

Total diverted flights:

**7,056**

For diverted flights:

| Field          | Finding         |
| -------------- | --------------- |
| `DEP_TIME`     | 7,056 populated |
| `ARR_TIME`     | 6,256 populated |
| `DEP_DELAY`    | 7,056 populated |
| `ARR_DELAY`    | 0 populated     |
| `ELAPSED_TIME` | 0 populated     |
| `AIR_TIME`     | 0 populated     |

A diverted flight can have an `ARR_TIME`, but does not have a normal `ARR_DELAY` in this dataset.

---

## 9. Missing-Value Findings

Important missing-value counts:

| Column                   |   Missing |
| ------------------------ | --------: |
| `DEP_TIME`               |    77,615 |
| `DEP_DELAY`              |    77,644 |
| `TAXI_OUT`               |    78,806 |
| `WHEELS_OFF`             |    78,806 |
| `WHEELS_ON`              |    79,944 |
| `TAXI_IN`                |    79,944 |
| `ARR_TIME`               |    79,942 |
| `ARR_DELAY`              |    86,198 |
| `ELAPSED_TIME`           |    86,198 |
| `AIR_TIME`               |    86,198 |
| `CRS_ELAPSED_TIME`       |        14 |
| `CANCELLATION_CODE`      | 2,920,860 |
| Each `DELAY_DUE_*` field | 2,466,137 |

Important finding:

**Missing values are not automatically data-quality problems.**

Several missing-value patterns correspond to operational status or non-applicable fields.

---

## 10. Delay-Cause Findings

Five delay-cause fields:

* `DELAY_DUE_CARRIER`
* `DELAY_DUE_WEATHER`
* `DELAY_DUE_NAS`
* `DELAY_DUE_SECURITY`
* `DELAY_DUE_LATE_AIRCRAFT`

Findings:

| Category                         |   Flights |
| -------------------------------- | --------: |
| At least one delay-cause value   |   533,863 |
| All five delay-cause fields null | 2,466,137 |

For all **533,863** records with delay-cause information:

* `CANCELLED = 0`
* `DIVERTED = 0`
* `ARR_DELAY` is populated
* Minimum `ARR_DELAY` = 15 minutes

Therefore, these fields appear to apply to a specific subset of delayed, non-cancelled, non-diverted flights rather than representing randomly missing data.

We have **not yet established** whether the five cause columns sum directly to `ARR_DELAY`.

---

## 11. Cardinality Findings

| Column              | Unique values |
| ------------------- | ------------: |
| `AIRLINE`           |            18 |
| `AIRLINE_DOT`       |            18 |
| `AIRLINE_CODE`      |            18 |
| `DOT_CODE`          |            18 |
| `ORIGIN`            |           380 |
| `ORIGIN_CITY`       |           373 |
| `DEST`              |           380 |
| `DEST_CITY`         |           373 |
| `CANCELLED`         |             2 |
| `DIVERTED`          |             2 |
| `CANCELLATION_CODE` |             4 |
| `DEP_TIME`          |         1,440 |
| `WHEELS_OFF`        |         1,440 |
| `WHEELS_ON`         |         1,440 |
| `ARR_TIME`          |         1,440 |

The `1,440` unique actual-time values support the interpretation that these fields represent **minute-level time-of-day values**.

---

## 12. Duplicate Findings

```text
df.duplicated().sum() = 0
```

No exact duplicate rows were found.

---

## 13. Current Data-Quality Conclusions

At the end of Phase 2:

### Confirmed / understood

* Dataset grain
* Date coverage
* Airline identifiers
* Airport/route fields
* Scheduled vs actual times
* Clock times vs durations
* Negative delays
* Cancelled vs diverted status
* Major missing-value patterns
* Structural missingness in several fields
* No exact duplicate rows

### Still to investigate in Phase 3

* Validity of numeric `HHMM` time encoding
* `2400` and other boundary values
* Whether any impossible time values exist
* The 14 missing `CRS_ELAPSED_TIME` values
* Whether any other invalid/outlier values require treatment
* Relationship between delay-cause totals and `ARR_DELAY`

This is now our **baseline Phase 2 record**. We can use it as the reference point while moving into Phase 3.
