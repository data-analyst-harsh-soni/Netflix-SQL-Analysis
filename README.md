# 🎬 Netflix Movies & TV Shows Data Analysis Using PostgreSQL

![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![SQL](https://img.shields.io/badge/Language-SQL-orange?style=for-the-badge&logo=postgresql&logoColor=white)
![Status](https://img.shields.io/badge/Project-Completed-success?style=for-the-badge)

## 📌 Project Overview

This project focuses on analyzing Netflix Movies and TV Shows data using **PostgreSQL** to uncover meaningful insights about Netflix's content library.

The analysis explores content distribution, ratings, release trends, country-wise contributions, genres, directors, actors, and content descriptions.

A collection of 15 SQL business questions has been solved using SQL concepts such as:

- Aggregate Functions (`COUNT`, `AVG`)
- GROUP BY and ORDER BY
- Common Table Expressions (CTEs)
- Window Functions (`RANK`)
- Subqueries
- String Functions
- Array Functions (`STRING_TO_ARRAY`, `UNNEST`)
- Conditional Logic (`CASE WHEN`)
- Date Functions
- Pattern Matching (`LIKE`, `ILIKE`)
- Type Casting

The objective is to demonstrate practical SQL querying and analytical problem-solving skills through a real-world entertainment dataset.

## 🎯 Project Objectives

- Analyze the distribution of Movies and TV Shows.
- Identify the most frequently assigned content ratings.
- Explore content release patterns across different years.
- Identify countries contributing the most Netflix content.
- Analyze movie durations and TV show seasons.
- Examine genre-wise content distribution.
- Explore Indian content production trends.
- Identify popular actors and directors.
- Categorize content using keywords found in descriptions.

## 📂 Dataset Information

The dataset is sourced from Kaggle.

**Dataset:** Netflix Movies and TV Shows

**Source:** Shivam Bansal

**Dataset Link:** https://www.kaggle.com/datasets/shivamb/netflix-shows

The dataset contains information about Netflix titles, including:

| Column | Description |
|---|---|
| show_id | Unique identifier for each title |
| type | Movie or TV Show |
| title | Title of the content |
| director | Director name |
| casts | Cast members |
| country | Country of production |
| date_added | Date added to Netflix |
| release_year | Original release year |
| rating | Content rating |
| duration | Movie duration or TV show seasons |
| listed_in | Genres and categories |
| description | Content description |

## 🗄️ Database Schema

The project uses PostgreSQL as its database management system.

### Table Creation

```sql
DROP TABLE IF EXISTS netflix;

CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## 💻 Business Questions & SQL Solutions

The following 15 business questions were solved using PostgreSQL.

### 1. Find the Total Number of Movies and TV Shows

**Business Question:** How many Movies and TV Shows are available in the Netflix dataset?

```sql
SELECT
    type,
    COUNT(*) AS total_content
FROM netflix
GROUP BY type;
```

**Objective:** Understand the distribution of different content types on Netflix.

### 2. Identify the Most Common Rating for Movies and TV Shows

**Business Question:** Which content rating occurs most frequently for Movies and TV Shows?

```sql
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
        RANK() OVER (
            PARTITION BY type
            ORDER BY rating_count DESC
        ) AS rank
    FROM RatingCounts
)
SELECT
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

**Objective:** Identify the most frequently assigned rating for each content type using CTEs and window functions.

### 3. Retrieve Movies Released in a Specific Year (2020)

**Business Question:** Find all Netflix movies released in 2020.

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
AND release_year = 2020;
```

**Objective:** Filter Netflix movies based on their release year.

### 4. Find the Top 5 Countries with the Highest Netflix Content

**Business Question:** Which five countries have contributed the most content to Netflix?

```sql
SELECT *
FROM
(
    SELECT
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

**Objective:** Analyze geographical content distribution using PostgreSQL array and string functions.

### 5. Identify the Longest Movie on Netflix

**Business Question:** Which movie has the longest duration?

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

**Objective:** Identify movies with the highest runtime by extracting numerical values from the duration column.

### 6. Find Content Added to Netflix in the Last 5 Years

**Business Question:** Retrieve Netflix content added during the last five years.

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY')
    >= CURRENT_DATE - INTERVAL '5 years';
```

**Objective:** Analyze recently added content using PostgreSQL date conversion and interval functions.

### 7. Find All Content Directed by Rajiv Chilaka

**Business Question:** Retrieve all movies and TV shows associated with director Rajiv Chilaka.

```sql
SELECT *
FROM
(
    SELECT
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS director_data
WHERE director_name = 'Rajiv Chilaka';
```

**Objective:** Extract individual director names from comma-separated values and identify matching content.

### 8. Find TV Shows with More Than 5 Seasons

**Business Question:** Which Netflix TV shows have more than five seasons?

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

**Objective:** Identify long-running TV shows using string extraction and numerical comparison.

### 9. Calculate the Number of Content Items in Each Genre

**Business Question:** How many Netflix titles belong to each genre?

```sql
SELECT
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```

**Objective:** Understand genre-wise content distribution using PostgreSQL array functions.

### 10. Find the Top 5 Years with the Highest Share of Indian Content Releases

**Business Question:** For each release year, calculate the number of Indian content releases and their percentage share of total Indian content. Return the top five years.

```sql
SELECT
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::NUMERIC /
        (
            SELECT COUNT(show_id)
            FROM netflix
            WHERE country = 'India'
        )::NUMERIC * 100,
        2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

**Objective:** Analyze the yearly contribution of Indian content to Netflix's overall Indian content collection.

### 11. Retrieve All Documentaries on Netflix

**Business Question:** Find all Netflix titles classified under Documentaries.

```sql
SELECT *
FROM netflix
WHERE listed_in LIKE '%Documentaries%';
```

**Objective:** Identify documentary content using SQL pattern matching.

### 12. Find All Content Without a Director

**Business Question:** Which Netflix titles do not have director information?

```sql
SELECT *
FROM netflix
WHERE director IS NULL;
```

**Objective:** Identify missing director information in the dataset.

### 13. Find Movies Featuring Salman Khan Released in the Last 10 Years

**Business Question:** Retrieve movies featuring Salman Khan released within the last ten years.

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
AND casts LIKE '%Salman Khan%'
AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

**Objective:** Filter movie records based on actor names and release-year conditions.

### 14. Find the Top 10 Actors with the Most Appearances in Indian Content

**Business Question:** Who are the top ten actors appearing most frequently in Indian-produced Netflix content?

```sql
SELECT
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*) AS total_appearances
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

**Objective:** Analyze actor appearances and identify frequently featured actors in Indian content.

### 15. Categorize Netflix Content Using Keywords in Descriptions

**Business Question:** Categorize Netflix content based on whether its description contains the keywords 'kill' or 'violence'.

- **Bad:** Description contains either keyword.
- **Good:** Description contains neither keyword.

Also calculate the number of titles in each category for Movies and TV Shows.

```sql
SELECT
    category,
    type,
    COUNT(*) AS content_count
FROM (
    SELECT
        *,
        CASE
            WHEN description ILIKE '%kill%'
              OR description ILIKE '%violence%'
            THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category, type
ORDER BY type;
```

**Objective:** Apply conditional logic and case-insensitive pattern matching to categorize Netflix content.

## 📊 Key SQL Concepts Demonstrated

| SQL Concept | Application |
|---|---|
| Aggregate Functions | Counting content and calculating totals |
| GROUP BY | Grouping records by type, country and genre |
| ORDER BY | Ranking and sorting query results |
| CTEs | Organizing complex SQL queries |
| Window Functions | Finding the most common ratings |
| RANK() | Ranking ratings within content types |
| Subqueries | Calculating percentage contributions |
| UNNEST() | Expanding array elements into rows |
| STRING_TO_ARRAY() | Splitting comma-separated values |
| SPLIT_PART() | Extracting duration values |
| CASE WHEN | Categorizing content |
| ILIKE | Case-insensitive keyword matching |
| TO_DATE() | Converting text into date values |
| EXTRACT() | Extracting year information |

## 🔍 Findings & Conclusion

This project demonstrates how PostgreSQL can be used to extract meaningful insights from a real-world entertainment dataset.

Through 15 business questions, the analysis explores:

- **Content Distribution:** Comparison between Movies and TV Shows.
- **Ratings Analysis:** Identification of the most common ratings across content types.
- **Geographical Analysis:** Identification of countries contributing the most content.
- **Content Duration:** Exploration of movie runtimes and TV show seasons.
- **Genre Analysis:** Distribution of Netflix titles across different genres.
- **Indian Content:** Analysis of Indian content release patterns and actor appearances.
- **Data Quality:** Identification of records with missing director information.
- **Text Analysis:** Categorization of content using description keywords.

The project strengthened my practical understanding of SQL, data exploration, analytical thinking, and business problem-solving.

## 🛠️ Tools & Technologies

- PostgreSQL
- SQL
- pgAdmin 4
- Kaggle Dataset

## 🚀 How to Run This Project

1. Download the Netflix dataset from Kaggle.
2. Install PostgreSQL and pgAdmin 4.
3. Create a new PostgreSQL database.
4. Execute the table creation SQL script.
5. Import the dataset into the `netflix` table.
6. Execute the SQL queries provided in this repository.
7. Explore the results and analyze the findings.

## 👨‍💻 Author

**Harsh Soni**

Aspiring Data Analyst | Data Engineer | SQL Enthusiast

I enjoy working with data, solving business problems, developing analytical solutions, and transforming raw data into meaningful insights.

### Connect With Me

- **LinkedIn:** [Connect with me](https://www.linkedin.com/in/YOUR_LINKEDIN_USERNAME/)
- **GitHub:** [Explore my repositories](https://github.com/YOUR_GITHUB_USERNAME)
- **Portfolio:** [Visit my Portfolio](https://datain.vercel.app/)

---

⭐ If you find this project useful, consider giving this repository a star.

**Thank you for visiting my project!**
