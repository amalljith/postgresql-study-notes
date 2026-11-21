# PostgreSQL — Lesson 05: Subqueries

Beginner-friendly reference | SQL Shell (psql) | English

## 1. Introduction

A **Subquery** is a query written inside another SQL query.

- Inner Query: The query inside another query.
- Outer Query: The main query that contains the subquery.

Example:

```sql
SELECT *
FROM students
WHERE marks = (
    SELECT MAX(marks)
    FROM students
    WHERE course = 'Python'
);
```

The inner query finds the highest marks in Python. The outer query returns the student details matching those marks.

## 2. Subquery with Comparison Operators

A subquery can be used with comparison operators such as:

- `=`
- `>`
- `<`
- `>=`
- `<=`
- `<>`

Example: Find students whose marks are greater than the overall average.

```sql
SELECT *
FROM students
WHERE marks > (
    SELECT AVG(marks)
    FROM students
);
```

Example: Find React students whose marks are below the React course average.

```sql
SELECT *
FROM students
WHERE course = 'React'
AND marks < (
    SELECT AVG(marks)
    FROM students
    WHERE course = 'React'
);
```

## 3. Subquery with IN and NOT IN

### IN

`IN` checks whether a value matches any value returned by a subquery.

```sql
SELECT *
FROM students
WHERE course IN (
    SELECT course
    FROM students
    WHERE marks > 90
);
```

This returns all students belonging to courses that have at least one student scoring above 90.

### NOT IN

`NOT IN` returns rows whose values do not match any value returned by the subquery.

```sql
SELECT *
FROM students
WHERE course NOT IN (
    SELECT course
    FROM students
    WHERE marks > 90
);
```

**Important:** `NOT IN` can behave unexpectedly when the subquery returns NULL. Filter NULL values when necessary.

## 4. Subquery with ANY

`ANY` returns TRUE if at least one comparison is TRUE.

Example:

```sql
SELECT *
FROM students
WHERE marks > ANY (
    SELECT marks
    FROM students
    WHERE course = 'React'
      AND marks IS NOT NULL
);
```

Meaning: Find students whose marks are greater than at least one non-NULL React mark.

## 5. Subquery with ALL

`ALL` returns TRUE only if every comparison is TRUE.

Example:

```sql
SELECT *
FROM students
WHERE marks > ALL (
    SELECT marks
    FROM students
    WHERE course = 'React'
      AND marks IS NOT NULL
);
```

Meaning: Find students whose marks are greater than every non-NULL React mark.

### ANY vs ALL

| ANY                                   | ALL                                    |
| ------------------------------------- | -------------------------------------- |
| At least one comparison must be TRUE. | Every comparison must be TRUE.         |
| One matching value is sufficient.     | All values must satisfy the condition. |

**NULL Note:** NULL comparisons produce UNKNOWN. When using ANY or ALL with nullable columns, filter NULL values if they should not affect the comparison.

## 6. EXISTS and NOT EXISTS

`EXISTS` checks whether a subquery returns at least one row.

Example:

```sql
SELECT *
FROM students
WHERE EXISTS (
    SELECT 1
    FROM students
    WHERE marks > 90
);
```

If the inner query returns one or more rows, EXISTS is TRUE.

If it returns zero rows, EXISTS is FALSE.

`SELECT 1` is commonly used because EXISTS checks whether rows exist, not what values they contain.

### EXISTS with Correlated Subquery

```sql
SELECT *
FROM students s
WHERE EXISTS (
    SELECT 1
    FROM students s2
    WHERE s2.course = s.course
      AND s2.marks > 90
);
```

This returns students from courses that have at least one student scoring above 90.

`NOT EXISTS` checks that the subquery returns no rows.

## 7. Correlated Subquery

A **Correlated Subquery** is a subquery that references a column from the outer query.

Example:

```sql
SELECT *
FROM students s
WHERE marks > (
    SELECT AVG(marks)
    FROM students
    WHERE course = s.course
);
```

Explanation:

- `s` is the alias of the outer students table.
- `s.course` refers to the current student's course.
- The inner query calculates the average marks for that course.
- The outer query checks whether the student's marks are greater than that average.

### Normal vs Correlated Subquery

| Normal Subquery                     | Correlated Subquery                            |
| ----------------------------------- | ---------------------------------------------- |
| Does not depend on the outer query. | References the outer query.                    |
| Can run independently.              | Depends on the current outer row.              |
| Example: Overall average.           | Example: Course-wise average for each student. |

## 8. Subquery in FROM Clause (Derived Table)

A subquery inside the FROM clause is called a **Derived Table**.

It behaves like a temporary result table within the query.

Example:

```sql
SELECT *
FROM (
    SELECT course, AVG(marks) AS average_marks
    FROM students
    GROUP BY course
) AS course_summary;
```

The inner query calculates the average marks for each course. The outer query reads the resulting rows.

Example: Find courses with an average above 80.

```sql
SELECT *
FROM (
    SELECT course, AVG(marks) AS average_marks
    FROM students
    GROUP BY course
) AS course_summary
WHERE average_marks > 80;
```

**Important:** A subquery in the FROM clause must have an alias.

## 9. Subquery in SELECT Clause

A subquery can also be used inside the SELECT list.

Example: Display each student's details with the overall average.

```sql
SELECT
    name,
    course,
    marks,
    (SELECT AVG(marks) FROM students) AS overall_average
FROM students;
```

The overall average appears alongside every student.

### Correlated Scalar Subquery in SELECT

```sql
SELECT
    name,
    course,
    marks,
    (
        SELECT ROUND(AVG(marks), 2)
        FROM students s2
        WHERE s2.course = s1.course
    ) AS course_average
FROM students s1;
```

This displays each student's course average alongside their details.

## 10. Scalar Subquery

