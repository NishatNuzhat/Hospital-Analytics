# Hospital-Analytics
## Overview
Helped Massachusetts General Hospital to prepare their annual performance report. Analyzed patient encounters, costs, coverage and behavior trends to support planning and improve care and operations.

## The Objectives
- Encounters Overview: Trends in encounter volume, types and lengths.
- Cost and Coverage Insights: Insurance coverage, procedures and claim costs.
- Patient Behavior Analysis: Visit patterns, length of stay and readmissions.

  ## Dataset
The data for this project is sourced from Maven Analytics.

## Schema

```sql
DROP DATABASE IF EXISTS hospital_db;
CREATE DATABASE hospital_db;
USE hospital_db;

--
-- Table structure for table `payers`
--

CREATE TABLE IF NOT EXISTS payers (
    Id CHAR(36) PRIMARY KEY,
    NAME VARCHAR(100),
    ADDRESS VARCHAR(255),
    CITY VARCHAR(100),
    STATE_HEADQUARTERED CHAR(2),
    ZIP VARCHAR(10),
    PHONE VARCHAR(20)
);

--
-- Table structure for table `patients`
--

CREATE TABLE IF NOT EXISTS patients (
    Id CHAR(36) PRIMARY KEY,
    BIRTHDATE DATE,
    DEATHDATE DATE,
    PREFIX VARCHAR(10),
    FIRST VARCHAR(100),
    LAST VARCHAR(100),
    SUFFIX VARCHAR(10),
    MAIDEN VARCHAR(100),
    MARITAL CHAR(1),
    RACE VARCHAR(50),
    ETHNICITY VARCHAR(50),
    GENDER CHAR(1),
    BIRTHPLACE VARCHAR(255),
    ADDRESS VARCHAR(255),
    CITY VARCHAR(100),
    STATE VARCHAR(100),
    COUNTY VARCHAR(100),
    ZIP VARCHAR(10),
    LAT DOUBLE,
    LON DOUBLE
);

--
-- Table structure for table `procedures`
--

CREATE TABLE procedures (
    START TIMESTAMP,
    STOP TIMESTAMP,
    PATIENT CHAR(36),
    ENCOUNTER CHAR(36),
    CODE VARCHAR(20),
    DESCRIPTION VARCHAR(255),
    BASE_COST INT,
    REASONCODE VARCHAR(20),
    REASONDESCRIPTION VARCHAR(255)
);

--
-- Table structure for table `encounters`
--

CREATE TABLE encounters (
  Id CHAR(36) PRIMARY KEY,
  START DATETIME NOT NULL,
  STOP DATETIME NOT NULL,
  PATIENT CHAR(36) NOT NULL,
  ORGANIZATION CHAR(36) NOT NULL,
  PAYER CHAR(36) NOT NULL,
  ENCOUNTERCLASS VARCHAR(50),
  CODE VARCHAR(20),
  DESCRIPTION VARCHAR(255),
  BASE_ENCOUNTER_COST DECIMAL(10,2),
  TOTAL_CLAIM_COST DECIMAL(10,2),
  PAYER_COVERAGE DECIMAL(10,2),
  REASONCODE VARCHAR(20),
  REASONDESCRIPTION VARCHAR(255)
);
```
## Business Problems and Solutions

### OBJECTIVE 1: ENCOUNTERS OVERVIEW

