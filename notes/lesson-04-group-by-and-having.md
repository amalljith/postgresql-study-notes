# **PostgreSQL — Lesson 04: GROUP BY and HAVING**

**Beginner-friendly reference | SQL Shell (psql) | English**

## **1. Introduction**

`GROUP BY` and `HAVING` are used to organize rows into groups and filter those groups based on conditions.

- `GROUP BY` groups rows with matching values.
- `HAVING` filters groups after grouping.
- Aggregate functions such as `COUNT()`, `SUM()`, and `AVG()` are commonly used with `GROUP BY`.

## **2. Example Students Table**

These examples use a simplified version of the `school_db.students` table.

|  id | name  | course | marks |
| --: | ----- | ------ | ----: |
|   1 | Arun  | React  |    85 |
|   2 | Rahul | Python |    72 |
|   3 | Anu   | Django |    90 |
|   4 | Kiran | Django |    78 |
|   5 | Amal  | React  |    88 |

Your actual database may contain more rows and different results.

## **3. GROUP BY: Group Rows**

`GROUP BY` combines rows that have the same value in the specified column.

```sql
SELECT course
FROM students
GROUP BY course;
```

This returns each distinct course once.

To count students in each course:

```sql
SELECT course, COUNT(*) AS total_students
FROM students
GROUP BY course
ORDER BY course ASC;
```

Expected output:

| course | total_students |
| ------ | -------------: |
| Django |              2 |
| Python |              1 |
| React  |              2 |

`GROUP BY course` creates a separate group for each course. `COUNT(*)` counts the rows inside each group.

**Important:** `GROUP BY` does not delete duplicate records or automatically sort the results. Use `ORDER BY` when you need a specific order.

## **4. GROUP BY with Aggregate Functions**

Aggregate functions calculate a result for each group.

### COUNT(): Count Rows

```sql
SELECT course, COUNT(*) AS total_students
FROM students
GROUP BY course;
```

Counts students in each course.

### SUM(): Total Marks

```sql
SELECT course, SUM(marks) AS total_marks
FROM students
GROUP BY course;
```

Calculates total marks for each course.

### AVG(): Average Marks

```sql
SELECT course, ROUND(AVG(marks), 2) AS average_marks
FROM students
GROUP BY course;
```

Calculates the average marks for each course and rounds the result to two decimal places.

### MIN() and MAX(): Lowest and Highest Marks

```sql
SELECT course,
       MIN(marks) AS lowest_marks,
       MAX(marks) AS highest_marks
FROM students
GROUP BY course;
```

Finds the lowest and highest marks in each course.

## **5. COUNT(\*) vs COUNT(column) with GROUP BY**

Both functions can be used with grouped data, but they behave differently.

| Function                 | Behaviour                                   |
| ------------------------ | ------------------------------------------- |
| `COUNT(*)`               | Counts all rows in each group               |
| `COUNT(course)`          | Counts only rows where `course` is not NULL |
| `COUNT(DISTINCT course)` | Counts distinct non-NULL course values      |

Example:

```sql
SELECT course, COUNT(*) AS total_rows
FROM students
GROUP BY course;
```

`COUNT(*)` counts every row in each group, even if some other column contains NULL.

**Note:** If `course` is NULL, those rows form a separate NULL group when grouping by `course`.

## **6. WHERE with GROUP BY**

`WHERE` filters individual rows before they are grouped.

```sql
SELECT course, COUNT(*) AS total_students
FROM students
WHERE marks >= 70
GROUP BY course;
```

Only students with marks greater than or equal to 70 are included in the groups.

**Important:** `WHERE` filters rows, not groups. Aggregate functions cannot normally be used directly in a `WHERE` condition.

## **7. HAVING: Filter Groups**

`HAVING` filters groups based on a condition, usually involving aggregate functions.

Example: Find courses with more than 1 student.

```sql
SELECT course, COUNT(*) AS total_students
FROM students
GROUP BY course
HAVING COUNT(*) > 1;
```

Expected output:

| course | total_students |
| ------ | -------------: |
| Django |              2 |
| React  |              2 |

Only groups with more than one student are returned.

## **8. WHERE vs HAVING**

| WHERE                                                    | HAVING                                       |
| -------------------------------------------------------- | -------------------------------------------- |
| Filters individual rows                                  | Filters groups                               |
| Executes before GROUP BY                                 | Executes after GROUP BY                      |
| Normally used with column conditions                     | Commonly used with aggregate conditions      |
| Cannot directly use aggregate functions in its condition | Can use aggregate functions in its condition |

Example using both:

```sql
SELECT course, COUNT(*) AS total_students
FROM students
WHERE marks >= 70
GROUP BY course
HAVING COUNT(*) > 1;
```

Explanation:

1. `WHERE marks >= 70` filters individual students.
2. `GROUP BY course` groups the remaining students.
3. `HAVING COUNT(*) > 1` keeps only courses with more than one student.

## **9. HAVING with Other Aggregate Functions**

### Filter by Average Marks

```sql
SELECT course, ROUND(AVG(marks), 2) AS average_marks
FROM students
GROUP BY course
HAVING AVG(marks) > 80;
```

