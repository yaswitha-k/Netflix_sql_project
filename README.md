# Netflix Movies and TV Shows Data Analysis using SQL

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

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

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
select type, 
count(*) as total_count 
from  netflix_titles group by type;
```

**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
select 
    type,
    rating,
    COUNT(*) as rating_count
from netflix_titles
where rating is not null
group by type, rating
order by type, rating_count desc;
```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2020)

```select * 
from netflix_titles
 where release_year = 2020;
```

**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
with recursive country_split as (
    select 
        trim(substring_index(country, ',', 1)) as country,
        substring(country, length(substring_index(country, ',', 1)) + 2) as remaining
    from netflix_titles
    where  country is  not null
      and trim(country) != ''

    union all

    select
        trim(substring_index(remaining, ',', 1)),
        substring_index(remaining, length(substring_index(remaining, ',', 1)) + 2)
    from country_split
    where remaining != ''
)
select
    country,
    count(*) as total_content
from country_split
group by country
order by  total_content desc
limit 5;
```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the Longest Movie

```sql
select
    title,
    duration,
    cast(substring_index(duration, ' ', 1) AS UNSIGNED) AS minutes
from netflix_titles
where type = 'Movie'
and duration is not null
order by cast(substring_index(duration, ' ', 1) AS UNSIGNED) DESC
LIMIT 1;
```

**Objective:** Find the movie with the longest duration.

### 6. Find Content Added in the Last 5 Years

```sql
select
    title,
    type,
    date_added
from netflix_titles
where str_to_date(date_added, '%M %d, %Y') >= date_sub(curdate(), interval 5 year);
```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
select 
    title,
    type,
    release_year
from netflix_titles
where director like '%Rajiv Chilaka%';
```

**Objective:** List all content directed by 'Rajiv Chilaka'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
select
    title,
    duration,
    cast(SUBSTRING_INDEX(duration, ' ', 1) as unsigned) as seasons
from netflix_titles
where  type = 'TV Show'
and cast(substring_index(duration, ' ', 1) as unsigned) > 5;
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
with recursive genre_split as (
select
        show_id,
        trim(substring_index(listed_in, ',', 1)) as genre,
        substring(listed_in, length(substring_index(listed_in, ',', 1)) + 2) as rest
   from netflix_titles
    
   union all
    
select
        show_id,
      trim(substring_index(rest, ',', 1)),
        substring(rest, length(substring_index(rest, ',', 1)) + 2)
    from genre_split
   where  rest != ''
)
select
    genre,
  count(*) as total_content
from genre_split
group by genre
order by total_content desc;
```

**Objective:** Count the number of content items in each genre.

### 10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!

```sql
select 
    release_year,
  count(*) AS total_content
from netflix_titles
where country like'%India%'
group by release_year
order by total_content desc
limit 5;
```

**Objective:** Calculate and rank years by the average number of content releases by India.

### 11. List All Movies that are Documentaries

```sql
select title from netflix_titles 
where  
 listed_in like '%Documentaries%'  
and  type = 'Movie';
```

**Objective:** Retrieve all movies classified as documentaries.

### 12. Find All Content Without a Director

```sql
select * from 
netflix_titles where 
director='' or director is null;
```

**Objective:** List content that does not have a director.

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```sql
select 
  count(*)as  total_movies
from netflix_titles
where type = 'Movie'
and cast like '%Salman Khan%'
and release_year >= year(curdate()) - 10;
 
```

**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql
with recursive actor_split as (
    
    select 
        show_id,
        trim(substring_index(cast, ',', 1)) as actor,
       substring(cast, length(substring_index(cast, ',', 1)) + 2) as rest
    from netflix_titles
where type = 'Movie'
      and country like '%India%'
     and cast is not null
    union all

    select 
        show_id,
        trim(substring_index(rest, ',', 1)) as actor,
        substring(rest, length(substring_index(rest, ',', 1)) + 2) as rest
    from actor_split
   where  rest is not null
     and rest <> ''
)

select
    actor,
    count(*) as movie_count
from actor_split
group by actor
order by  movie_count desc
limit 10;
```

**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

```sql

select 
   case
       when lower(description) like '%kill%' 
             or lower(description) like '%violence%'
        then 'Bad'
        else 'Good'
   end as content_category,
    count(*) as total_count
from netflix_titles
group by content_category;
```

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

