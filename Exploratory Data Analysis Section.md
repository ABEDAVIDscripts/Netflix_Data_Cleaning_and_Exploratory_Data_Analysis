## Exploratory Data Analysis

## Exploratory Data Analysis

#### 16 Business Questions and It analysis:

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

#### Q4. List all movies released in a specific year (2020)

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
SET
	duration_minute = CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED)
	WHERE duration LIKE '%min%';
```

```SQL
UPDATE netflix_exp
SET
	duration_season = CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED)
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
	AND type = 'TV show'
ORDER BY duration_season;
```    

#### Q10. Count the number of content items in each genre

```SQL
SELECT
	SUBSTRING_INDEX(listed_in, ',', 1) AS Genre,
	COUNT(*) AS No_of_Content
FROM netflix_exp
GROUP BY 1
ORDER BY No_of_Content DESC;
```

#### Q11. Find each year and the average numbers of content release by United States on Netflix. Return top 5 year with highest avg content released.

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

#### Q15. Find the top 10 actors who have appeared in the highest number of movies produced in United Kingdom.

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

<BR>
