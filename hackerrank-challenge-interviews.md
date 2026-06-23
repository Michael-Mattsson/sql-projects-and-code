## HackerRank - Interviews

[Problem Link](https://www.hackerrank.com/challenges/interviews/problem?isFullScreen=true)

Samantha interviews many candidates from different colleges using coding challenges and contests.  
Write a query to print the `contest_id`, `hacker_id`, `name`, and the sums of `total_submissions`, `total_accepted_submissions`, `total_views`, and `total_unique_views` for each contest, sorted by `contest_id`.  

> Exclude the contest from the result if all four sums are 0.

---

## Tables

**Contests**  
- `contest_id` (INT)  
- `hacker_id` (INT)  
- `name` (STRING)

**Colleges**  
- `college_id` (INT)  
- `contest_id` (INT)

**Challenges**  
- `challenge_id` (INT)  
- `college_id` (INT)

**View_Stats**  
- `challenge_id` (INT)  
- `total_views` (INT)  
- `total_unique_views` (INT)

**Submission_Stats**  
- `challenge_id` (INT)  
- `total_submissions` (INT)  
- `total_accepted_submissions` (INT)

---

## Solution

*Used CTEs to break down the problem into steps for clarity:*

1. **`view_stats_summary`**: Summarizes total views and total unique views for each challenge.  
2. **`submission_stats_summary`**: Summarizes total submissions and total accepted submissions for each challenge.  

> **Note:** The solution requires multiple joins because data is spread across different tables.  
> Be careful: use `LEFT JOIN` for views and submissions so that challenges with 0 values are still included. Avoid using only `INNER JOIN`.

```sql
WITH view_stats_summary AS (
    SELECT
        challenge_id,
        SUM(total_views) AS t_views,
        SUM(total_unique_views) AS u_views
    FROM View_Stats
    GROUP BY challenge_id
),

submission_stats_summary AS (
    SELECT
        challenge_id,
        SUM(total_submissions) AS t_sub,
        SUM(total_accepted_submissions) AS t_a_sub
    FROM Submission_Stats
    GROUP BY challenge_id
)

SELECT
    c.contest_id,
    c.hacker_id,
    c.name,
    SUM(ss.t_sub) AS total_submissions,
    SUM(ss.t_a_sub) AS total_accepted_submissions,
    SUM(vs.t_views) AS total_views,
    SUM(vs.u_views) AS total_unique_views
FROM Contests c
JOIN Colleges col 
    ON c.contest_id = col.contest_id
JOIN Challenges ch 
    ON col.college_id = ch.college_id
LEFT JOIN view_stats_summary vs 
    ON ch.challenge_id = vs.challenge_id
LEFT JOIN submission_stats_summary ss 
    ON ch.challenge_id = ss.challenge_id
GROUP BY 
    c.contest_id,
    c.hacker_id,
    c.name
HAVING 
    (SUM(ss.t_sub) +
     SUM(ss.t_a_sub) +
     SUM(vs.t_views) +
     SUM(vs.u_views)) > 0
ORDER BY 
    c.contest_id;
