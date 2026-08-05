# NetflixDataset_SQL_Project
[Netflix LOGO] {https://github.com/Kaivalya401/NetflixDataset_SQL_Project/blob/main/NETFLIX.png}

#OVERVIEW
This project performs an end-to-end exploratory data analysis (EDA) on a Netflix dataset using T-SQL (Microsoft SQL Server). The analysis addresses key business questions around content distribution, regional trends, director and actor participation, genre categorization, and duration metrics. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.


#OBJECTIVE
- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

#DATASET
The data for this project is sourced from the Kaggle dataset:

Dataset Link:  https://www.kaggle.com/datasets/sahibjotchandla/netflixdata 

#Schema
DROP TABLE IF EXISTS NetflixDataset;


CREATE TABLE dbo.NetflixData (
    show_id NVARCHAR(50),
    show_type NVARCHAR(50),
    title NVARCHAR(MAX),
    director NVARCHAR(MAX),
    cast VARCHAR(MAX),
    country NVARCHAR(MAX),
    date_added NVARCHAR(100),
    release_year INT,
    rating NVARCHAR(50),
    duration NVARCHAR(50),
    listed_in NVARCHAR(MAX),
    description NVARCHAR(MAX)
);

#Business Problems and Solutions

# 1. Count the split between Movies and TV Shows
SELECT 
type, 
COUNT(*) as show_count 
FROM NetflixData 
GROUP BY type;

Objective : Count the split between Movies and TV Shows.

#2. Find the most common rating for each content type
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
Objective : Rank content ratings within each type (Movie vs TV Show)

# 3. Extract all movies released exactly in the year 2020
Select * 
FROM NetflixData 
WHERE type='Movie' AND release_year ='2020';

Objective : Extract all movies released specifically in the year 2020


