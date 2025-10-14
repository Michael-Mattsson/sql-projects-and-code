## HackerRank - 15 Days of Learning SQL

##### https://www.hackerrank.com/challenges/15-days-of-learning-sql/problem?isFullScreen=true

Julia conducted a 15 Days of Learning SQL contest.  
The start date of the contest was **March 01, 2016** and the end date was **March 15, 2016**.  

Write a query to print:  

1. Total number of unique hackers who made at least 1 submission each day (starting on the first day of the contest).  
2. The `hacker_id` and `name` of the hacker who made the maximum number of submissions each day.  

> If more than one hacker has the maximum submissions, print the **lowest hacker_id**.  
> The query should print this information for each day of the contest, sorted by date.

---

## Tables

**Hackers**  
- `hacker_id` (INT)  
- `name` (STRING)

**Submissions**  
- `submission_date` (DATE)  
- `submission_id` (INT)  
- `hacker_id` (INT)  
- `score` (INT)

---

## Solution

*Decided to use CTEs to break down the problem into multiple steps for clarity and ease of reading. Steps include:*  

1. Tracking each hacker's submission streak.  
2. Counting the number of hackers that submitted every day.  
3. Ranking hackers by most submissions per day.  
4. Combining everything for the final output.  

```sql
WITH submission_tracking AS (
    SELECT
        s1.submission_date,
        s1.hacker_id,
        (
            SELECT COUNT(DISTINCT s.submission_date)
            FROM Submissions s
            WHERE s.hacker_id = s1.hacker_id
              AND s.submission_date <= s1.submission_date
        ) AS submission_streak
    FROM (
        SELECT DISTINCT submission_date, hacker_id
        FROM Submissions
    ) s1
),

everyday_hacker_count AS (
    SELECT
        s1.submission_date,
        COUNT(*) AS everyday_hacker
    FROM submission_tracking s1
    WHERE s1.submission_streak = (
        SELECT COUNT(DISTINCT s2.submission_date)
        FROM Submissions s2
        WHERE s2.submission_date <= s1.submission_date
    )
    GROUP BY s1.submission_date
),

daily_submission_rank AS (
    SELECT
        s.submission_date,
        s.hacker_id,
        h.name,
        COUNT(*) AS submission_on_day,
        RANK() OVER (
            PARTITION BY s.submission_date 
            ORDER BY COUNT(*) DESC, s.hacker_id ASC
        ) AS daily_rank
    FROM Submissions s
    JOIN Hackers h ON s.hacker_id = h.hacker_id
    GROUP BY s.submission_date, s.hacker_id, h.name
)

SELECT
    dsr.submission_date,
    ehc.everyday_hacker,
    dsr.hacker_id,
    dsr.name
FROM daily_submission_rank dsr
JOIN everyday_hacker_count ehc 
    ON dsr.submission_date = ehc.submission_date
WHERE dsr.daily_rank = 1
ORDER BY dsr.submission_date, dsr.hacker_id ASC;
