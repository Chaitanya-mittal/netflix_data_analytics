# Netflix Data Analytics using MySql

![NETFLIX_LOGO](https://images.ctfassets.net/y2ske730sjqp/1aONibCke6niZhgPxuiilC/2c401b05a07288746ddf3bd3943fbc76/BrandAssets_Logos_01-Wordmark.jpg?w=940)

# 17 Business Problems

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)



### 0. View all data

```sql
select * from netflix; 
```

### 1. count the number of movies and tv shows

```sql
select category , count(*)
from netflix
group by category;
```

### 2. Find the most common rating for movies and TV shows

```sql
 select category , rating ,given from 
 (select category , rating , count(*) as given , rank() Over(partition by category order by count(*) desc ) as ranking
 from netflix
 group by category , rating ) as t2
 where ranking =1 ;
```

### 3. List all movies released in a specific year 

```sql
select * 
from netflix 
where category='Movie' 
and  release_year = '2020';
```

### 4. Find the top 5 countries with the most content on Netflix

```sql
select DISTINCT(LTRIM(UNNEST(STRING_TO_ARRAY(country , ',')))) as newCountry , count(show_id) as total_content
From netflix
group by newCountry
order by total_content desc
Limit 5;
```

### 5. Max Duration movie

```sql
Select *
from netflix
where Cast(Substring(duration , 1 , length(duration)-4) As Integer) =
  (  Select MAX(CAST(SUBSTRING(duration,1,length(duration)-4) as Integer))
     from netflix
     where category = 'Movie')
and category='Movie';
```


### 6.Find content added in the last 5 years

```sql
select Current_Date - INTERVAL '5 years' ,  TO_DATE('january 22,2021','Month DD,YYYY');
select * 
from netflix
where TO_DATE(date_added , 'Month DD,YYYY') >= Current_date - Interval  '5 years';
```

### 7. Find all the movies/TV shows by director 'Rajiv Chilaka'

```sql
-- like is case sensitive
-- Ilike is not case sensitive

select * 
from netflix
where director like '%Rajiv Chilaka%';
```

### 8.List all TV shows with more than 5 seasons

```sql
select * 
from netflix 
where category = 'TV Show'
AND SPLIT_PART(duration,' ',1)::numeric >= 5;
```

### 9.Count the number of content items in each genre

```sql
select TRIM(UNNEST(STRING_TO_ARRAY(listed_in,','))) as genre, count(show_id) as total_content
from netflix
group by genre 
Order by genre , total_content desc;
```

### 10.Find each year and the average numbers of content release in India on netflix.Return top 5 year with highest avg content release!

```sql 
select EXTRACT(YEAR FROM TO_DATE(date_added,'Month DD, YYYY')) as year , ROUND((count(*)::numeric /(select count(*)
from netflix
where country like 'India')::numeric) * 100,2) as avg_content_release
from netflix
where country ILike 'India'
group by 1
order by 2 desc
LIMIT 5;

-- whenever doing calculations , if not working cast to numeric
-- TO_DATE to convert to date , Extract to extract year from date 
```

### 11.  List all movies that are documentaries

```sql
-- select * from netflix
-- where category = 'Movie'
-- and
-- 	listed_in Ilike '%documentaries%';
```


### 12. Find all content without a director

```sql
select *
from netflix
where director is null;
```

### 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!

```sql
select * 
from netflix
where category='Movie'
and
      casts ILike '%Salman Khan%'
and   Extract(Year from current_date)::numeric - release_year <=10;
```

### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.

```sql
select count(show_id) as noOfMovies, LTRIM(UNNEST(STRING_TO_ARRAY(casts,','))) as actor 
from netflix
where category='Movie'
 and
	country Ilike '%India%'
Group by 2 
Order by 1 desc, 2 asc 
LIMIT 10;
```

### 15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.

```sql
select  count(*) , 
CASE
  WHEN (description Ilike '%kill%' or description Ilike '%violence%') THEN 'Bad'
  ELSE 'GOOD'
END As Label
from netflix
Group by 2;
```

### 16. Remove Data where director , casts and country is null .

```sql
Select * into NewTable from netflix;
Insert into newtable select * fron netflix;

-- Truncate newtable;
select * from newtable;
Insert into newtable select * , Rank() OVER(PARTITION BY show_id) as rank from netflix;

delete  from newtable where rank =2;

Alter table newtable drop column rank;

delete from newtable
where casts is NULL 
 	OR director is NULL 
 	OR country is NULL;
```


### 17.Find the number of movies released each year , for every country and also return the percentage of movie released by a country for that year.

```sql
select  release_year,
TRIM(UNNEST(STRING_TO_ARRAY(country,','))) as new_country ,
count(*) as movie_released,
SUM(COUNT(*)) OVER (PARTITION BY release_year) AS movie_released_that_year,
ROUND(count(*)::numeric/(SUM(COUNT(*)) OVER (PARTITION BY release_year))::numeric *100,2) as movie_percentage_year
from newtable
where category='Movie'
group by 1,2
order by 1,2,3;
```
