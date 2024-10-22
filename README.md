# Netflix Dataset Analysis: Data Cleaning & Exploratory Data Analysis (EDA)


## Table of Contents

- [Project Overview](#project-overview)
- [Dataset](#dataset)
- [Objectives](#objectives)
- [Steps Taken in Cleaning Layoffs Dataset](#steps-taken-in-cleaning-layoffs-dataset)
  - [1. Data importing and inspection](#1-data-importing-and-inspection)
  - [2. Remove Duplicates](#2-remove-duplicates)
  - [3. Standardize the data](#3-standardize-the-data)
  - [4. Handling Null Values or Blank Values](#4-handling-null-values-or-blank-values)
  - [5. Remove Irrelevant Records and Columns](#5-remove-irrelevant-records-and-columns)
  - [6. Verify the Cleaned Dataset](#6-verify-the-cleaned-dataset)

<br>
<br>

### Project Overview
The purpose of this project is to perform Data Cleaning and Exploratory Data Analysis (EDA) on the Netflix dataset. This project aims to clean and format the data to ensure accuracy and then conduct exploratory analysis to answer business-related questions that can provide insights into Netflix's content library.

<br>

### Dataset
The dataset includes information about various content on Netflix, including movies, TV shows, and documentaries, with attributes like title, director, casts, country, date added, duration, and more.

#### About the Dataset:
- **Source:** [netflix_title.csv](https://www.kaggle.com/datasets/shivamb/netflix-shows)
- **Total Rows:** 8807+
- **Columns:** show_id, type, title, director, casts, country, date_added, release_year, rating, duration, listed_in, description

<br>

### Objectives

The project has two main objectives:

1. **Data Cleaning:** Address duplicate values, correct inconsistent formatting, and standardize the dataset for further analysis.
2. **Exploratory Data Analysis (EDA):** Perform an in-depth analysis to answer 15 business-related questions based on the Netflix content data.

<br>

### Tools
The tools used for this project:
- MySQL Workbench:  main tool used in data cleaning and exploratory
- Excel:  CSV file containing Netflix_titles

<br>
<br>

## Data Cleaning
Steps in Data Cleaning:

#### 1. Create Database, Create a Table and Import the Dataset

*Create Database*

```sql
CREATE DATABASE project;
USE projects;
```

*Create a Table*

```sql
CREATE TABLE netflix
	(show_id varchar(10), 
	type varchar(15), 
	title varchar(150),
	director text,
	casts text,
	country varchar(150),
	date_added varchar(50),
	release_year INT,
	rating varchar(10),
	duration varchar(20),
	listed_in varchar(100),
	description text );
```

*Import data with local infile.*

```sql
LOAD DATA LOCAL INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/netflix_titles.csv'
INTO TABLE netflix
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

*verify the table*

```sql
SELECT * FROM netflix;
DESCRIBE netflix;
```

*Create a new work-table to protect the raw dataset.*

```sql
CREATE TABLE netflix_exp
LIKE netflix;
```

*Copy dataset into the new work table*

```sql
INSERT INTO netflix_exp
SELECT * FROM netflix;
```

*verify the New Work Table*

```sql
SELECT * FROM netflix_exp;
```

<br>

#### 2. Remove Duplicates

*check for duplicates using CTE*

```sql
WITH netflix_duplicate
AS
	(SELECT *, 
	ROW_NUMBER() OVER(PARTITION BY type, title, director, country, release_year, duration, listed_in) As Count
	FROM netflix_exp)
SELECT * FROM netflix_duplicate
WHERE count > 1;
```

*verify the duplicates*

```sql
SELECT *
FROM netflix_exp
WHERE 
title IN 
('Love in a Puff', 'Esperando la carroza', 'Sin senos sí hay paraíso')
ORDER BY title;
```

*delete the duplicates*

```sql
WITH netflix_duplicate
AS
	(SELECT *, 
	ROW_NUMBER() OVER(PARTITION BY type, title, director, country, release_year, duration, listed_in) As Count
	FROM netflix_exp)
DELETE FROM netflix_exp
WHERE show_id IN
	(SELECT show_id FROM netflix_duplicate WHERE count >1);
```

<br>
#### 2. Remove Duplicates

- Whitespace standardization: check for Leading or Trailing Spaces in the columns

```sql
SELECT *
FROM netflix_exp
WHERE 
	title != TRIM(title) OR
	director != TRIM(director) OR
	casts != TRIM(casts) OR
  country != TRIM(country) OR
	date_added != TRIM(date_added) OR
	rating != TRIM(rating) OR
  duration != TRIM(duration) OR
	listed_in != TRIM(listed_in) OR
	description != TRIM(description);
```

*whitespaces found! update to remove the whitespaces*

```sql
UPDATE netflix_exp
SET 
	  show_id = TRIM(show_id), 
    type = TRIM(type), 
    title = TRIM(title), 
    director = TRIM(director), 
    casts = TRIM(casts), 
    country = TRIM(country), 
    date_added = TRIM(date_added), 
    release_year = TRIM(release_year), 
    rating = TRIM(rating),
    duration = TRIM(duration), 
    listed_in = TRIM(listed_in), 
    description = TRIM(description);
```

