# Austin_crime
Analyzing the austin crime dataset by Richard Ndah

# This dataset was sourced from big query public data on google cloud 
# Below are results from analysis done on the crime dataset of the city of Austin  

### The questions I wanted to answer through my SQL queries were:
    1. What are the top five crimes recorded in the city of Austin.
    2. What are the crime counts by district.
    3. Describe the seasonal trends of the recorded crimes.
    4. What are the seasonal percentage distribution of the crimes recorded.
    5. What are the seasonal ranking of the crimes recorded.
    6. Calculate the crime peak season
    7. What are the yearly crime rate in the city of Austin

## Top five crimes 

``` sql
SELECT
  primary_type,
  COUNT(*) AS crime_count
FROM
  bigquery-public-data.austin_crime.crime
WHERE 
  primary_type IS NOT NULL
GROUP BY
 primary_type
ORDER  BY
  crime_count DESC
LIMIT  5
```
Rows |   primary_type                 | crime_count
------------------------------------------------
   1 | **Theft**                      | 14515
   2 | **Theft: All Other Larceny**   | 13539
   3 | **Theft: BOV**                 | 10545
   4 | **Burglary**                   | 10098
   5 | **Auto Theft**                 | 6231

**Summary**
 # This data snippet displays the conclusion that theft in its various 
forms is the overwhelmingly dominant type of crime reported within the city of Austin. 
The general Theft category is the single most common incident by a very large margin. 
Among more specific property crimes, burglary was reported more frequently than auto theft. 
The classification structure indicates a hierarchy within theft types.

## Crime counts by district

``` sql
SELECT
  council_district_code,
  COUNT(*) AS crime_locations 
FROM
  `bigquery-public-data.austin_crime.crime`
WHERE
  council_district_code IS NOT NULL  
GROUP BY 
  council_district_code
ORDER  BY
  crime_locations DESC
```

Rows | council_district_code | crime_locations
------------------------------------------------
   1 |                     3 |         18109
   2 |                     9 |         17662
   3 |                     4 |         16656
   4 |                     7 |         13439
   5 |                     1 |         11900
   6 |                     2 |         10397
   7 |                     5 |         10130
   8 |                     8 |          6226
   9 |                     6 |          6223
  10 |                    10 |          5255


**Summary**
 # The analysis shows that crime is not evenly distributed across council districts.
 District 3 is the highest, followed closely by 9 and 4.
 District 10 has the least crime.
 The disparity between the highest and lowest is substantial (over 3 times).

 This information can be used for resource allocation, policing strategies, and further investigation into why these disparities exist (e.g., population density, socio-economic factors, etc.).

Disparities
- The top three districts (3, 9, 4) account for 18,109 + 17,662 + 16,656 = 52,427 crimes, which is about 45.2% of the total.
  This indicates nearly half of all crimes occur in just 30% of the districts and 4 require urgent intervention

- The bottom three districts (8, 6, 10) account for 6,226 + 6,223 + 5,255 = 17,704 crimes, which is about 15.3% of the total. 

- Mid-Tier Attention: Districts like 7 (13.4k crimes) may need monitoring to prevent escalation.

Patterns
- There is a clear pattern: the top three districts (3, 9, 4) have significantly higher crime counts than the others.

- The middle group (districts 7, 1, 2, 5) have crime counts in the range of 10,000 to 13,500.

- The bottom three (8, 6, 10) are below 6,500.

## Seasonal trends of the recorded crimes

``` sql
SELECT
  primary_type,
  CASE
    WHEN EXTRACT(MONTH FROM timestamp) IN (3, 4, 5) THEN 'spring'
    WHEN EXTRACT(MONTH FROM timestamp) IN (6, 7, 8) THEN 'summer'
    WHEN EXTRACT(MONTH FROM timestamp) IN (9, 10, 11) THEN 'autumn'
    ELSE 'winter'
  END  AS  season ,
  COUNT(*) AS crime_count
FROM
  `bigquery-public-data.austin_crime.crime`
WHERE
  timestamp IS NOT NULL
GROUP BY
  primary_type, season
ORDER  BY  
  crime_count DESC;
```

**Summary**

- Theft in general is by far the most common crime, with over 14,000 incidents in summer. 

-Most theft subcategories, robbery, assault, and rape reach their highest counts during summer.

- All theft categories show a consistent seasonal pattern: highest in summer, followed by spring, then autumn, and lowest in winter. 
This makes sense as warmer weather means more people outdoors, creating more opportunities for theft.

-Burglary peaks in winter rather than summer. 
This could be because homes are more vulnerable during holidays or due to longer nights. 
And also,burglary stands out as a major crime type exhibiting the opposite seasonal pattern (Winter high, Summer low) compared to theft.

-Auto theft also shows an unusual pattern with higher counts in winter, which might relate to people warming up unattended cars.

