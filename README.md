# CRISP-DM Phase 3: Preparing Netflix Weekly Data
In the previous phase, we examined the structure, variables, data types, and values of the Netflix dataset. We will now clean and standardize the data before analyzing it.

## Learning objectives
By the end of this tutorial, you will be able to:
- standardize column names and text values;
- convert columns to appropriate data types;
- identify and handle missing values;
- remove duplicate records;
- detect invalid numerical values;
- create useful variables; and
- save a cleaned dataset.

## Business question
Our merchandising company needs reliable and consistent data before identifying titles with commercial potential.
Incorrect data types, duplicate records, missing values, or inconsistent categories could produce misleading results.

### 1. Read the CSV file
Load the file into a pandas DataFrame.
```python
import kagglehub
import pandas as pd
# Download the latest version of the dataset
DATASET_PATH = kagglehub.dataset_download("risakashiwabara/netfllix-weekly-views-data")
print("Path to dataset files:", DATASET_PATH)
FILE = DATASET_PATH + "/2026-05-25_global_weekly.csv"
df = pd.read_csv(FILE)
df.info()
```
---
## 3. Eliminate unnecessary spaces
We have four variables containing text: "week", "category", "show_title", and "season_title".
To eliminate all the spaces at the beginning and end of the text, we use the `strip()` function:
```python
df["COLUMN_NAME"] = df["COLUMN_NAME"].str.strip()
```
Now replace repeated spaces within the text with a single space:
```python
df["COLUMN_NAME"] = df["COLUMN_NAME"].str.replace(r"\s+", " ", regex=True)
```
where `r"\s+"` is a regular expression that identifies one or more consecutive whitespace characters, `" "` is the replacement value (a single space), and `regex=True` tells pandas to interpret the first argument as a regular expression instead of literal text.

Example:
```text
"Films   (English)" → "Films (English)"
```
Repeat the same procedure with the rest of the columns containing text values.

---
## 4. Correct the data type for dates
Convert `week` to a date:
```python
df["week"] = pd.to_datetime(df["week"], errors="coerce")
```
The `errors="coerce"` argument converts invalid dates to `NaT`.

Check how many values are invalid:
```python
df["week"].isna().sum()
```
Check that the data type is now a datetime type:
```python
df.info()
```
---
## 5. Standardize text values
Capitalization differences and repeated spaces may cause equivalent values to be treated as different categories.

Convert text to lowercase:
```python
df["category"] = df["category"].str.lower()
```
Repeat the same procedure with the rest of the columns containing text values.

## 6. Remove records with missing essential values
The variable `required_columns` selects the columns in which the presence of a value is essential. Then the `dropna` function removes rows from the DataFrame that contain missing values in any of the columns listed in `required_columns`.
```python
required_columns = ["week", "category", "weekly_rank", "show_title", "season_title", "weekly_hours_viewed", "runtime", "weekly_views", "cumulative_weeks_in_top_10"]
df = df.dropna(subset=required_columns)
```
---
## 7. Remove duplicate records
First, count the number of duplicated rows:
```python
print("Duplicate records:", df.duplicated().sum())
```
`duplicated()` compares all columns and identifies rows that are exactly equal to a previous row.

Remove duplicated rows:
```python
df = df.drop_duplicates()
```
To detect and remove duplicates based only on specific columns:
```python
columns = ["show_title", "weekly_rank"]
print("Duplicate records:", df.duplicated(subset=columns).sum())
df = df.drop_duplicates(subset=columns)
```
Replace the column names with the variables you want to compare. In both cases, pandas keeps the first occurrence.

---
## 8. Validate numerical values
Apply filters on numerical columns. The following command:
```python
valid_rows = (
    df["weekly_rank"].between(1, 10)
    & (df["weekly_hours_viewed"] >= 0)
    & (df["weekly_views"] >= 0)
    & (df["runtime"] > 0)
    & (df["cumulative_weeks_in_top_10"] >= 1)
)
```
also deletes rows with NaN values. To keep the records that contain NaN values, we use the OR operator:
```python
valid_rows = (
    (df["weekly_rank"].between(1, 10) | df["weekly_rank"].isna())
    & ((df["weekly_hours_viewed"] >= 0) | df["weekly_hours_viewed"].isna())
    & ((df["weekly_views"] >= 0) | df["weekly_views"].isna())
    & ((df["runtime"] > 0) | df["runtime"].isna())
    & ((df["cumulative_weeks_in_top_10"] >= 1) | df["cumulative_weeks_in_top_10"].isna())
)
```
Depending on the variable, we can use mixed options:
```python
valid_rows = (
    df["weekly_rank"].between(1, 10)
    & (df["weekly_hours_viewed"] > 0)
    & (df["weekly_views"] > 0)
    & ((df["runtime"] >= 0) | df["weekly_views"].isna())
    & (df["cumulative_weeks_in_top_10"] >= 1)
)
```

Now we print the number of invalid records and their indexes:
```python
invalid_indices = df.index[~valid_rows]
print("Invalid record indices:", invalid_indices.tolist())
print("Number of invalid records:", len(invalid_indices))
```

Visualize the invalid records:
```python
df.loc[invalid_indices].head(1)
```

Delete invalid records from the DataFrame:
```python
df.drop(index=invalid_indices, inplace=True)
```

---
## 9. Rename text values
In column "COLUMN_NAME", replace the value "TEXT_1" with "TEXT_2":
```python
df["COLUMN_NAME"] = df["COLUMN_NAME"].replace("TEXT_1", "TEXT_2")
```
---
For example, we can replace categorical text values with category numbers:
```python
df["category"] = df["category"].replace("films (english)", 1)
df["category"] = df["category"].replace("films (non-english)", 2)
df["category"] = df["category"].replace("tv (english)", 3)
df["category"] = df["category"].replace("tv (non-english)", 4)
```
You can check the content with the command `df["category"]`, but if you run `df.info()` you will notice that the `Dtype` is `object`. To change it to `int64`, run the following code:
```python
df["category"] = pd.to_numeric(df["category"], errors="coerce").astype("Int64")
```
## 10. Inspect the cleaned dataset
```python
df.info()
```
Compare the original and cleaned datasets.

---
## 11. Save the cleaned dataset
```python
df.to_csv("netflix_weekly_clean.csv", index=False)
```
The new file will be saved in the `/content` directory.

---


## Checkpoint

1. Which columns contained missing values?
2. Were duplicate records found?
3. Why is a missing `season_title` acceptable for films?
4. Which data types were corrected?
5. How many samples remain after cleaning?

## Expected result

At the end of this activity, you should have:

- a cleaned DataFrame named `df`;
- standardized column names and text values;
- appropriate data types;
- no duplicate or invalid records;
- a file named `netflix_weekly_clean.csv`.

---

## Sources

- Kaggle, *Netflix Weekly Views Data*:  
  <https://www.kaggle.com/datasets/risakashiwabara/netfllix-weekly-views-data>
- CRISP-DM course material, Phase 3: Data Preparation.



