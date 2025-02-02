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

### 1. Count the number of Movies vs TV Shows
```sql
	SELECT 
	type,
	COUNT(*)
	FROM netflix
	GROUP BY 1
```

### 2. Find the most common rating for movies and TV shows
```sql
	WITH RatingCounts AS (
	    SELECT type, rating,  COUNT(*) AS rating_count
	    FROM netflix
	    GROUP BY type, rating
	),
	RankedRatings AS (
	    SELECT type, rating,  rating_count,
		RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
	    FROM RatingCounts
	)
	SELECT type, rating AS most_frequent_rating
	FROM RankedRatings
	WHERE rank = 1;
```

### 3. List all movies released in a specific year (e.g., 2020)
```sql
	SELECT * 
	FROM netflix
	WHERE release_year = 2020
```

### 4. Find the top 5 countries with the most content on Netflix
```sql
	SELECT * 
	FROM
	(
		SELECT -- country, UNNEST(STRING_TO_ARRAY(country, ',')) as country,
		COUNT(*) as total_content
		FROM netflix
		GROUP BY 1
	)as t1
	WHERE country IS NOT NULL
	ORDER BY total_content DESC
	LIMIT 5
```

### 5. Identify the longest movie
```sql
	SELECT 	*
	FROM netflix
	WHERE type = 'Movie'
	ORDER BY SPLIT_PART(duration, ' ', 1)::numeric DESC
```


### 6. Find content added in the last 5 years
```sql
	SELECT *
	FROM netflix
	WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'
```

### 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
```sql
	SELECT *
	FROM
	(
	SELECT *,
	UNNEST(STRING_TO_ARRAY(director, ',')) as director_name
	FROM netflix
	)
	WHERE director_name = 'Rajiv Chilaka'
```

### 8. List all TV shows with more than 5 seasons
```sql
	SELECT *
	FROM netflix
	WHERE 
	TYPE = 'TV Show' AND SPLIT_PART(duration, ' ', 1)::numeric > 5
```

### 9. Count the number of content items in each genre
```sql
	SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(*) as total_content
	FROM netflix
	GROUP BY 1
```

### 10. Find each year and the average numbers of content release by India on netflix, return top 5 year with highest avg content release !
```sql
	SELECT country, release_year, COUNT(show_id) as total_release,
	ROUND(COUNT(show_id)::numeric / (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100 ,2) as avg_release
	FROM netflix
	WHERE country = 'India' 
	GROUP BY country, 2
	ORDER BY avg_release DESC 
	LIMIT 5
```

### 11. List all movies that are documentaries
```sql
	SELECT * FROM netflix
	WHERE listed_in LIKE '%Documentaries'
```

### 12. Find all content without a director
```sql
	select *
	from netflix
	where director is null
```

### 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
```sql
	select *
	from netflix
	where casts ilike '%Salman khan%' and release_year > extract(year from current_date) - 10
```

### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
```sql
	select 
	unnest(string_to_array(casts, ',')) as actors,
	count(*) as total_count
	from netflix
	where country ilike '%india'
	group by 1
	order by 2 desc
	limit 10
```

### 15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.
```sql
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
```


### 16. Calculate the rolling average of titles released per year.
```sql
	SELECT release_year, COUNT(*) AS yearly_count,
	round(AVG(COUNT(*)) OVER (ORDER BY release_year ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS rolling_avg
	FROM netflix
	GROUP BY 1
	ORDER BY 1 desc;
```

### 17. Determine the top 3 countries producing the most titles per type 
```sql
	SELECT country, type, COUNT(*) AS title_count,
	RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS rank
	FROM netflix
	WHERE country IS NOT NULL
	GROUP BY 1, 2 
	order by 3 
	--limit 3
	--HAVING rating <= 3;
```

### 18. Identify the top 3 directors with the highest number of titles on Netflix.
```sql
	SELECT director, COUNT(*) AS title_count
	FROM netflix
	WHERE director IS NOT NULL
	GROUP BY director
	ORDER BY title_count DESC
	LIMIT 3;
```

### 19. Find the top 5 countries with the highest number of TV Shows, sorted by count.
```sql
	SELECT country, COUNT(*) AS tv_show_count
	FROM netflix
	WHERE type = 'TV Show' AND country IS NOT NULL
	GROUP BY 1
	ORDER BY 2 DESC
	LIMIT 5;
```

### 20. Retrieve all titles with "Romantic" in their genre and sort them by release year (newest first).
```sql
	SELECT title, release_year, listed_in
	FROM netflix
	WHERE listed_in LIKE '%Romantic%'
	ORDER BY 2 DESC;
```
# Findings and conclusion
### Findings:

1. Content Distribution:
Movies make up a larger percentage of Netflix's content compared to TV Shows. 
The top genres include Documentaries, Dramas, and International TV Shows, showcasing Netflix's diverse offerings.

2. Directors and Production Insights:
The top directors, such as [Top Director Names], have contributed significantly to Netflix's library.
Countries like the United States, India, and the United Kingdom dominate the production of Netflix content.

3. Trends Over Time:
Netflix has seen consistent growth in adding TV Shows, as shown by cumulative counts of TV show additions over the years.
The rolling average of content released per year highlights spikes in production in recent years.

4. Content Characteristics:
Romantic genres and TV-MA-rated titles dominate specific regions, including India, emphasizing regional preferences.
On average, movies on Netflix are about X minutes long, while TV Shows average Y seasons.

5. Global Representation:
Countries like India and the United States produce significant TV Shows, with India also excelling in romantic genres. 
The dataset highlights the importance of international collaborations in Netflix's global strategy.

6. Unique Observations:
Duplicate titles exist, but their occurrence is minimal, suggesting high curation standards.
The most frequent ratings, such as TV-MA and PG-13, align with Netflix's target audience demographics.

### Conclusion:
The Netflix dataset reveals the platform's global reach, diverse content library, and strategic focus on high-demand genres and regions. Using SQL queries, uncovered trends in content release, production, and preferences, offering valuable insights for content creators, marketers, and data analysts.

This project demonstrates the power of SQL in analyzing real-world datasets, extracting actionable insights, and understanding the dynamics of a global streaming platform like Netflix. Future work could involve building visual dashboards to complement these findings or integrating machine learning models for content recommendation.