-Violent crimes like assault and robbery also follow the summer peak pattern, though less dramatically than theft. 

-Murder counts are interesting - while "Murder" entries show summer peaks as well.

-The rarest crimes like purse snatching and coin machine theft have very low counts (just 1-3 per season).

## Seasonal percentage distribution of the crimes recorded.

``` sql
SELECT
    primary_type,
    CASE
      WHEN EXTRACT(MONTH FROM timestamp) IN (3, 4, 5) THEN 'spring'
      WHEN EXTRACT(MONTH FROM timestamp) IN (6, 7, 8) THEN 'summer'
      WHEN EXTRACT(MONTH FROM timestamp) IN (9, 10, 11) THEN 'autumn'
      ELSE 'winter'
    END AS season,
    COUNT(*) AS crime_count
  FROM
    `bigquery-public-data.austin_crime.crime`
  WHERE
    timestamp IS NOT NULL
  GROUP BY
    primary_type, season
)

SELECT
  season,
  primary_type,
  crime_count,
  ROUND(crime_count * 100.0 / SUM(crime_count) OVER (PARTITION BY season), 2) AS seasonal_percentage
FROM seasonal_trends
ORDER BY 
  season,
  crime_count DESC;
```

| Crime Category          | Summer | Winter | Key Trend              |
|-------------------------|--------|--------|------------------------|
| **Theft (Total)**       | 47.4%  | 46.1%  | Peak in summer         |
| **Burglary**            | 7.6%   | 9.6%   | Peak in winter         |
| **Auto Theft**          | 5.1%   | 5.7%   | Peak in winter         |
| **Violent Crimes**      | ~7.9%  | ~7.3%  | Higher in warm months  |
| **Homicides**           | 0.08%  | 0.08%  | Minimal seasonal swing |

**Summary**
- Theft-related crimes are the most prevalent, consistently accounting for over 46% of all crimes in every season. 
The summer months show the highest overall theft rates (47.43%), while winter sees a slight dip (46.05%).

- We can also note that the top 3 crime types (Theft, Theft: All Other Larceny, Theft: BOV) together make up about 67-68% of all crimes in every season.

- Other crime types, such as assault and robbery, show less pronounced seasonal variations but generally follow the pattern of being higher in warmer months (spring and summer) and lower in colder months (autumn and winter).

- The least common crimes (like purse snatching, murder) account for less than 0.1% of crimes in any season.

## Seasonal ranking of the crimes recorded.

``` sql
WITH seasonal_trends AS (
  SELECT
    CASE 
      WHEN primary_type IN ('Agg Assault', 'Aggravated Assault') 
        THEN 'Aggravated Assault' 
      ELSE primary_type 
    END AS primary_type,  
    CASE
      WHEN EXTRACT(MONTH FROM timestamp) IN (3, 4, 5) THEN 'spring'
      WHEN EXTRACT(MONTH FROM timestamp) IN (6, 7, 8) THEN 'summer'
      WHEN EXTRACT(MONTH FROM timestamp) IN (9, 10, 11) THEN 'autumn'
      ELSE 'winter'
    END AS season,
    COUNT(*) AS crime_count
  FROM
    `bigquery-public-data.austin_crime.crime`
  WHERE
    timestamp IS NOT NULL
  GROUP BY
    primary_type, season  
)
SELECT
  primary_type,
  season,
  crime_count,
  RANK() OVER (PARTITION BY primary_type ORDER BY crime_count DESC) AS season_rank
FROM seasonal_trends
ORDER BY primary_type, season_rank;
```
**Summary**
From the results of this query it is clear that:
Summer is the peak season for violent crimes (Aggravated Assault, Rape, Robbery) and most theft categories.

Winter sees the highest rates of Burglary, Auto Theft, and Theft of Auto Parts.

Spring/Autumn show moderate crime levels, with spring peaking in some property crimes (e.g., Burglary/Breaking & Entering).

Burglary/Breaking & Entering: Unusually high in spring (1,549 incidents), contrary to broader burglary trends.

Theft: Auto Parts: Highest in winter (65 incidents), unlike other theft subtypes.

Purse Snatching: Extremely rare (≤3 incidents/season), with no clear seasonal pattern.

## Calculate the crime peak season

