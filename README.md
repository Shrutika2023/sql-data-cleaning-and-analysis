# NICS-Firearm-Background-Check Data Cleaning and Analysis using SQL
Analyze trends and insights in the NICS Firearm Background Checks dataset. Utilize SQL queries, documentation, and reports for data cleaning and analysis. 

The FBI created the NICS (National Instant Criminal Background Check System) in collaboration with the ATF and local law enforcement agencies following the implementation of the Brady Act in 1998. This system enables background checks on firearm applicants. Monthly data has been released by the FBI since November 1998, providing insights into approved and denied requests. Over 300 million requests have been approved, while 1.5 million have been denied.


#### Source
---
The FBI releases monthly NICS Firearm Background Checks data in PDF format. A GitHub repository maintained by Jeremy Singer Vine from Buzzfeed provides a parsed CSV version of the data. Access the CSV file here:

https://raw.githubusercontent.com/BuzzFeedNews/nics-firearm-background-checks/master/data/nics-firearm-background-checks.csv
<br>

#### Settting up Database
---
First let's create a new sqlite database

```bash
sqlite3 nics-firearm.db 
```

Now, we populate this db using the csv file that we downloaded from the link above. To do this first, we create a new table setting the appopriate data type for each column.

<details>
    <summary>Create Table Query</summary>
    
```sql
CREATE TABLE nics (
    month TEXT NOT NULL,
    state TEXT NOT NULL,
    permit INTEGER,
    permit_recheck INTEGER,
    handgun INTEGER,
    long_gun INTEGER,
    other INTEGER,
    multiple INTEGER,
    admin INTEGER,
    prepawn_handgun INTEGER,
    prepawn_long_gun INTEGER,
    prepawn_other INTEGER,
    redemption_handgun INTEGER,
    redemption_long_gun INTEGER,
    redemption_other INTEGER,
    returned_handgun INTEGER,
    returned_long_gun INTEGER,
    returned_other INTEGER,
    rentals_handgun INTEGER,
    rentals_long_gun INTEGER,
    private_sale_handgun INTEGER,
    private_sale_long_gun INTEGER,
    private_sale_other INTEGER,
    return_to_seller_handgun INTEGER,
    return_to_seller_long_gun INTEGER,
    return_to_seller_other INTEGER,
    totals INTEGER
);

```
</details>

<br>
Then we can import the data from the csv file to the new table nics that we just created.

```bash
sqlite3 nics-firearm.db
.mode csv
.import '/Users/shrutikapandey/Document/sql-data-cleaning-and-analysis/nics-firearm-background-checks.csv' nics
```

We now have successfully imported the records from the csv file to the sqlite database.
<br>

#### Data Validation
---
This section focuses on data cleaning and integrity by identifying and addressing potential issues such as duplicate entries and missing values in the NICS Firearm Background Checks dataset.

**1. Duplicate Entries:**
Let's perform analysis to identify any duplicate entries within the dataset for months and state.


```sql
SELECT COUNT(*) AS num_duplicates
FROM (
 SELECT month, state, COUNT(*) AS count
 FROM nics
 GROUP BY month, state HAVING count > 1
);
```
The result is '0' which means there are no duplicate records that violate the uniqueness of month and state columns.

**2. Missing Values:**
Let's conduct an assessment to identify missing values across the dataset. By examining each column, such as month, state, permit etc. we determine the presence of any null or empty values. Once we find out the number of null/blank values, we can transform it to our needs in order to perform aggregations more accurately.

Let's find out how many colummns have null or blank values:

<details>
    <summary>Query</summary>
    