### a. How many total encounters occurred each year?
```sql
SELECT 
YEAR(START) AS 'Year',
COUNT(ID) AS Encounter_Count
from encounters
GROUP BY 1
ORDER BY 1;
```
### b. For each year, what percentage of all encounters belonged to each encounter class
-- (ambulatory, outpatient, wellness, urgent care, emergency, and inpatient)?
```sql
SELECT 
YEAR(START) AS 'Year', 
ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'ambulatory' THEN 1 ELSE 0 END)/COUNT(*)*100,1) AS Ambulatory,
ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'outpatient' THEN 1 ELSE 0 END)/COUNT(*)*100,1) AS Outpatient,
ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'wellness' THEN 1 ELSE 0 END)/COUNT(*)*100,1) AS Wellness,
ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'urgentcare' THEN 1 ELSE 0 END)/COUNT(*)*100,1) AS Urgentcare,
ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'emergency' THEN 1 ELSE 0 END)/COUNT(*)*100,1) AS Emergency,
ROUND(SUM(CASE WHEN ENCOUNTERCLASS = 'inpatient' THEN 1 ELSE 0 END)/COUNT(*)*100,1) AS Inpatient
FROM encounters
GROUP BY 1
ORDER BY 1;
```
### c. What percentage of encounters were over 24 hours versus under 24 hours?
```sql
SELECT
ROUND(SUM(CASE WHEN timestampdiff(HOUR,START,STOP) > 24 THEN 1 ELSE 0 END )/COUNT(*)*100,1) AS 'OVER 24 HOURS',
ROUND(SUM(CASE WHEN timestampdiff(HOUR,START,STOP) = 24 THEN 1 ELSE 0 END )/COUNT(*)*100,1) AS '24 HOURS',
ROUND(SUM(CASE WHEN timestampdiff(HOUR,START,STOP)< 24 THEN 1 ELSE 0 END)/COUNT(*)*100,1) AS 'UNDER 24 HOURS'
FROM 
encounters;
```
### OBJECTIVE 2: COST & COVERAGE INSIGHTS

### a. How many encounters had zero payer coverage, and what percentage of total encounters does this represent?
```sql
select 
SUM(CASE WHEN PAYER_COVERAGE = 0 THEN 1 ELSE 0 END) AS 'Zero payer coverage',
ROUND(SUM(CASE WHEN PAYER_COVERAGE = 0 THEN 1 ELSE 0 END)/COUNT(*)*100,1) AS 'Percentage of zero payer coverage'
from
encounters;
```
### b. What are the top 10 most frequent procedures performed and the average base cost for each?
```sql
select 
CODE,DESCRIPTION ,COUNT(*) AS total_count,ROUND(AVG(BASE_COST),2) as avg_cost
from procedures
group by 1,2
order by 3 desc
limit 10;
```
### c. What are the top 10 procedures with the highest average base cost and the number of times they were performed?
```sql
select CODE,DESCRIPTION,ROUND(AVG(BASE_COST),2) as avg_cost,COUNT(DESCRIPTION) AS total_count
from procedures
group by 1,2
order by 3 desc
limit 10;
```
### d. What is the average total claim cost for encounters, broken down by payer?
```sql
select 
p.NAME as payer, ROUND(AVG(e.TOTAL_CLAIM_COST),2) as avg_total_claim_cost
from encounters e
left join payers p 
on e.PAYER =p.ID
group by 1
order by 2 desc;
```
### OBJECTIVE 3: PATIENT BEHAVIOR ANALYSIS

### a. How many unique patients were admitted each quarter over time?
```sql
select YEAR(START) AS 'YEAR',QUARTER(START) AS 'QUARTER',
COUNT(distinct PATIENT) as unique_patients
from encounters
group by 1,2
Order by 1;
```
### b. How many patients were readmitted within 30 days of a previous encounter?
```sql
With cte as (select 
PATIENT,START,STOP,
LEAD(START) over (partition by PATIENT order by START) AS next_start_date
from
encounters)
select
COUNT(distinct PATIENT) AS num_patients
from
cte
where
datediff(next_start_date,STOP) < 30;
```
### c. Which patients had the most readmissions?
```sql
With cte as (select 
PATIENT,START,STOP,
LEAD(START) over (partition by PATIENT order by START) AS next_start_date
from
encounters)
select
PATIENT,COUNT(*) AS num_readmissions
from
cte
where
datediff(next_start_date,STOP) < 30
group by 1
order by 2 desc;
```
