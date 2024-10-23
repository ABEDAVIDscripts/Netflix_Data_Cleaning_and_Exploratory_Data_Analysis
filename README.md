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

#### 3. Standardization

- (A). Whitespace standardization: check for Leading or Trailing Spaces in the columns

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


- (B). Standardization by categorical values : check through the columns.
  
*check with TITLE column*

```sql
SELECT
	show_id, title 
FROM netflix_exp
WHERE
	title LIKE '%#%' OR title LIKE '%*%';
```
    
*update TITLE column to remove the '#'*

```sql
UPDATE
	netflix_exp
SET 
	title = REPLACE(title, '#', '');
```

*check with COUNTRY column*

```sql
SELECT DISTINCT country FROM netflix_exp;
```

*update the COUNTRY column to remove the error in (, France, Algeria) and (, South Korea)*

```sql
UPDATE netflix_exp
SET 
	country = REPLACE(country, ', ', '')
	WHERE country LIKE ',%';
```

<br>

#### 4. Standardize the Date column

```sql
SELECT DISTINCT date_added FROM netflix_exp;
```

*covert to standard DATE format*

```sql
UPDATE netflix_exp 
SET 
	date_added = STR_TO_DATE(date_added, '%M %d, %Y')
	WHERE date_added IS NOT NULL 
	AND date_added != '';
```

<br>

#### 5. Standardize the Datatype

```sql
DESCRIBE netflix_exp;
```

*convert to DATE datatype: there are empty strings in date_added column, hence I change the empty string to a Null value*

```sql
UPDATE netflix_exp
SET 
	date_added = NULL 
	WHERE date_added = '';
```

*convert*

```sql
ALTER TABLE netflix_exp
MODIFY date_added date;
```

<br>

#### 6. Verify the updates and Prepare the Dataset for Exploration

```sql
SELECT * 
FROM netflix_exp;

DESCRIBE netflix_exp;
```

<br>
<br>
<br>

## DATA EXPLORATION

Business Problems and solution:

#### Q1. How many Type of movie do we have in this dataset?

```SQL
SELECT 
	COUNT(DISTINCT type) AS Type_of_Movie
FROM netflix_exp;
```

#### Q2. Count the number of Movies vs TV Shows

```SQL
SELECT 
	type, count(*) AS no_count
FROM netflix_exp
GROUP BY 1;    
```

#### Q3. Find the most common rating for movies and TV shows

```SQL
WITH most_common_rating
	AS
	(SELECT 
		type, rating, COUNT(*) AS COUNT,
		ROW_NUMBER() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) AS row_numb
	FROM netflix_exp
	GROUP BY 1,2)
SELECT type, rating
FROM most_common_rating
WHERE row_numb = 1;
```

#### Q4. List all movies released in a specific year (e.g 2020)

```SQL
SELECT show_id, type, title, release_year
FROM netflix_exp
	WHERE release_year = 2020
    AND type = 'movie';
```

#### Q5. Find the top 5 countries with the most content on Netlix

```SQL
SELECT 
	SUBSTRING_INDEX(Country, ',', 1) AS New_Country, 
    COUNT(*) AS Count
FROM netflix_exp
WHERE country != ''
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```

#### Q6. Identify the longest movie

*create a new column for duration minute and duration season, and extract the numeric values into them*

```SQL
ALTER TABLE netflix_exp
ADD COLUMN
	duration_minute INT,
ADD COLUMN
    duration_season INT;
```

*update the new columns*

```SQL
UPDATE netflix_exp
SET duration_minute = CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED)
WHERE duration LIKE '%min%';
```

```SQL
UPDATE netflix_exp
SET duration_season = CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED)
WHERE duration LIKE '%season%'; 
```

*Identify the longest movie*

```SQL
SELECT 
	title, MAX(duration_minute) AS MAX_NO
FROM netflix_exp
	WHERE type = 'movie'
	AND duration_minute = (SELECT MAX(duration_minute) FROM netflix_exp)
GROUP BY 1;
```

#### Q7. Find content added in the last 5 years
```SQL
SELECT *
FROM netflix_exp
	WHERE date_added >= DATE_SUB(CURDATE(), INTERVAL 5 YEAR);
```

#### Q8. Find all the movies/TV Shows by director 'Rajiv Chilaka'!

```SQL
SELECT 
	show_id, type, title, director
FROM netflix_exp
	WHERE director LIKE '%Rajiv Chilaka%';
```

#### Q9. List all TV Shows with more than 5 seasons

```SQL
SELECT 
	type, title, duration_season
FROM netflix_exp
	WHERE duration_season > 5
    AND type = 'TV show';
```    

#### Q10. Count the number of content items in each genre

```SQL
SELECT
	SUBSTRING_INDEX(listed_in, ',', 1) AS Genre,
    COUNT(*)
FROM netflix_exp
GROUP BY 1
ORDER BY 1;
```

#### Q11. Find each year and the average numbers of content release by India on Netflix. Return top 5 year with highest avg content released.

 ```SQL
WITH average_no_of_content
AS
	(SELECT 
		YEAR(date_added) AS Years, COUNT(*) AS Total
	FROM netflix_exp
		WHERE country LIKE '%India%'
	GROUP BY 1   
	ORDER BY 2 DESC) 
SELECT 
	years, AVG(Total) AS AVERAGE
FROM average_no_of_content
GROUP BY 1
LIMIT 5;
```

#### Q12. Find all movies that are documentaries

```SQL
SELECT 
	show_id, type, title
FROM netflix_exp
	WHERE type = 'movie'
    AND listed_in LIKE '%Document%';
```

#### Q13. Find all content without director

```SQL
SELECT *
FROM netflix_exp
	WHERE director = ''
    OR director IS NULL;
```

#### Q14. Find how many movies actor 'Salman Khan' appeared in the last 10 years!

```SQL
SELECT 
	count(*) AS TOTAL_MOVIES
FROM netflix_exp
	WHERE casts LIKE '%Salman Khan%'
    AND type = 'movie'
    AND date_added >= DATE_SUB(CURDATE(), INTERVAL 10 YEAR);
```    

#### Q15. Find the top 10 actors who have appeared in the highest number of movies produced in India.

```SQL
SELECT 
	SUBSTRING_INDEX(casts, ',', 1) AS Actors,
    COUNT(*)
FROM netflix_exp
	WHERE country LIKE '%India%'
    AND type = 'movie'
    AND casts != ''
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10 ;
```

#### Q16. Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'.  Count how many items fall into each category.

```SQL
SELECT 
	CASE 
    WHEN description LIKE '%kill%' OR '%violence%' THEN 'Bad'
    ELSE 'Good'
    END AS Content_Category,
    COUNT(*) AS Count
FROM netflix_exp
GROUP BY 1;
```