```sql
SELECT
COUNT(CASE WHEN month IS NULL OR month = '' THEN 1 END) AS null_month_count,
COUNT(CASE WHEN state IS NULL OR state = '' THEN 1 END) AS null_state_count,
COUNT(CASE WHEN permit IS NULL OR permit = '' THEN 1 END) AS null_permit_count,
COUNT(CASE WHEN permit_recheck is NULL or permit_recheck = '' THEN 1 END) AS null_permit_recheck_count,
COUNT(CASE WHEN handgun is NULL or handgun = '' THEN 1 END) AS null_handgun_count,
COUNT(CASE WHEN long_gun is NULL or long_gun = '' THEN 1 END) AS null_long_gun_count,
COUNT(CASE WHEN other is NULL or other = '' THEN 1 END) AS null_other_count,
COUNT(CASE WHEN multiple is NULL or multiple = '' THEN 1 END) AS null_multiple_count,
COUNT(CASE WHEN admin is NULL or admin = '' THEN 1 END) AS null_admin_count,
COUNT(CASE WHEN prepawn_handgun is NULL or prepawn_handgun = '' THEN 1 END) AS null_prepawn_handgun_count,
COUNT(CASE WHEN prepawn_long_gun is NULL or prepawn_long_gun = '' THEN 1 END) AS null_prepawn_long_gun_count,
COUNT(CASE WHEN prepawn_other is NULL or prepawn_other = '' THEN 1 END) AS null_prepawn_other_count,
COUNT(CASE WHEN redemption_handgun is NULL or redemption_handgun = '' THEN 1 END) AS null_redemption_handgun_count,
COUNT(CASE WHEN redemption_long_gun is NULL or redemption_long_gun = '' THEN 1 END) AS null_redemption_long_gun_count,
COUNT(CASE WHEN redemption_other is NULL or redemption_other = '' THEN 1 END) AS null_redemption_other_count,
COUNT(CASE WHEN returned_handgun is NULL or returned_handgun = '' THEN 1 END) AS null_returned_handgun_count,
COUNT(CASE WHEN returned_long_gun is NULL or returned_long_gun = '' THEN 1 END) AS null_returned_long_gun_count,
COUNT(CASE WHEN returned_other is NULL or returned_other = '' THEN 1 END) AS null_returned_other_count,
COUNT(CASE WHEN rentals_handgun is NULL or rentals_handgun = '' THEN 1 END) AS null_rentals_handgun_count,
COUNT(CASE WHEN rentals_long_gun is NULL or rentals_long_gun = '' THEN 1 END) AS null_rentals_long_gun_count,
COUNT(CASE WHEN private_sale_handgun is NULL or private_sale_handgun = '' THEN 1 END) AS null_private_sale_handgun_count,
COUNT(CASE WHEN private_sale_long_gun is NULL or private_sale_long_gun = '' THEN 1 END) AS null_private_sale_long_gun_count,
COUNT(CASE WHEN private_sale_other is NULL or private_sale_other = '' THEN 1 END) AS null_private_sale_other_count,
COUNT(CASE WHEN return_to_seller_handgun is NULL or return_to_seller_handgun = '' THEN 1 END) AS null_return_to_seller_handgun_count,
COUNT(CASE WHEN return_to_seller_long_gun is NULL or return_to_seller_long_gun = '' THEN 1 END) AS null_return_to_seller_long_gun_count,
COUNT(CASE WHEN return_to_seller_other is NULL or return_to_seller_other = '' THEN 1 END) AS null_return_to_seller_other_count,
COUNT(CASE WHEN totals is NULL THEN 1 END) AS null_totals_count
FROM nics;
```
</details>
<br>
The above query returns then number of blank/null rows for each column. Now, we can replace the rest of the null/blank values with 0 as they’re all integer columns.
Below is an example to do this for the column `permit`.

```sql
UPDATE nics
SET permit = 0
WHERE permit IS NULL OR permit = '';
```

We can repeat this for all the remaining colums which is given below after which we can now perform any aggregations on numeric columns without an issue.

<details>
    <summary>Query</summary>
    