Returns courses whose average marks are greater than 80.

### Filter by Total Marks

```sql
SELECT course, SUM(marks) AS total_marks
FROM students
GROUP BY course
HAVING SUM(marks) > 150;
```

Returns courses whose total marks are greater than 150.

### Filter by Minimum Marks

```sql
SELECT course, MIN(marks) AS lowest_marks
FROM students
GROUP BY course
HAVING MIN(marks) > 60;
```

Returns courses where the lowest mark is greater than 60.

## **10. GROUP BY with Multiple Columns**

You can group rows using more than one column.

```sql
SELECT course, gender, COUNT(*) AS total_students
FROM students
GROUP BY course, gender;
```

This creates a separate group for each unique combination of `course` and `gender`.

For example, React-Male and React-Female are separate groups.

## **11. HAVING Without GROUP BY**

`HAVING` can also be used without an explicit `GROUP BY` clause.

In an aggregate query without `GROUP BY`, the entire input is treated as a single group.

```sql
SELECT COUNT(*) AS total_students
FROM students
HAVING COUNT(*) > 10;
```

- If the total number of rows is greater than 10, the query returns the count.
- Otherwise, it returns no rows.

## **12. SQL Logical Execution Order**

The simplified logical execution order is:

1. `FROM` — Select the source table.
2. `WHERE` — Filter individual rows.
3. `GROUP BY` — Create groups.
4. `HAVING` — Filter groups.
5. `SELECT` — Return the requested columns and calculations.
6. `ORDER BY` — Sort the result.
7. `LIMIT` — Restrict the number of returned rows.

This order helps explain why `WHERE` cannot normally filter using aggregate results, while `HAVING` can.

## **13. Common Mistakes**

1. Using `WHERE COUNT(*) > 5` instead of `HAVING COUNT(*) > 5`.
2. Forgetting `GROUP BY` when selecting a non-aggregated column alongside an aggregate.
3. Assuming `GROUP BY` automatically sorts results.
4. Confusing `COUNT(*)` with `COUNT(column)` when NULL values exist.
5. Using `HAVING` to filter individual rows instead of groups.
6. Forgetting the `FROM` clause.
7. Assuming `GROUP BY` deletes duplicate records from the table.

**Important:** When using `GROUP BY`, every selected expression must either be grouped, aggregated, or otherwise valid under PostgreSQL's grouping rules.

## **14. Quick Reference**

| Goal                         | Pattern                      |
| ---------------------------- | ---------------------------- |
| Group by course              | `GROUP BY course`            |
| Count rows per course        | `COUNT(*)` with `GROUP BY`   |
| Calculate total per course   | `SUM(marks)` with `GROUP BY` |
| Calculate average per course | `AVG(marks)` with `GROUP BY` |
| Find lowest value per group  | `MIN(marks)`                 |
| Find highest value per group | `MAX(marks)`                 |
| Filter rows before grouping  | `WHERE condition`            |
| Filter groups after grouping | `HAVING condition`           |
| Group by multiple columns    | `GROUP BY column1, column2`  |
| Sort grouped results         | `ORDER BY column`            |

## **15. Practice Exercises**

Use the `students` table.

1. Count the students in each course.
2. Calculate the total marks for each course and sort from highest to lowest.
3. Calculate the average marks for each course and round to two decimal places.
4. Find the number of students in each course whose marks are at least 70.
5. Display courses that have more than 5 students.
6. Display courses whose average marks are greater than 80.
7. Find courses with more than 3 students after filtering out students with marks below 70, and display their average marks.
8. Group students by both course and gender, and count the students in each group.

### **Answer Key**

```sql
-- 1
SELECT course, COUNT(*) AS total_students
FROM students
GROUP BY course;

-- 2
SELECT course, SUM(marks) AS total_marks
FROM students
GROUP BY course
ORDER BY total_marks DESC;

-- 3
SELECT course, ROUND(AVG(marks), 2) AS average_marks
FROM students
GROUP BY course
ORDER BY average_marks DESC;

-- 4
SELECT course, COUNT(*) AS total_students
FROM students
WHERE marks >= 70
GROUP BY course;

-- 5
SELECT course, COUNT(*) AS total_students
FROM students
GROUP BY course
HAVING COUNT(*) > 5;

-- 6
SELECT course, ROUND(AVG(marks), 2) AS average_marks
FROM students
GROUP BY course
HAVING AVG(marks) > 80;

-- 7
SELECT course,
       COUNT(*) AS total_students,
       ROUND(AVG(marks), 2) AS average_marks
FROM students
WHERE marks >= 70
GROUP BY course
HAVING COUNT(*) > 3;

-- 8
SELECT course, gender, COUNT(*) AS total_students
FROM students
GROUP BY course, gender;
```

Topics reviewed:

- Grouping rows using `GROUP BY`.
- Counting rows using `COUNT(*)`.
- Filtering groups using `HAVING`.
- Difference between `WHERE` and `HAVING`.
- Grouping with multiple columns.
- Using `HAVING` without an explicit `GROUP BY`.

**Key revision point:** `HAVING` can be used without `GROUP BY` in an aggregate query. The entire input is then treated as a single group.

**Next lesson:** Subqueries.
