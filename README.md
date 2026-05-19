# Spotify Advanced SQL Project & Query Optimization

**Project Category:** Advanced SQL  
📦 [Click Here to Get the Dataset](https://www.kaggle.com/datasets/sanjanchaudhari/spotify-dataset)

![Spotify Logo](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_logo.jpg)

---

## 📌 Overview

This project involves in-depth analysis of a **Spotify dataset** containing rich attributes about tracks, albums, and artists using **PostgreSQL**. The project covers an end-to-end SQL workflow — from data cleaning and exploration to writing queries of increasing complexity and optimizing their performance.

The primary goals are to:
- Practice **advanced SQL skills** (CTEs, window functions, subqueries)
- Generate **meaningful insights** from music streaming data
- Demonstrate **query optimization** techniques using indexing

---

## 🗄️ Database Schema

```sql
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist          VARCHAR(255),
    track           VARCHAR(255),
    album           VARCHAR(255),
    album_type      VARCHAR(50),
    danceability    FLOAT,
    energy          FLOAT,
    loudness        FLOAT,
    speechiness     FLOAT,
    acousticness    FLOAT,
    instrumentalness FLOAT,
    liveness        FLOAT,
    valence         FLOAT,
    tempo           FLOAT,
    duration_min    FLOAT,
    title           VARCHAR(255),
    channel         VARCHAR(255),
    views           FLOAT,
    likes           BIGINT,
    comments        BIGINT,
    licensed        BOOLEAN,
    official_video  BOOLEAN,
    stream          BIGINT,
    energy_liveness FLOAT,
    most_played_on  VARCHAR(50)
);
```

---

## 🔍 Project Steps

### 1. Data Exploration
Understand the structure and content of the dataset before writing queries:
- `artist` — Performer of the track
- `track` — Song name
- `album` — Album the track belongs to
- `album_type` — Type of album (e.g., single, album)
- Audio features: `danceability`, `energy`, `loudness`, `tempo`, `liveness`, `valence`, and more

### 2. Data Cleaning
```sql
-- Remove tracks with zero duration
DELETE FROM spotify
WHERE duration_min = 0;
```

### 3. Querying the Data
Queries are organized into three levels of complexity:

| Level    | Focus |
|----------|-------|
| 🟢 Easy   | Filtering, basic aggregations, simple retrievals |
| 🟡 Medium | Grouping, joins, aggregation functions |
| 🔴 Advanced | CTEs, window functions, nested subqueries, optimization |

### 4. Query Optimization
Improving performance using indexing and execution plan analysis.

---

## 📝 some Practice Questions & Solutions

---

### 🟡 Medium Level

**6. Average danceability per album**
```sql
SELECT 
    album,
    ROUND(AVG(danceability)::NUMERIC, 3) AS avg_danceability
FROM spotify
GROUP BY album
ORDER BY avg_danceability DESC;
```

**9. Total views per album (with cumulative view using window function)**
```sql
WITH cte AS (
    SELECT album, track, SUM(views) AS total_views
    FROM spotify
    GROUP BY album, track
)
SELECT 
    album,
    track,
    total_views,
    SUM(total_views) OVER (PARTITION BY album) AS album_total_views
FROM cte
ORDER BY album_total_views DESC;
```

---

### 🔴 Advanced Level

**11. Top 3 most-viewed tracks per artist (Window Function)**
```sql
WITH cte AS (
    SELECT
        artist,
        track,
        SUM(views) AS total_views,
        ROW_NUMBER() OVER (PARTITION BY artist ORDER BY SUM(views) DESC) AS rnk
    FROM spotify
    GROUP BY artist, track
)
SELECT *
FROM cte
WHERE rnk <= 3;
```

**12. Tracks with liveness score above average**
```sql
SELECT DISTINCT track, liveness
FROM spotify
WHERE liveness > (
    SELECT ROUND(AVG(liveness)::NUMERIC, 2) FROM spotify
);
```

**13. Energy stats per album using Window Functions**
```sql
SELECT DISTINCT
    album,
    MAX(ROUND(energy_liveness::NUMERIC, 2)) OVER (PARTITION BY album) AS max_energy,
    MIN(ROUND(energy_liveness::NUMERIC, 2)) OVER (PARTITION BY album) AS min_energy,
    ROUND(AVG(energy_liveness::NUMERIC)     OVER (PARTITION BY album), 2) AS avg_energy
FROM spotify
ORDER BY avg_energy DESC;
```

**14. Tracks where energy-to-liveness ratio > 1.2**
```sql
WITH cte AS (
    SELECT 
        track,
        ROUND(100 * (energy_liveness::NUMERIC / liveness)) AS energy_liveness_ratio
    FROM spotify
)
SELECT *
FROM cte
WHERE energy_liveness_ratio > 1.2;
```
**15. Cumulative sum of likes ordered by views (Window Function)**
```sql
SELECT
    track,
    total_views,
    total_likes,
    SUM(total_likes) OVER (
        ORDER BY total_views DESC
    ) AS cumulative_likes
FROM (
    SELECT
        track,
        SUM(views) AS total_views,
        SUM(likes) AS total_likes
    FROM spotify
    GROUP BY track
) t
ORDER BY total_views DESC;
---

## ⚡ Query Optimization

### Before Optimization — Using `EXPLAIN`

Initial query performance on the `artist` column (sequential scan):

| Metric | Value |
|--------|-------|
| Execution Time | ~7 ms |
| Planning Time  | ~0.17 ms |

### Index Creation

```sql
CREATE INDEX idx_artist ON spotify(artist);
```

### After Optimization

| Metric | Value |
|--------|-------|
| Execution Time | ~0.153 ms |
| Planning Time  | ~0.152 ms |

> ✅ **Result:** ~97% reduction in execution time after indexing — demonstrating the dramatic performance gains possible with proper index strategy.

---

## 🛠️ Technology Stack

| Tool | Purpose |
|------|---------|
| **PostgreSQL** | Primary database engine |
| **pgAdmin 4** | SQL editor and query runner |
| **SQL** | DDL, DML, Aggregations, Joins, Subqueries, Window Functions |
---
---

*Built with ❤️ using PostgreSQL and real-world Spotify data.*
