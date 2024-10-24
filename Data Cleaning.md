## Data Cleaning
Steps in the Netflix Data Cleaning:

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

#### 3. General Standardization

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

#### 4. Date Standardization

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

#### 5. Datatype Standardization

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

#### 6. Verification and Dataset Preparation

```sql
SELECT * 
FROM netflix_exp;

DESCRIBE netflix_exp;
```