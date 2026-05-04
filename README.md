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
-- Table structure for table `procedures`
--