A **Scalar Subquery** returns a single value (one row and one column).

Example:

```sql
SELECT AVG(marks)
FROM students;
```

This returns one average value.

A scalar subquery can be used in SELECT, WHERE, or other expressions where a single value is expected.

**Important:** If a scalar subquery returns more than one row, PostgreSQL raises an error.

## 11. Subquery with UPDATE

A subquery can be used in an UPDATE condition.

Example:

```sql
UPDATE students
SET marks = 50
WHERE marks < (
    SELECT AVG(marks)
    FROM students
)
AND marks IS NOT NULL;
```

This updates students whose marks are below the overall average.

### Safe Practice Using Transactions

```sql
BEGIN;

UPDATE students
SET marks = 50
WHERE marks < (
    SELECT AVG(marks)
    FROM students
)
AND marks IS NOT NULL;

SELECT * FROM students;

ROLLBACK;
```

- `BEGIN` starts a transaction.
- `UPDATE` temporarily changes the matching rows.
- `SELECT` checks the changes.
- `ROLLBACK` cancels the changes.

## 12. Subquery with DELETE

A subquery can be used to identify rows for deletion.

Example:

```sql
DELETE FROM students
WHERE marks < (
    SELECT AVG(marks)
    FROM students
)
AND marks IS NOT NULL;
```

### Safe Practice Using Transactions

```sql
BEGIN;

DELETE FROM students
WHERE marks < (
    SELECT AVG(marks)
    FROM students
)
AND marks IS NOT NULL;

SELECT * FROM students;

ROLLBACK;
```

`ROLLBACK` restores the deleted rows because the deletion was inside an uncommitted transaction.

**Important:** Always verify the WHERE condition before running DELETE. Use transactions when practising destructive operations.

## 13. Subquery with INSERT

A subquery can insert the result of a SELECT query into another table.

Example:

```sql
INSERT INTO high_performers (id, name, course, marks)
SELECT id, name, course, marks
FROM students
WHERE marks > 85;
```

This copies students scoring above 85 from the students table into high_performers.

Verify the inserted rows:

```sql
SELECT * FROM high_performers;
```

Count the inserted rows:

```sql
SELECT COUNT(*)
FROM high_performers;
```

**Difference:**

- `INSERT ... VALUES` inserts explicitly provided values.
- `INSERT ... SELECT` inserts rows returned by another query.

## 14. Subquery with Multiple Columns

A subquery can return multiple columns when used with a matching row comparison.

Example: Find the student with the highest marks in each course.

```sql
SELECT *
FROM students
WHERE (course, marks) IN (
    SELECT course, MAX(marks)
    FROM students
    GROUP BY course
);
```

Here, `(course, marks)` is compared with the course and maximum marks pairs returned by the inner query.

## 15. Important Notes

- A subquery is enclosed in parentheses.
- A subquery can be used in SELECT, FROM, WHERE, INSERT, UPDATE, and DELETE.
- A scalar subquery returns one value.
- A subquery used with IN, ANY, or ALL can return multiple values.
- EXISTS checks whether at least one row exists.
- Correlated subqueries reference columns from the outer query.
- A FROM subquery requires an alias.
- NULL values can affect comparisons, especially with NOT IN, ANY, and ALL.
- Use transactions to safely practise UPDATE and DELETE.

## 16. Common Mistakes

| Mistake                                                   | Correction                                       |
| --------------------------------------------------------- | ------------------------------------------------ |
| Using `=` when the subquery returns multiple rows         | Use `IN`, `ANY`, or `ALL` when appropriate.      |
| Forgetting the alias for a FROM subquery                  | Add an alias after the closing parenthesis.      |
| Confusing EXISTS with returned values                     | EXISTS checks only whether rows exist.           |
| Forgetting the outer table alias in a correlated subquery | Reference the correct alias, such as `s.course`. |
| Ignoring NULL values                                      | Use `IS NOT NULL` when necessary.                |
| Running UPDATE or DELETE without checking the condition   | Test with SELECT and use a transaction.          |

## 17. Quick Reference

| Concept              | Purpose                         |
| -------------------- | ------------------------------- |
| Comparison Subquery  | Compare with a single value     |
| IN                   | Match any returned value        |
| NOT IN               | Exclude returned values         |
| ANY                  | At least one comparison is TRUE |
| ALL                  | Every comparison is TRUE        |
| EXISTS               | Check whether rows exist        |
| NOT EXISTS           | Check whether no rows exist     |
| Correlated Subquery  | Use outer query values          |
| Derived Table        | Subquery inside FROM            |
| Scalar Subquery      | Return one value                |
| INSERT ... SELECT    | Insert rows from another query  |
| UPDATE with Subquery | Update rows based on a subquery |
| DELETE with Subquery | Delete rows based on a subquery |

## 18. Practice Exercises

1. Find students whose marks are greater than the overall average.
2. Find students belonging to courses that have at least one student scoring above 90.
3. Find students whose marks are greater than every React student's non-NULL marks.
4. Find students whose marks are above their own course average.
5. Display each course with its average using a derived table.
6. Find the student with the highest marks in each course.
7. Display each student alongside their course average.
8. Insert students scoring above 85 into the high_performers table.

## 19. Final Quiz Review

Topics to revise:

- Normal and correlated subqueries
- Comparison operators
- IN and NOT IN
- ANY and ALL
- EXISTS and NOT EXISTS
- Scalar subqueries
- Derived tables
- Subqueries in SELECT
- INSERT, UPDATE, and DELETE with subqueries
- Multiple-column subqueries

## Next Lesson

**Lesson 06 — PostgreSQL Constraints**

Topics:

- PRIMARY KEY
- FOREIGN KEY
- UNIQUE
- NOT NULL
- CHECK
- DEFAULT