``` sql
WITH seasonal_trends AS (
  SELECT
    CASE 
      WHEN primary_type IN ('Agg Assault', 'Aggravated Assault') 
        THEN 'Aggravated Assault' 
      ELSE primary_type 
    END AS primary_type, 
    CASE
      WHEN EXTRACT(MONTH FROM timestamp) IN (3, 4, 5) THEN 'spring'
      WHEN EXTRACT(MONTH FROM timestamp) IN (6, 7, 8) THEN 'summer'
      WHEN EXTRACT(MONTH FROM timestamp) IN (9, 10, 11) THEN 'autumn'
      ELSE 'winter'
    END AS season,
    COUNT(*) AS crime_count
    FROM
    `bigquery-public-data.austin_crime.crime`
  WHERE
    timestamp IS NOT NULL
  GROUP BY
    primary_type, season
),
ranked_crimes AS (
  SELECT
    primary_type,
    season,
    crime_count,
    ROW_NUMBER() OVER (PARTITION BY primary_type ORDER BY crime_count DESC) AS rnk
  FROM seasonal_trends
)
SELECT
    primary_type,
    season AS peak_season,
    crime_count
FROM ranked_crimes
WHERE rnk = 1
ORDER BY primary_type;
```

**Summary*
1. Violent Crimes Peak in Warmer Months:

Aggravated Assault: Summer peak (1,573 incidents)

Rape: Summer peak (502 incidents)

Robbery: Summer peak (748 incidents)

Murder: Summer peak (19 incidents)

Exception: Nonnegligent Manslaughter peaks in autumn (14 incidents)

2. Property Crimes Show Divergent Patterns:

Burglary: Winter peak (2,701 incidents)

Auto Theft: Winter peak (1,606 incidents)

Theft: Auto Parts: Winter peak (65 incidents)

Theft Overall: Summer peak (14,201 incidents - 11% > winter)

## What are the yearly crime rate in the city of Austin

``` sql
SELECT
  year,
  COUNT(*) AS year_count
  FROM
  `bigquery-public-data.austin_crime.crime`
WHERE
  description IS NOT NULL 
GROUP BY
  year
ORDER  BY 
  year_count DESC
```
 Row  | year        | year_count
 ---------------------------------
    1 |        2014 |       40640
    2 |        2015 |       38572
    3 |        2016 |       37460

**Key Take aways**
This information can be used for resource allocation, policing strategies, and further investigation into why these disparities exist (e.g., population density, socio-economic factors, etc.).

The analysis shows that crime is not evenly distributed across council districts.
District 3 is the highest, followed closely by 9 and 4.
District 10 has the least crime.
The disparity between the highest and lowest is substantial (over 3 times).

**Disparities**

- The top three districts (3, 9, 4) account for 18,109 + 17,662 + 16,656 = 52,427 crimes, which is about 45.2% of the total.
  This indicates nearly half of all crimes occur in just 30% of the districts and 4 require urgent intervention

- The bottom three districts (8, 6, 10) account for 6,226 + 6,223 + 5,255 = 17,704 crimes, which is about 15.3% of the total. 

- Mid-Tier Attention: Districts like 7 (13.4k crimes) may need monitoring to prevent escalation.

**Patterns**

- There is a clear pattern: the top three districts (3, 9, 4) have significantly higher crime counts than the others.

- The middle group (districts 7, 1, 2, 5) have crime counts in the range of 10,000 to 13,500.

- The bottom three (8, 6, 10) are below 6,500.

##**Conclusion:** Austin's crime profile is strongly seasonal, with theft dominating summer months and property crimes (burglary/auto theft) rising in winter. Stakeholders should deploy seasonally-targeted interventions while maintaining year-round focus on violent crimes.

Resource Allocation:
Prioritize theft prevention (patrols, public awareness) in summer, especially for high-theft areas like retail zones.
Increase burglary/auto theft enforcement in winter.

Violent Crime Focus:
Summer/spring require heightened attention for assaults and robberies. Rape prevention efforts should align with summer trends.

##**Solutions and Strategic Implications:**

**Summer Resource Allocation**

- Prioritize patrols in theft/violent crime hotspots (public spaces, events)

- Launch public safety campaigns for personal security and vehicle/property protection

**Winter Focus Areas**

- Strengthen burglary/auto theft prevention (neighborhood watches, lighting improvements)

- Target "Theft: Auto Parts" with surveillance in parking zones

**Spring Anomalies**

- Investigate spring spikes in Burglary/B&E and Theft: BOV for unique causal factors

- Monitor Purse Snatching despite low counts due to psychological impact

- High-Volume Targets

- Theft (14,201 summer incidents) and Burglary (2,701 winter incidents) offer highest ROI for prevention efforts


##**Note:**
 
1. "Burglary" and "Burglary/Breaking & Entering" are tracked separately due to the fact that breaking and entering is a component of burglary, but not all breaking and entering constitutes burglary. AS Breaking and entering is the act of unlawfully entering a structure without permission, while burglary requires that the illegal entry be made with the intent to commit a crime inside, such as theft.

Seasonality significantly impacts crime rates, with summer driving violent offenses and winter favoring property crimes. Strategic resource allocation aligned with these patterns can optimize community safety efforts. Further analysis of socioeconomic factors (e.g., holidays, unemployment) is recommended to refine interventions.

2. Agg Assault and Aggravated Assault are the same and have been merged as values to produce more concise reporting.
