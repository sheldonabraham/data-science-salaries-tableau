# Calculated fields

Every calculated field in `DataSalaries.twbx`, plus the aliases and settings that go with them.

## Before Tableau

The source file has 42 exact duplicate rows. In Excel, I deleted the unnamed first column (row numbers 0–606), then used Data → Remove Duplicates. That left 565 rows, saved as `ds_salaries_clean.csv`.

## Aliases and sort order

| Field | Codes → names | Sort |
| --- | --- | --- |
| experience_level | EN → Entry, MI → Mid, SE → Senior, EX → Executive | Entry, Mid, Senior, Executive |
| employment_type | FT → Full-time, PT → Part-time, CT → Contract, FL → Freelance | |
| company_size | S → Small, M → Medium, L → Large | Small, Medium, Large |

`company_location` and `employee_residence` have the geographic role Country/Region. GB and ES were matched by hand to United Kingdom and Spain.

## Measures

```
Jobs = COUNT([job_title])

Median Salary = MEDIAN([salary_in_usd])

// Used on the "US employers" tile
US Median = MEDIAN(IF [company_location] = "US" THEN [salary_in_usd] END)

Rest Median = MEDIAN(IF [company_location] <> "US" THEN [salary_in_usd] END)

US Multiple = [US Median] / [Rest Median]

Fully Remote % = SUM(IF [remote_ratio] = 100 THEN 1 ELSE 0 END) / [Jobs]
```

## Groupings

```
// Splits every chart into two colours: blue for US, grey for everywhere else
Region = IF [company_location] = "US" THEN "US employers" ELSE "Everywhere else" END

Work Setting =
CASE [remote_ratio]
    WHEN 0 THEN "On-site"
    WHEN 50 THEN "Hybrid"
    ELSE "Fully remote"
END

// 50 job titles grouped into 5 families. Leadership words are checked first,
// so "Lead Data Analyst" counts as Leadership, not Analyst.
Role Family =
IF CONTAINS(LOWER([job_title]), "head") OR CONTAINS(LOWER([job_title]), "director")
   OR CONTAINS(LOWER([job_title]), "manager") OR CONTAINS(LOWER([job_title]), "lead")
   OR CONTAINS(LOWER([job_title]), "principal") THEN "Leadership"
ELSEIF CONTAINS(LOWER([job_title]), "analy") THEN "Analyst"
ELSEIF CONTAINS(LOWER([job_title]), "machine") OR CONTAINS(LOWER([job_title]), "ml")
   OR CONTAINS(LOWER([job_title]), "vision") OR CONTAINS(LOWER([job_title]), "nlp")
   OR CONTAINS(LOWER([job_title]), "ai ") THEN "ML / AI"
ELSEIF CONTAINS(LOWER([job_title]), "engineer") OR CONTAINS(LOWER([job_title]), "architect")
   OR CONTAINS(LOWER([job_title]), "etl") OR CONTAINS(LOWER([job_title]), "developer") THEN "Data Engineering"
ELSE "Data Science"
END
```

## Number formats

| Field | Format |
| --- | --- |
| Median Salary, US Median, Rest Median | Currency, 0 decimals, thousands (K) |
| US Multiple | Number, 1 decimal, suffix "x" |
| Fully Remote % | Percentage, 0 decimals |

## Map filter

Pay Map has `Jobs` on Filters set to "at least 5", so a country with one or two jobs can't show an extreme median.
