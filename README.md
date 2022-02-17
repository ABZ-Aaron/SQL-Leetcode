# SQL-Leetcode

A repository where I'll store my solutions for SQL Leetcode Problems. I know basic SQL, but still finding myself struggling with thinking in terms of sets, and don't feel my SQL skills are particularly strong. By working through Leetcode problems, I hope to amend this.

I will post my first working solution. Then have a look in Leetcode's discussion section to see what the best, or most popular answers are, and try work out why it's better than my solution.

## QUESTION 1303

```
+---------------+---------+
#| Column Name   | Type    |
#+---------------+---------+
#| employee_id   | int     |
#| team_id       | int     |
#+---------------+---------+
```

employee_id is the primary key for this table. Each row of this table contains the ID of each employee and their respective team. Write an SQL query to find the team size of each of the employees. Return result table in any order.

### My Solution

```SQL
SELECT em.employee_id,
       sub.total team_size
FROM Employee em
JOIN (
  SELECT team_id, COUNT(*) total
  FROM Employee
  GROUP by team_id) sub ON sub.team_id = em.team_id;
```
Not the greatest solution. Here's a more efficient way using a Window Function:

```SQL
SELECT employee_id, 
    COUNT(team_id) OVER(PARTITION BY team_id) AS team_size
FROM Employee
```
Here we are partitioning by Team ID, then working out the count. We know it's a window function due to the use of `OVER`.

## QUESTION 1142

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| session_id    | int     |
| activity_date | date    |
| activity_type | enum    |
+---------------+---------+
```

Write an SQL query to find the average number of sessions per user for a period of 30 days ending 2019-07-27 inclusively, rounded to 2 decimal places. The sessions we want to count for a user are those with at least one activity in that time period.

### My Solution

```SQL
WITH myquery AS 
       (
       SELECT user_id, 
              session_id
       FROM Activity
       WHERE activity_date BETWEEN '2019-06-28' AND '2019-07-27'
       GROUP BY user_id, session_id
       )
SELECT 
    IFNULL(ROUND(COUNT(*) / COUNT(DISTINCT user_id), 2), 0.00) average_sessions_per_user
FROM myquery;
```

Again, not the best solution.

Here's a better example:

```SQL
SELECT ifnull(ROUND(COUNT(DISTINCT session_id) / COUNT(DISTINCT user_id), 2), 0.00) 
AS average_sessions_per_user
FROM Activity 
WHERE activity_date >= '2019-06-28' and activity_date <= '2019-07-27'; 
```

This example is better, although maybe it could have used `BETWEEN`. It's more simple than my solution, simply dividng all distinct Session IDs and dividing by all distinct User IDs. 

I have a strange habit of overcomplicating things.
