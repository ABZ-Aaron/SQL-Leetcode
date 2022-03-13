# SQL-Leetcode

A repository where I'll store my solutions for SQL Leetcode Problems. I know basic SQL, but still finding myself struggling with thinking in terms of sets, and don't feel my SQL skills are particularly strong. By working through Leetcode problems, I hope to amend this.

I will post my first working solution. Then have a look in Leetcode's discussion section to see what the best, or most popular answers are, and try work out why it's better than my solution.

The DBMS I'll be working with is MYSQL.

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

### MY SOLUTION

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

### MY SOLUTION

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

This example is better, although maybe it could have used `BETWEEN`. It's more simple than my solution, simply dividing all distinct Session IDs and dividing by all distinct User IDs. 

I have a strange habit of over complicating things.

## QUESTION 1809

```+----------------+----------+
| Column Name    | Type     |
+----------------+----------+
| user_id        | int      |
| time_stamp     | datetime |
+----------------+----------+
```

Write an SQL query to report the latest login for all users in the year 2020. Do not include the users who did not login in 2020. Return the result table in any order.

### MY SOLUTION

```SQL
SELECT user_id,
       MAX(time_stamp) last_stamp
FROM Logins
WHERE YEAR(time_stamp) = '2020'
GROUP BY user_id;
```

## QUESTION 1083

```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| product_id   | int     |
| product_name | varchar |
| unit_price   | int     |
+--------------+---------+
```
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| seller_id   | int     |
| product_id  | int     |
| buyer_id    | int     |
| sale_date   | date    |
| quantity    | int     |
| price       | int     |
+-------------+---------+
```

Write an SQL query that reports the buyers who have bought S8 but not iPhone. Note that S8 and iPhone are products present in the Product table.



```SQL
SELECT DISTINCT buyer_id 
FROM Sales 
WHERE
    buyer_id IN (
        SELECT buyer_id
        FROM Sales s
        JOIN Product p ON p.product_id = s.product_id
        WHERE product_name = 'S8') AND
    buyer_id NOT IN (
        SELECT buyer_id
        FROM Sales s
        JOIN Product p ON p.product_id = s.product_id
        WHERE product_name = 'iPhone');
```

Alternative, more efficient solution:

```SQL
SELECT buyer_id
FROM Sales s
JOIN Product p ON p.product_id = s.product_id
GROUP BY buyer_id
HAVING SUM(product_name = 'S8') > 0 AND SUM(product_name = 'iPhone') = 0;
```

## QUESTION 1495

### MY SOLUTION

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| program_date  | date    |
| content_id    | int     |
| channel       | varchar |
+---------------+---------+
```
```
+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| content_id       | varchar |
| title            | varchar |
| Kids_content     | enum    |
| content_type     | varchar |
+------------------+---------+
```

Write an SQL query to report the distinct titles of the kid-friendly movies streamed in June 2020.

```SQL
SELECT title FROM Content 
WHERE content_type = 'Movies' AND Kids_content = 'Y' AND content_id IN (
SELECT content_id FROM TVProgram
WHERE MONTH(program_date) = '6');
```

Alternative. This joins the tables rather than uses a subquery. It also makes sure the year is correct. Something I missed.

```SQL
SELECT DISTINCT c.title
FROM Content c
JOIN TVProgram p
    ON c.content_id = p.content_id
WHERE c.Kids_content = 'Y'
    AND c.content_type = 'Movies'
    AND MONTH(p.program_date) = 6
    AND YEAR(p.program_date) = 2020;
```

## QUESTION 175

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| personId    | int     |
| lastName    | varchar |
| firstName   | varchar |
+-------------+---------+
```
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| addressId   | int     |
| personId    | int     |
| city        | varchar |
| state       | varchar |
+-------------+---------+
```

Write an SQL query to report the first name, last name, city, and state of each person in the Person table. If the address of a personId is not present in the Address table, report null instead.

### MY SOLUTION

```SQL
SELECT firstName, lastName, IFNULL(city, null) city, IFNULL(state, null) state
FROM Person p 
LEFT JOIN Address a ON a.personId = p.personId
```

The `IFNULL` isn't strictly required, and will actually slow our query down.

## QUESTION 181

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| salary      | int     |
| managerId   | int     |
+-------------+---------+
```

Write an SQL query to find the employees who earn more than their managers.

### MY SOLUTION

```SQL
SELECT em.name Employee
FROM Employee em
JOIN Employee ma ON ma.id = em.managerId
WHERE ma.salary < em.salary;
```

## QUESTION 182

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| email       | varchar |
+-------------+---------+
```
Write an SQL query to report all the duplicate emails.

### MY SOLUTION

```SQL
SELECT email Email
FROM Person
GROUP BY email
HAVING COUNT(email) > 1
```

## QUESTION 183

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
```
 ```
