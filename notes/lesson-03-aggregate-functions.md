# PostgreSQL — Lesson 03: Aggregate Functions

**Beginner-friendly reference | SQL Shell (psql) | English**

## 1. Introduction

Aggregate functions summarize many rows into calculated results.

| Function | Purpose |
|---|---|
| `COUNT()` | Count rows or non-NULL values |
| `SUM()` | Calculate the total |
| `AVG()` | Calculate the average |
| `MIN()` | Find the smallest value |
| `MAX()` | Find the largest value |

## 2. Example Data

| name | course | marks |
|---|---|---:|
| Arun | React | 85 |
| Rahul | Python | 72 |
| Anu | Django | 90 |
| Kiran | Django | 78 |
| Amal | React | 88 |

These are illustrative rows. Your actual `school_db.students` table contains more records and may produce different results.

## 3. COUNT(*)

```sql
SELECT COUNT(*) AS total_students
FROM students;

SELECT COUNT(*) AS react_students
FROM students
WHERE course = 'React';
```

The second query returns **2** for the example data. `COUNT(*)` counts all matching rows, even when some columns are NULL.

## 4. COUNT(column)

```sql
SELECT COUNT(email) AS students_with_email
FROM students;
```

`COUNT(email)` counts only non-NULL email values. If marks are `80`, `90`, `NULL`, and `70`, then `COUNT(*) = 4` but `COUNT(marks) = 3`.

## 5. COUNT(DISTINCT)

```sql
SELECT COUNT(DISTINCT course) AS unique_courses
FROM students;

SELECT COUNT(DISTINCT age) AS unique_ages
FROM students;
```

`COUNT(DISTINCT column)` counts distinct non-NULL values. The example data contains 3 unique courses: React, Python, and Django.

## 6. SUM(): Total

```sql
SELECT SUM(marks) AS total_marks
FROM students;

SELECT SUM(marks) AS react_total_marks
FROM students
WHERE course = 'React';
```

For the example data: all marks total **413**; React marks total **173**.

`SUM()` ignores NULL values. If there are no non-NULL values, it returns NULL, not zero.

## 7. AVG(): Average

```sql
SELECT AVG(marks) AS average_marks
FROM students;

SELECT AVG(marks) AS python_avg_marks
FROM students
WHERE course = 'Python';
```

Average = sum of non-NULL values / number of non-NULL values.

For marks `80`, `90`, `NULL`, `70`:

`(80 + 90 + 70) / 3 = 80`

Use `ROUND(AVG(marks), 2)` to round a numeric average to two decimal places.

## 8. MIN(): Lowest Value

```sql
SELECT MIN(marks) AS lowest_marks
FROM students;
```

Returns the smallest non-NULL mark.

## 9. MAX(): Highest Value

```sql
SELECT MAX(marks) AS highest_marks
FROM students;
```

Returns the largest non-NULL mark.

Both functions in a single query:

```sql
SELECT MIN(marks) AS lowest_marks,
       MAX(marks) AS highest_marks
FROM students
WHERE course = 'Django';
```

**Your practical result:** lowest marks = **84**, highest marks = **95**. Results vary with actual database contents.

## 10. Combining Aggregate Functions

```sql
SELECT COUNT(*) AS total_students,
       COUNT(marks) AS students_with_marks,
       SUM(marks) AS total_marks,
       AVG(marks) AS average_marks,
       MIN(marks) AS lowest_marks,
       MAX(marks) AS highest_marks
FROM students;
```

Without `GROUP BY`, this query returns one summary row.

## 11. Aggregate Functions with WHERE

```sql
SELECT COUNT(*) AS total_students,
       SUM(marks) AS total_marks,
       AVG(marks) AS average_marks
FROM students
WHERE course = 'Django';
```

The `WHERE` clause filters individual rows before the aggregate functions run.

## 12. Common Mistakes

1. Confusing `COUNT(*)` with `COUNT(column)` when NULL values exist.
2. Using `SUM()` instead of `AVG()` or vice versa.
3. Calling an average `total_marks` instead of `average_marks` in the alias.
4. Forgetting a required `WHERE` filter.
5. Expecting `SUM()` to return zero when no non-NULL values exist.
6. Assuming `COUNT(DISTINCT column)` includes NULL as a distinct value.

## 13. Debugging Tips

```sql
-- In psql: inspect columns and types
\d students

-- Inspect values before calculating
SELECT name, course, marks FROM students ORDER BY course, name;

-- Compare all rows to rows with known marks
SELECT COUNT(*) AS all_rows,
       COUNT(marks) AS rows_with_marks
FROM students;
```

Extra inserted rows, duplicate records, and NULL values can change your results.

## 14. Quick Reference

| Goal | Expression |
|---|---|
| Count all students | `COUNT(*)` |
| Count known marks | `COUNT(marks)` |
| Count unique courses | `COUNT(DISTINCT course)` |
| Total marks | `SUM(marks)` |
| Average marks | `AVG(marks)` |
| Lowest marks | `MIN(marks)` |
| Highest marks | `MAX(marks)` |

## 15. Practice Exercises

1. Count all students.
2. Count students with email addresses.
3. Count distinct ages.
4. Count React students.
5. Calculate total React marks.
6. Calculate average Python marks.
7. Find lowest and highest Django marks.
8. Compute all five aggregate functions together.

### Answer Key

```sql
-- 1
SELECT COUNT(*) AS total_students FROM students;
-- 2
SELECT COUNT(email) AS students_with_email FROM students;
-- 3
SELECT COUNT(DISTINCT age) AS unique_ages FROM students;
-- 4
SELECT COUNT(*) AS react_students FROM students WHERE course = 'React';
-- 5
SELECT SUM(marks) AS react_total_marks FROM students WHERE course = 'React';
-- 6
SELECT AVG(marks) AS python_avg_marks FROM students WHERE course = 'Python';
-- 7
SELECT MIN(marks) AS lowest_marks, MAX(marks) AS highest_marks
FROM students WHERE course = 'Django';
-- 8
SELECT COUNT(*) AS total_students, SUM(marks) AS total_marks,
       AVG(marks) AS average_marks, MIN(marks) AS lowest_marks,
       MAX(marks) AS highest_marks FROM students;
```

## 16. Final Quiz Review

**Score: 5/5.** Topics: non-NULL counting, average with NULL, `SUM()` with `WHERE`, lowest marks using `MIN()`, and `COUNT(marks)` versus `COUNT(*)`.

**Next lesson:** GROUP BY and HAVING.
