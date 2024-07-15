# CHALLENGE

# [SQL Zoo Question 13](https://sqlzoo.net/wiki/The_JOIN_operation)

List every match with the goals scored by each team as shown. This will use ["CASE WHEN"](https://sqlzoo.net/wiki/CASE) which has not been explained in any previous exercises.

Notice in the query given every goal is listed. If it was a team1 goal then a 1 appears in score1, otherwise there is a 0. You could SUM this column to get a count of the goals scored by team1. Sort your result by mdate, matchid, team1 and team2.

## Understand `CASE WHEN`

The SQL expression `CASE` is a generic **conditional expression**, similar to if/else satements in other programming languages:

`CASE` allows you to return different values under different conditions. CASE clauses can be used wherever an expression is valid. Each `condition` is an expression that returns a boolean result. 

If the `condition`'s result is true, the value of the CASE expression is the result that follows the `condition`, and the remainder of the CASE expression is not processed. If the `condition`'s result is not true, any subsequent WHEN clauses are examined in the same manner. 

If no `WHEN` `condition` yields true, the value of the CASE expression is the result of the ELSE clause. If the ELSE clause is omitted and no `condition` is true, the result is null.

```sql

CASE 
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE resultN
END

```

## Understand the problem 

- To list every match with the goals scored by each team using the `CASE WHEN` statement.
- Then, sum the columns to get the count of goals scored by team1 and team2.

```sql


SELECT
    g.mdate,
    g.team1,
    SUM(CASE WHEN go.teamid = g.team1 THEN 1 ELSE 0 END) AS score1,
    g.team2,
    SUM(CASE WHEN go.teamid = g.team2 THEN 1 ELSE 0 END) AS score2
FROM
    game g
LEFT JOIN
    goal go ON g.id = go.matchid
GROUP BY
    g.mdate, g.id, g.team1, g.team2
ORDER BY
    g.mdate, g.id, g.team1, g.team2;
```

## Understand the solution

1. Use a `LEFT JOIN` to join the game table with the goal table on the match ID. Why use `LEFT JOIN` in the query?
   
The purpost of using `LEFT JOIN` is to ensure that all matches from the `game` table are inclded in the result, even if no goals were scored in that match. If an `INNER JOIN` was used, only matches with at least one goal would be included in the results. 

By including matches with no goals, we can accurately sum the goals for each team. Matches with no goals will have a count of zero, which is necessary for a complete and accurate report. 

2. Use `CASE WHEN` to determine if a goal was scored by team1 or team2.

3. Use the `SUM` function to aggregate the goals for each team.

`SUM(CASE WHEN go.teamid = g.team1 THEN 1 ELSE 0 END)` is a common SQL pattern used to conditionally count or sum rows based on specific criteria.

 - `WHEN go.teamid = g.team1`: This condition checks if the teamid in the goal table (`go`) matches the team1 column in the game table (`g`).
 - `THEN 1`: If the condition is true (i.e., the `teamid` matches `team1`), the CASE statement returns `1`.
 - `ELSE 0`: If the condition is false (i.e., the `teamid` does not match `team1`), the CASE statement returns `0`.

 The `SUM` function adds up all the values returned by the `CASE` statement for each row in the result set.
 - This `SUM` function iterates over each row in the result set and evaluates the `CASE` statement.
 - For each row where the condition `go.teamid = g.team1` is true, it adds `1` to the sum.
 - For each row where the condition is false, it adds `0` to the sum.
 - The result is the total number of rows where `go.teamid matches g.team1`.

4. Group by the match details to get the total goals for each match.

# [SQL Zoo Question 8](https://sqlzoo.net/wiki/More_JOIN_operations)

List the films in which 'Harrison Ford' has appeared. 

## Understand the problem 

1. Identify the relevant tables:
   
    - `movie` contains the `title` of the films. 
    - `actor` contains the `name` of the actor.
    - `casting` connects movies to actors with `movieid`, `actorid`

2. Understand the Relationships:

    - Each movie can have multiple actors (one-to-many relationship between movie and casting).
    - Each actor can appear in multiple movies (one-to-many relationship between actor and casting).

3. Steps to Achieve the Goal:

    - Find the actor ID for 'Harrison Ford' in the actor table, connects to casting table.
    - Use the casting table to find all movieid entries associated with 'Harrison Ford's actor ID.
    - Use the movieid to look up the movie details in the movie table.
  
# [SQL ZOO Question 12](https://sqlzoo.net/wiki/More_JOIN_operations)

List the film title and the leading actor for all of the films 'Julie Andrews' played in. 

```sql
-- a list of movie id where Julie Andres played in
SELECT movieid FROM casting
WHERE actorid IN (
    -- give us 'Julie Andrews' actor id
  SELECT id FROM actor
  WHERE name='Julie Andrews')

-- get all actors
SELECT title, actorid FROM movie m
JOIN casting c ON m.id = movieid
WHERE id IN (SELECT movieid FROM casting
 WHERE actorid = (SELECT id from actor
  WHERE name = 'Julie Andrews'));

-- `ord=1` focus on the leading actor
SELECT title, actorid, ord FROM movie m
JOIN casting c ON m.id = movieid AND ord=1
WHERE id IN (SELECT movieid FROM casting
 WHERE actorid = (SELECT id from actor
  WHERE name = 'Julie Andrews'));

-- Join the `actor` table to see the name of leading actor
SELECT title, name FROM movie m
JOIN casting c ON m.id = movieid AND ord=1
JOIN actor a ON a.id = actorid 
    WHERE m.id IN (SELECT movieid FROM casting
        WHERE actorid = (SELECT id from actor
            WHERE name = 'Julie Andrews'));

-- Alternative
SELECT 
    m.title, 
    a.name 
FROM 
    movie m
JOIN 
    casting c ON m.id = c.movieid AND c.ord = 1
JOIN 
    actor a ON a.id = c.actorid
JOIN 
    casting c2 ON m.id = c2.movieid
JOIN 
    actor a2 ON c2.actorid = a2.id
WHERE 
    a2.name = 'Julie Andrews';

```

What is wrong with the following:

SELECT name FROM casting
JOIN actor ON actorid=actor.id
GROUP BY actorid HAVING ord=1 AND COUNT(actorid)>14
ORDER BY name

1. Incorrect HAVING Clause: The HAVING clause is used to filter groups after the GROUP BY operation, but the condition ord=1 should be part of the WHERE clause, not the HAVING clause. 
   
2. The HAVING clause should only have aggregate functions.
Missing Grouping Column in Select Clause: The SELECT clause should include the column that is being grouped, which in this case is actor.name.

3. Grouping on Non-Grouped Column: You need to group by the actor's name as well to avoid issues since you're selecting it.


# [SQL ZOO Question 15](https://sqlzoo.net/wiki/More_JOIN_operations)

List all the people who have worked with 'Art Garfunkel'. 

SELECT DISTINCT name FROM casting c
JOIN actor a ON a.id = c.actorid
WHERE movieid IN (
 SELECT movieid FROM casting c 
  WHERE actorid = (SELECT id FROM actor
   WHERE name = 'Art Garfunkel'))
AND name != 'Art Garfunkel'