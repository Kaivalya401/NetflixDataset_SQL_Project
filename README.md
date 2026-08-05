# NetflixDataset_SQL_Project
![NETFLIX LOGO](NETFLIXLOGO.jpg)

# OVERVIEW

This project performs an end-to-end exploratory data analysis (EDA) on a Netflix dataset using T-SQL (Microsoft SQL Server). The analysis addresses key business questions around content distribution, regional trends, director and actor participation, genre categorization, and duration metrics. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.


# OBJECTIVE

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

# DATASET
The data for this project is sourced from the Kaggle dataset:[Netflix Dataset](https://www.kaggle.com/datasets/sahibjotchandla/netflixdata) 


### 1. Count the split between Movies and TV Shows
```
SELECT 
type, 
COUNT(*) as show_count 
FROM NetflixData 
GROUP BY type;
```
### 2. Find the most common rating for each content type
```
WITH cte1 AS (

SELECT type, 
    rating,
    RANK() OVER(PARTITION BY type ORDER BY COUNT(*)  desc) as common_rating 
    FROM NetflixData 
    GROUP BY type,rating
    )
    
    SELECT type,
    rating,
    common_rating 
    FROM cte1;
```
### 3. Extract all movies released exactly in the year 2020
```
Select * 
FROM NetflixData 
WHERE type='Movie' AND release_year ='2020';
```
###4. Identify the top 5 countries with the most titles on Netflix

WITH SplitCountry AS (
    SELECT 
        LTRIM(RTRIM(value)) AS individual_country
    FROM NetflixData
    CROSS APPLY STRING_SPLIT(country, ',')
),

CountCountry as (
SELECT individual_country,COUNT(*) as totalCountries
FROM SplitCountry
GROUP BY individual_country
)

SELECT  TOP 5 totalCountries  ,
    individual_country   
FROM CountCountry
WHERE individual_country is not null
ORDER BY  totalCountries desc;

###5. Find the single longest movie by its duration

WITH durationSplit as 
(
SELECT TRIM(REPLACE(duration, 'min', '')) AS duration_num,type,title
FROM NetflixData
CROSS APPLY STRING_SPLIT(duration, ' '))

SELECT title,
duration_num
FROM durationSplit
WHERE type ='Movie'
GROUP BY title,duration_num
ORDER BY duration_num desc;

###6. Find directors who have directed both at least one Movie and one TV Show

WITH count_shows AS (
    SELECT director,
        COUNT(CASE WHEN type = 'Movie' THEN 1 END) AS movie_count,
        COUNT(CASE WHEN type = 'TV Show' THEN 1 END) AS tv_count
    FROM NetflixData
    GROUP BY director
)
SELECT director, movie_count, tv_count
FROM count_shows
WHERE director IS NOT NULL 
  AND movie_count > 0 
  AND tv_count > 0;

###7. Find the total amount of content added over the last 5 years

SELECT *
FROM NetflixData
WHERE DATEDIFF(year, date_added, GETDATE()) <= 5;

###8. List all content classified under the "Documentaries" genre
SELECT * 
FROM NetflixData 
WHERE listed_in LIKE '%Documentaries%';

###9. Find all TV shows or movies that completely lack data for the director field
SELECT type, COUNT(*) AS missing_directors
FROM NetflixData
WHERE director='Unknown'
GROUP BY type;

###10. Find All Movies/TV Shows by Director 'Anurag Kashyap'

SELECT COUNT(*)AS Count_ofShows,
LTRIM(RTRIM(director)) as Director 
FROM NetflixData
CROSS APPLY STRING_SPLIT(director, ',')
WHERE LTRIM(RTRIM(director)) = 'Anurag Kashyap'
GROUP BY LTRIM(RTRIM(director));

###11.Final All Movies/TV Shows with title directed by India country
WITH UnnestedData AS (
    SELECT 
        TRIM(d.value) AS individual_director,
        TRIM(c.value) AS individual_country,
        type,
        title 
    FROM NetflixData
    CROSS APPLY STRING_SPLIT(director, ',') d
    CROSS APPLY STRING_SPLIT(country, ',') c
    WHERE director IS NOT NULL AND country IS NOT NULL
),

DirectorCounts AS (
    SELECT 
        individual_director AS director_name,
        COUNT(CASE WHEN type = 'Movie' THEN 1 END) AS movie_count,
        COUNT(CASE WHEN type = 'TV Show' THEN 1 END) AS tv_count,
        COUNT(*) AS total_content,
        STRING_AGG(title, ', ') AS all_titles 
    FROM UnnestedData
    WHERE individual_country = 'India'
    GROUP BY individual_director
)

SELECT 
    director_name,
    movie_count,
    tv_count,
    total_content,
    all_titles 
FROM DirectorCounts
WHERE movie_count > 0 
  AND tv_count > 0
ORDER BY total_content DESC;

###12. Count the Number of Content Items in Each Genre
SELECT 
    TRIM(value) AS individual_genre,
    COUNT(*) AS total_content
FROM NetflixData
CROSS APPLY STRING_SPLIT(listed_in, ',')
GROUP BY TRIM(value)
ORDER BY total_content DESC;


###13. Find each year and the average numbers of content release in India on netflix

WITH YearlyCounts AS (
    SELECT 
        release_year,
        COUNT(show_id) AS year_total
    FROM NetflixData
    WHERE country LIKE '%India%'
    GROUP BY release_year
)
SELECT TOP 5
    release_year,
    year_total,
    AVG(CAST(year_total AS FLOAT)) OVER() AS overall_avg_per_year
FROM YearlyCounts
ORDER BY year_total DESC;


###14. Find how many movies actor 'Salman Khan' appeared in last 10 years

SELECT * 
FROM NetflixData 
WHERE cast LIKE '%Salman%'
AND release_year > DATEPART(year,release_year)  - 10;

###15. Find the top 10 actors who have appeared in the highest number of movies produced in India

SELECT TOP 10
    TRIM(value) AS actor,
    COUNT(*) AS No_of_movies
FROM NetflixData
CROSS APPLY STRING_SPLIT(cast,',')
WHERE country = 'India'
GROUP BY TRIM(value)
ORDER BY COUNT(*) DESC;

#Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category 

SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM NetflixData
) AS categorized_content
GROUP BY category;




