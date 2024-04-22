# Video Games Analysis

## Table of Content
- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning](#data-cleaning)
- [Expoloratory Data](#exploratory-data)
- [Data Analysis](#data-analysis)
- [Results](#results)
- [Recommendations](#recommendations)
### Project Overview
This project analyzes video game sales data using SQL queries, exploring top-selling games by region and publishers' genre preferences. It also examines global sales trends and provides insights into successful publishers and popular game genres.
### Data Sources
Dataset : https://www.kaggle.com/datasets/ulrikthygepedersen/video-games-sales
### Tools
- Excel - Data Cleaning
- SQL Server - Data Analysis

### Data Cleaning
To prepare the data for analysis, the following tasks were performed:
1. Data loading and inspection.
2. Handling missing values.
3. Data cleaning and formatting involved changing data types.
### Exploratory Data
1. What are the best-selling games in each region?
2. What genres do publishers prefer?
3. How many games are there for each genre?
4. What are the top 5 genres based on global sales?
5. Which publishers have the highest global sales and how many games have they released?
6. How many games are there for each publisher?
7. What are the total global sales for each genre by year?
8. What percentage of total sales does each genre contribute?
9. What is the percentage of sales contributed by each region?
10. Which consoles were published by the same company and had games released for them?
### Data Analysis
```sql
-- Retrieve the best-selling games in each region
SELECT name,
       genre,
       year,
       publisher,
       na_sales,
       eu_sales,
       jp_sales,
       other_sales,
       global_sales
FROM (
    SELECT *,
           ROW_NUMBER() OVER (ORDER BY na_sales DESC) AS rn_na,
           ROW_NUMBER() OVER (ORDER BY eu_sales DESC) AS rn_eu,
           ROW_NUMBER() OVER (ORDER BY jp_sales DESC) AS rn_jp,
           ROW_NUMBER() OVER (ORDER BY other_sales DESC) AS rn_other,
           ROW_NUMBER() OVER (ORDER BY global_sales DESC) AS rn_global
    FROM Games.dbo.video_games_sales$
) AS ranked
WHERE rn_na = 1 OR rn_eu = 1 OR rn_jp = 1 OR rn_other = 1 OR rn_global = 1;

-- Determine the preferred genres for each publisher
WITH PublisherGenre AS (
						SELECT  publisher,
								genre,
								COUNT(*) AS game_count,
								ROW_NUMBER() OVER (PARTITION BY publisher ORDER BY COUNT(*) DESC) AS ranking
								FROM Games.dbo.video_games_sales$
								GROUP BY publisher, genre)
SELECT publisher,
		genre,
		game_count
FROM PublisherGenre
WHERE ranking = 1;
					
-- Calculate total sales for each genre
SELECT genre,
		COUNT(*) AS game_count
FROM Games.dbo.video_games_sales$
GROUP BY genre
ORDER BY game_count DESC;

-- Identify the top 5 genres based on global sales
WITH RankedGenres AS (
    SELECT genre,
           SUM(global_sales) AS total_global_sales,
           RANK() OVER (ORDER BY SUM(global_sales) DESC) AS genre_rank
    FROM Games.dbo.video_games_sales$
    GROUP BY genre
)
SELECT genre,
       total_global_sales
FROM RankedGenres
WHERE genre_rank <= 5;

-- Find publishers with the highest global sales and their game counts
WITH RankedPublisher AS (
    SELECT publisher,
           COUNT(*) AS game_count,
           SUM(global_sales) AS total_global_sales,
           RANK() OVER (ORDER BY SUM(global_sales) DESC) AS publisher_rank
    FROM Games.dbo.video_games_sales$
    GROUP BY publisher
)
SELECT publisher,
       game_count,
       publisher_rank
FROM RankedPublisher
WHERE publisher_rank <= 10
ORDER BY game_count DESC;

-- Total global sales for each genre by year
SELECT	genre,
		year,
       SUM(global_sales) AS total_global_sales
FROM Games.dbo.video_games_sales$
WHERE year IS NOT NULL
GROUP BY genre, year
ORDER BY total_global_sales DESC;

-- Determine the percentage of total sales contributed by each genre
WITH TotalGlobalSales AS (
    SELECT SUM(global_sales) AS total_sales
    FROM Games.dbo.video_games_sales$
)
SELECT genre,
       SUM(global_sales) AS genre_sales,
       (SUM(global_sales) / (SELECT total_sales FROM TotalGlobalSales)) * 100 AS percentage_of_total_sales
FROM Games.dbo.video_games_sales$
GROUP BY genre
ORDER BY genre_sales DESC;

-- Calculate the percentage of sales contributed by each region
WITH TotalGlobalSales AS (
    SELECT SUM(global_sales) AS total_sales
    FROM Games.dbo.video_games_sales$
)
SELECT (SUM(eu_sales) / (SELECT total_sales FROM TotalGlobalSales)) * 100 AS eu_percentage_sales_global,
		(SUM(jp_sales) / (SELECT total_sales FROM TotalGlobalSales)) * 100 AS jp_percentage_sales_global,
		(SUM(na_sales) / (SELECT total_sales FROM TotalGlobalSales)) * 100 AS na_percentage_sales_global,
		(SUM(other_sales) / (SELECT total_sales FROM TotalGlobalSales)) * 100 AS other_percentage_sales_global
FROM Games.dbo.video_games_sales$;

-- Retrieve consoles published by the same company with games released for them
SELECT c.console_name,
		c.company,
		v.publisher,
		v.platform,
		v.year,
		v.name
		FROM games.dbo.Console_Data$ AS c
INNER JOIN games.dbo.video_games_sales$ AS v
ON v.year = c.released_year
WHERE c.company = v.publisher
AND c.console_name = v.platform
ORDER BY year DESC;
```
### Results
#### Regional Game Sales:
- "Wii Sports" dominates in North America, "Pokemon Red/Pokemon Blue" in Japan, and "Grand Theft Auto: San Andreas" in Europe.
#### Publisher Preferences:
- Electronic Arts focuses on sports games, Activision on action, and Konami on sports as well.
#### Top Genres by Global Sales:
- Action and sports games lead global sales, followed by shooter, role-playing, and platform games.
#### Genre Contribution to Total Sales:
- Action games contribute the most to total sales (19.63%), followed by sports and shooter games.
#### Regional Sales Breakdown:
- North America leads in global sales (49.25%), followed by Europe (27.29%) and Japan (14.47%).
### Recommendations
- Publishers should capitalize on popular genres like action and sports, tailoring marketing efforts to match regional preferences.
- While maintaining focus on core genres, exploring new genres or niche markets could provide opportunities for growth.
- Companies should consider expanding into regions with untapped potential, leveraging successful titles to penetrate new markets.
- Developing strong communities around popular titles can foster brand loyalty and encourage repeat purchases.
- Constantly monitoring trends and consumer preferences is crucial for adapting strategies and staying competitive in the dynamic gaming industry.
