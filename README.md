# Data-Analysis
# Netflix Movies and TV Shows Data Analysis using SQL
## Overview
This project demonstrates advanced SQL querying techniques on the Netflix dataset, focusing on both exploratory and analytical insights. Key features include:
1. Content Trends: Analyze popular genres, growth of TV shows over time, and rolling averages of content release.
2. Rankings and Insights: Use window functions to rank content, identify top directors, and determine frequent ratings.
3. Country-Specific Analysis: Highlight top countries producing content and their distribution between movies and TV shows.
4. Performance Metrics: Queries provide percentages, cumulative counts, and averages for enhanced data comprehension.
These queries showcase SQL capabilities for handling real-world datasets, offering value to anyone exploring data analytics or Netflix content trends.

## Dataset
The dataset is taken from kaggle.

# Schemas
select * from netflix

-- 1. Count the number of Movies vs TV Shows
SELECT 
type,
COUNT(*)
FROM netflix
GROUP BY 1

-- 2. Find the most common rating for movies and TV shows
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;

-- 3. List all movies released in a specific year (e.g., 2020)
SELECT * 
FROM netflix
WHERE release_year = 2020


-- 4. Find the top 5 countries with the most content on Netflix
SELECT * 
FROM
(
	SELECT 
		-- country,
		UNNEST(STRING_TO_ARRAY(country, ',')) as country,
		COUNT(*) as total_content
	FROM netflix
	GROUP BY 1
)as t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5


-- 5. Identify the longest movie
SELECT 	*
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::numeric DESC


-- 6. Find content added in the last 5 years
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'


-- 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
SELECT *
FROM
(
SELECT *,
UNNEST(STRING_TO_ARRAY(director, ',')) as director_name
FROM netflix
)
WHERE director_name = 'Rajiv Chilaka'


-- 8. List all TV shows with more than 5 seasons
SELECT *
FROM netflix
WHERE 
TYPE = 'TV Show' AND SPLIT_PART(duration, ' ', 1)::numeric > 5


-- 9. Count the number of content items in each genre
SELECT 
UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
COUNT(*) as total_content
FROM netflix
GROUP BY 1


-- 10. Find each year and the average numbers of content release by India on netflix, return top 5 year with highest avg content release !
SELECT country, release_year, COUNT(show_id) as total_release,
ROUND(COUNT(show_id)::numeric / (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100 ,2) as avg_release
FROM netflix
WHERE country = 'India' 
GROUP BY country, 2
ORDER BY avg_release DESC 
LIMIT 5


-- 11. List all movies that are documentaries
SELECT * FROM netflix
WHERE listed_in LIKE '%Documentaries'


-- 12. Find all content without a director
select *
from netflix
where director is null

-- 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
select *
from netflix
where casts ilike '%Salman khan%' and release_year > extract(year from current_date) - 10

-- 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
select 
unnest(string_to_array(casts, ',')) as actors,
count(*) as total_count
from netflix
where country ilike '%india'
group by 1
order by 2 desc
limit 10

-- 15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.
with new_table
as
(
select *,
case 
when description ilike '%kill%' or
	 description ilike '%violence%' then 'bad_content'
	 else 'good_content'
end category
from netflix
)

select category, count(*) as total_content
from new_table
group by 1


-- 16. Calculate the rolling average of titles released per year.
SELECT release_year, COUNT(*) AS yearly_count,
round(AVG(COUNT(*)) OVER (ORDER BY release_year ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS rolling_avg
FROM netflix
GROUP BY 1
ORDER BY 1 desc;


-- 17. Determine the top 3 countries producing the most titles per type 
SELECT country, type, COUNT(*) AS title_count,
RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS rank
FROM netflix
WHERE country IS NOT NULL
GROUP BY 1, 2 
order by 3 
--limit 3
--HAVING rating <= 3;


-- 18. Identify the top 3 directors with the highest number of titles on Netflix.
SELECT director, COUNT(*) AS title_count
FROM netflix
WHERE director IS NOT NULL
GROUP BY director
ORDER BY title_count DESC
LIMIT 3;

-- 19. Find the top 5 countries with the highest number of TV Shows, sorted by count.
SELECT country, COUNT(*) AS tv_show_count
FROM netflix
WHERE type = 'TV Show' AND country IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

-- 20. Retrieve all titles with "Romantic" in their genre and sort them by release year (newest first).
SELECT title, release_year, listed_in
FROM netflix
WHERE listed_in LIKE '%Romantic%'
ORDER BY 2 DESC;