```sql
UPDATE nics
SET permit = 0
WHERE permit IS NULL OR permit = '';
UPDATE nics
SET permit_recheck = 0
WHERE permit_recheck IS NULL OR permit_recheck = '';
UPDATE nics
SET handgun = 0
WHERE handgun IS NULL OR handgun = '';
UPDATE nics
SET long_gun = 0
WHERE long_gun IS NULL OR long_gun = '';
UPDATE nics
SET other = 0
WHERE other IS NULL OR other = '';
UPDATE nics
SET multiple = 0
WHERE multiple IS NULL OR multiple = '';
UPDATE nics
SET admin = 0
WHERE admin IS NULL OR admin = '';
UPDATE nics
SET prepawn_handgun = 0
WHERE prepawn_handgun IS NULL OR prepawn_handgun = '';
UPDATE nics
SET prepawn_long_gun = 0
WHERE prepawn_long_gun IS NULL OR prepawn_long_gun = '';
UPDATE nics
SET prepawn_other = 0
WHERE prepawn_other IS NULL OR prepawn_other = '';
UPDATE nics
SET redemption_handgun = 0
WHERE redemption_handgun IS NULL OR redemption_handgun = '';
UPDATE nics
SET redemption_long_gun = 0
WHERE redemption_long_gun IS NULL OR redemption_long_gun = '';
UPDATE nics
SET redemption_other = 0
WHERE redemption_other IS NULL OR redemption_other = '';
UPDATE nics
SET returned_handgun = 0
WHERE returned_handgun IS NULL OR returned_handgun = '';
UPDATE nics
SET returned_long_gun = 0
WHERE returned_long_gun IS NULL OR returned_long_gun = '';
UPDATE nics
SET returned_other = 0
WHERE returned_other IS NULL OR returned_other = '';
UPDATE nics
SET rentals_handgun = 0
WHERE rentals_handgun IS NULL OR rentals_handgun = '';
UPDATE nics
SET rentals_long_gun = 0
WHERE rentals_long_gun IS NULL OR rentals_long_gun = '';
UPDATE nics
SET private_sale_handgun = 0
WHERE private_sale_handgun IS NULL OR private_sale_handgun = '';
UPDATE nics
SET private_sale_long_gun = 0
WHERE private_sale_long_gun IS NULL OR private_sale_long_gun = '';
UPDATE nics
SET private_sale_other = 0
WHERE private_sale_other IS NULL OR private_sale_other = '';
UPDATE nics
SET return_to_seller_handgun = 0
WHERE return_to_seller_handgun IS NULL OR return_to_seller_handgun = '';
UPDATE nics
SET return_to_seller_long_gun = 0
WHERE return_to_seller_long_gun IS NULL OR return_to_seller_long_gun = '';
UPDATE nics
SET return_to_seller_other = 0
WHERE return_to_seller_other IS NULL OR return_to_seller_other = '';
UPDATE nics
SET totals = 0
WHERE totals IS NULL OR totals = '';
```
</details>

 <br> 

Now that we've completed our data cleaning, we can move towards conducting descriptive analysis as well as aggregations. Please note that the specific techniques and methods employed for data validation may vary depending on the characteristics and requirements of the dataset.

<br>
####Descriptive Analysis
---

In this section, let's perform a descriptive analysis of the NICS Firearm Background Checks dataset, focusing on the distribution of background check totals across various categories. By examining the dataset's key categories such as months, states and permits we aim to gain insights into the patterns and variations in background check totals.

> Q: What’s the total gun permits issued by each State? Which State issued the highest and the least number of gun permits between 1998- 2023?

```sql

SELECT state, SUM(permit) AS permit_totals
FROM nics
GROUP BY state
ORDER BY permit_totals DESC;

```
By running this query, we obtain a ranked list of States based on the total number of gun permits issued in descending order. Kentucky emerges as the leading state with the highest number of permits, while Vermont holds the record for the lowest non-zero number of permits issued. Notably, Rhode Island, Puerto Rico, New Jersey, Mariana Islands, and Guam did not issue any gun permits.





> Q: In which year were the most gun permits issued between 1998-2023?


```sql
SELECT DISTINCT substr(month, 0, 5) AS year, Sum(permit) AS permit_sum
FROM nics
GROUP BY year
ORDER BY permit_sum DESC;

```

After running this query, we found that most gun permits were issued in 2016.


> Q: What are the most selling guns in all 50 States between 1998-2023?

```sql
SELECT
SUM(handgun) AS handgun_total,
SUM(long_gun) AS long_gun_total,
SUM(other) AS other_gun_total
FROM nics;

```
Based on the result, it is evident that the highest number of firearms sold were categorized as "long guns" (130267067) followed by "handguns" (123767991) and "other" (5827105) types of guns.


This descriptive analysis aims to provide a comprehensive overview of the distribution of background check totals across different categories in the NICS Firearm Background Checks dataset. By studying patterns and variations, we can gain a better understanding of firearm-related activities and how they differ in different regions and over time.