# SQL Queries for Spotify Data Exploration

Below are the SQL queries used for data exploration on the Spotify dataset:

```sql
-- Spotify Data Exploration Queries

-- Retrieve distinct artists
SELECT DISTINCT(artist) FROM spotify;

-- Retrieve distinct platforms
SELECT DISTINCT(most_played_on) FROM spotify;

-- Find the maximum duration of tracks
SELECT MAX(duration_min) FROM spotify;

-- Find the minimum duration of tracks
SELECT MIN(duration_min) FROM spotify;

-- Find tracks with a duration of 0
SELECT * FROM spotify WHERE duration_min = 0;

-- Delete tracks with a duration of 0
DELETE FROM spotify WHERE duration_min = 0;

-- Retrieve tracks with more than 1 billion streams
SELECT * FROM spotify WHERE stream >= 100000000;

-- List all albums along with their respective artists
SELECT DISTINCT(album), artist FROM spotify GROUP BY 1, 2;

-- Get the total number of comments for licensed tracks
SELECT SUM(comments) FROM spotify WHERE licensed = 'true';

-- Find all tracks belonging to album type 'single'
SELECT * FROM spotify WHERE album_type = 'single';

-- Count the total number of tracks by each artist
SELECT artist, COUNT(track) AS counts FROM spotify GROUP BY 1 ORDER BY 2 DESC;

-- Calculate the average danceability of tracks in each album
SELECT album, ROUND(AVG(danceability)::NUMERIC, 2) AS average FROM spotify GROUP BY 1 ORDER BY 2 DESC;

-- Find the top 5 tracks with the highest energy values
SELECT track, energy FROM spotify GROUP BY 1, 2 ORDER BY 2 DESC LIMIT 5;

-- List all tracks along with views and likes where official video is true
SELECT track, SUM(views), SUM(likes) FROM spotify WHERE official_video = 'true' GROUP BY 1 ORDER BY 2, 3 DESC;

-- Calculate total views for all tracks in each album
SELECT track, album, SUM(views) AS tot FROM spotify GROUP BY 1, 2 ORDER BY 3 DESC;

-- Retrieve tracks streamed more on Spotify than YouTube
SELECT * FROM (
    SELECT track,
           COALESCE(SUM(CASE WHEN most_played_on = 'Spotify' THEN stream END), 0) AS spotify_count,
           COALESCE(SUM(CASE WHEN most_played_on = 'Youtube' THEN stream END), 0) AS youtube_count
    FROM spotify
    GROUP BY 1
) t1
WHERE spotify_count > youtube_count AND spotify_count <> 0;

-- Advanced Level Queries

-- Find the top 3 most-viewed tracks for each artist using window functions
WITH view_cte AS (
    SELECT artist, track, SUM(views) AS tot,
           DENSE_RANK() OVER (PARTITION BY artist ORDER BY SUM(views) DESC) AS dk
    FROM spotify
    GROUP BY 1, 2
)
SELECT * FROM view_cte WHERE dk <= 3;

-- Find tracks where the liveness score is above the average
SELECT track, liveness FROM spotify WHERE liveness > (SELECT AVG(liveness) FROM spotify);

-- Calculate the difference between highest and lowest energy values for tracks in each album
WITH cte1 AS (
    SELECT track, album, MAX(energy) AS high_energy, MIN(energy) AS low_energy
    FROM spotify
    GROUP BY 1, 2
)
SELECT track, album, ROUND((high_energy - low_energy)::NUMERIC, 2) AS energy_diff FROM cte1 ORDER BY 3 DESC;

-- Find tracks where the energy-to-liveness ratio is greater than 1.2
SELECT track, artist, album, energy, liveness,
       (energy / liveness) AS energy_to_liveness_ratio
FROM spotify
WHERE (energy / liveness) > 1.2
ORDER BY energy_to_liveness_ratio DESC;

-- Calculate the cumulative sum of likes for tracks ordered by views using window functions
SELECT track, artist, views, likes,
       SUM(likes) OVER (ORDER BY views DESC) AS cumulative_likes
FROM spotify
ORDER BY views;

-- Find the artist with the most-streamed tracks on Spotify and their total streams
SELECT artist, SUM(stream) AS streams
FROM spotify
WHERE most_played_on = 'Spotify'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;

-- Find tracks with above-average Spotify streams and below-average YouTube views
SELECT track FROM spotify
WHERE stream > (SELECT AVG(stream) FROM spotify)
  AND views < (SELECT AVG(views) FROM spotify);

-- Find the percentage of tracks for each artist where Spotify is the top platform
SELECT artist,
       COUNT(CASE WHEN most_played_on = 'Spotify' THEN 1 END) * 100.0 / COUNT(*) AS spotify_percentage
FROM spotify
GROUP BY artist
ORDER BY 2 ASC;

-- Calculate the ratio of Spotify streams to YouTube views for each track
SELECT track, artist, stream, views,
       (stream::FLOAT / NULLIF(views, 0)) AS ratio
FROM spotify
WHERE views > 0
ORDER BY ratio DESC;

-- Find tracks where energy is high but valence is low
SELECT track, artist, energy, valence
FROM spotify
WHERE energy > 0.8 AND valence < 0.3
ORDER BY energy DESC, valence ASC;
