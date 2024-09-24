# Spotify Advanced SQL Project and Query Optimization P-6
Project Category: Advanced
[Click Here to get Dataset](https://www.kaggle.com/datasets/sanjanchaudhari/spotify-dataset)

![Spotify Logo](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_logo.jpg)

## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

```sql
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```
## Project Steps

### 1. Data Exploration
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.

### 2.. Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data. Queries are categorized into **easy**, **medium**, and **advanced** levels to help progressively develop SQL proficiency.

3. Query Optimization
In advanced stages, the focus shifts to improving query performance. Some optimization strategies include:
- **Indexing**: Adding indexes on frequently queried columns.
- **Query Execution Plan**: Using `EXPLAIN ANALYZE` to review and refine query performance.
  
4.Retrieve the names of all tracks that have more than 1 billion streams.

select * from spotify
where stream >= 100000000;

5.List all albums along with their respective artists.

select distinct(album), artist from spotify
group by 1,2

6.Get the total number of comments for tracks where licensed = TRUE.
 select sum(comments) from spotify
  where licensed= 'true';
  
7.Find all tracks that belong to the album type single.
select * from spotify
where album_type= 'single';

8.Count the total number of tracks by each artist.
select artist, count(track) as counts from spotify
group by 1
order by 2 desc;

9.Calculate the average danceability of tracks in each album.

select album ,round(avg(danceability):: numeric,2) as average from spotify
group by 1
order by 2 desc;

10.Find the top 5 tracks with the highest energy values.
select track,energy from spotify
group by 1,2
order by 2 desc
limit 5;

11.List all tracks along with their views and likes where official_video = TRUE.
select track,sum(views),sum(likes )from spotify 
where official_video= 'true'
	group by 1
	order  by 2,3 desc;
 
12.For each album, calculate the total views of all associated tracks.
select track,album,sum(views) as tot, from spotify
group by 1,2
order by 3 desc;

13.Retrieve the track names that have been streamed on Spotify more than YouTube.
select * from
	(
select track,
   coalesce (sum(case when most_played_on='Spotify' then stream end),0) as spotify_count,
   coalesce( sum(case when most_played_on='Youtube' then stream end) ,0)as youtube_count
	from spotify
	group by 1
	)t1
where spotify_count > youtube_count
and spotify_count <> 0;

14.Find the top 3 most-viewed tracks for each artist using window functions.

with view_cte as
	(
	select artist,
	    track,
	    sum(views) as tot ,
       dense_rank() over(partition by artist order by sum(views) desc) as dk
from spotify
group by 1,2
order by 1
	)
select * from view_cte
where dk<=3;

15.Write a query to find tracks where the liveness score is above the average.

select track ,liveness from spotify
where liveness > (select avg(liveness) from spotify);
16.Use a WITH clause to calculate the difference between the highest and 
lowest energy values for tracks in each album.
with cte1 as
	(
select 
   track,album,
max(energy) as high_energy,
min(energy) as low_energy from spotify
group by 1,2
	)
select track,album,
   round((high_energy-low_energy):: numeric,2)as energy_diff from cte1
order by 3 desc;

17.Find tracks where the energy-to-liveness ratio is greater than 1.2.
sELECT Track, Artist, Album, Energy, Liveness,
       (Energy / Liveness) AS energy_to_liveness_ratio
FROM spotify
WHERE (Energy / Liveness) > 1.2
ORDER BY energy_to_liveness_ratio DESC;

18.Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.

SELECT Track, Artist, Views, Likes, 
       SUM(Likes) OVER (ORDER BY Views desc) AS cumulative_likes
FROM spotify
ORDER BY Views;
19. Find the Artist with the Most Streamed Tracks on Spotify and the Total Streams

select artist,sum(stream) as streams
	from spotify
	where most_played_on ='Spotify'
group by 1
order by 2 desc
limit 1;

20.Find Tracks with Above-Average Spotify Streams and Below-Average YouTube Views
select track from spotify
where stream > (select avg(stream) from spotify)
and views < (select avg(views) from spotify);

 21.Find the Percentage of Tracks for Each Artist Where Spotify is the Top Platform
 select artist,
      COUNT(CASE WHEN most_played_on = 'Spotify' THEN 1 END) * 100.0 / COUNT(*) AS spotify_percentage
	 from spotify
group by artist
order by 2 asc

22. Calculate the Ratio of Spotify Streams to YouTube Views for Each Track
sELECT Track, Artist, Stream, Views, 
       (Stream::float / NULLIF(Views, 0)) AS ratio
FROM spotify
WHERE Views > 0
ORDER BY ratio DESC;

23.Find the Tracks Where Energy is High but Valence is Low

SELECT Track, Artist, Energy, Valence
FROM spotify
WHERE Energy > 0.8 AND Valence < 0.3
ORDER BY Energy DESC, Valence ASC;



## Query Optimization Technique 

To improve query performance, we carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - We began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **7 ms**
        - Planning time (P.T.): **0.17 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_before_index.png)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.153 ms**
        - Planning time (P.T.): **0.152 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_after_index.png)

- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%203.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%202.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%201.png)

This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project.
---

## Technology Stack
- **Database**: PostgreSQL
- **SQL Queries**: DDL, DML, Aggregations, Joins, Subqueries, Window Functions
- **Tools**: pgAdmin 4 (or any SQL editor), PostgreSQL (via Homebrew, Docker, or direct installation)

## How to Run the Project
1. Install PostgreSQL and pgAdmin (if not already installed).
2. Set up the database schema and tables using the provided normalization structure.
3. Insert the sample data into the respective tables.
4. Execute SQL queries to solve the listed problems.
5. Explore query optimization techniques for large datasets.

---

## Next Steps
- **Visualize the Data**: Use a data visualization tool like **Tableau** or **Power BI** to create dashboards based on the query results.
- **Expand Dataset**: Add more rows to the dataset for broader analysis and scalability testing.
- **Advanced Querying**: Dive deeper into query optimization and explore the performance of SQL queries on larger datasets.

---