+-------------+------+
| Column Name | Type |
+-------------+------+
| id          | int  |
| customerId  | int  |
+-------------+------+
```

Write an SQL query to report all customers who never order anything.

### MY SOLUTION

```SQL
SELECT name Customers
FROM Customers
WHERE id NOT IN (SELECT customerId FROM Orders);
```

## QUESTION 196

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| email       | varchar |
+-------------+---------+
```

Write an SQL query to delete all the duplicate emails, keeping only one unique email with the smallest id. Note that you are supposed to write a DELETE statement and not a SELECT one.

### MY SOLUTION

```SQL
DELETE FROM Person WHERE id NOT IN (
    SELECT * FROM (
        SELECT MIN(id)
        FROM Person
        GROUP BY email) myalias);
```

Surprisingly, this performed well. The logic is that we are grouping by email and selecting the smallest ID for those groups. We then delete any records where ID is not present in that output. The reason we have to do `SELECT * FROM (SELECT...)` is because in MYSQL we can't delete the table we are querying. So we have to query the table within the query.


## QUESTION 620

Table: Cinema
```
+----------------+----------+
| Column Name    | Type     |
+----------------+----------+
| id             | int      |
| movie          | varchar  |
| description    | varchar  |
| rating         | float    |
+----------------+----------+
```
id is the primary key for this table.
Each row contains information about the name of a movie, its genre, and its rating.
rating is a 2 decimal places float in the range [0, 10]
 

Write an SQL query to report the movies with an odd-numbered ID and a description that is not "boring".

Return the result table ordered by rating in descending order.

The query result format is in the following example.

### MY SOLUTION

```SQL
SELECT * 
FROM Cinema
WHERE description != 'boring' AND
      MOD(id, 2) != 0
ORDER BY rating DESC;
```

## QUESTION 577

Table: Employee
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| empId       | int     |
| name        | varchar |
| supervisor  | int     |
| salary      | int     |
+-------------+---------+
```
empId is the primary key column for this table.
Each row of this table indicates the name and the ID of an employee in addition to their salary and the id of their manager.
 

Table: Bonus
```
+-------------+------+
| Column Name | Type |
+-------------+------+
| empId       | int  |
| bonus       | int  |
+-------------+------+
```
empId is the primary key column for this table.
empId is a foreign key to empId from the Employee table.
Each row of this table contains the id of an employee and their respective bonus.
 
Write an SQL query to report the name and bonus amount of each employee with a bonus less than 1000.

Return the result table in any order.

The query result format is in the following example.

## MY SOLUTION

```SQL
SELECT name, bonus
FROM Employee e
LEFT JOIN Bonus b ON b.empId = e.empId
WHERE bonus IS NULL OR bonus < 1000;
```

## QUESTION 595

Table: World
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| name        | varchar |
| continent   | varchar |
| area        | int     |
| population  | int     |
| gdp         | int     |
+-------------+---------+
```
name is the primary key column for this table.
Each row of this table gives information about the name of a country, the continent to which it belongs, its area, the population, and its GDP value.
 

A country is big if:

it has an area of at least three million (i.e., 3000000 km2), or
it has a population of at least twenty-five million (i.e., 25000000).
Write an SQL query to report the name, population, and area of the big countries.

Return the result table in any order.

The query result format is in the following example.

### MY SOLUTION

```SQL
SELECT name, population, area
FROM World
WHERE area >= 3000000 OR population >= 25000000;
```
## QUESTION 2072

Table: NewYork
```
+-------------+------+
| Column Name | Type |
+-------------+------+
| student_id  | int  |
| score       | int  |
+-------------+------+
```
student_id is the primary key for this table.
Each row contains information about the score of one student from New York University in an exam.
 

Table: California
```
+-------------+------+
| Column Name | Type |
+-------------+------+
| student_id  | int  |
| score       | int  |
+-------------+------+
```
student_id is the primary key for this table.
Each row contains information about the score of one student from California University in an exam.
 

There is a competition between New York University and California University. The competition is held between the same number of students from both universities. The university that has more excellent students wins the competition. If the two universities have the same number of excellent students, the competition ends in a draw.

An excellent student is a student that scored 90% or more in the exam.

Write an SQL query to report:

"New York University" if New York University wins the competition.
"California University" if California University wins the competition.
"No Winner" if the competition ends in a draw.

### MY SOLUTION

```SQL
```
## QUESTION

### MY SOLUTION

```SQL
```
## QUESTION

### MY SOLUTION

```SQL
```
## QUESTION

### MY SOLUTION

```SQL
```
## QUESTION

### MY SOLUTION

```SQL
```
## QUESTION

### MY SOLUTION

```SQL
```
## QUESTION

### MY SOLUTION

```SQL
```
## QUESTION

### MY SOLUTION

```SQL
```
## QUESTION

### MY SOLUTION

```SQL
```
## QUESTION

### MY SOLUTION

```SQL
```
## QUESTION

### MY SOLUTION

```SQL
```
