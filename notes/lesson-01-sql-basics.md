# PostgreSQL — Study Notes

**Beginner-friendly reference | SQL Shell (psql) | English**

## 1. What is PostgreSQL?

PostgreSQL is a relational database management system. It stores related data in tables made of rows (records) and columns (fields). SQL is the language used to work with that data.

Our practice database is `school_db`, and the table is `students` with four columns: `id`, `name`, `age`, and `course`.

## 2. Essential psql Commands

| Command        | Purpose                             |
| -------------- | ----------------------------------- |
| `\l`           | List databases                      |
| `\c school_db` | Connect to school_db                |
| `\dt`          | List tables in the current database |
| `\d students`  | Show students table structure       |
| `\q`           | Quit SQL Shell                      |

**Remember:** psql backslash commands do not need a semicolon. SQL statements normally end with `;`.

## 3. Create a Table

```sql
CREATE TABLE students (
    id INTEGER,
    name VARCHAR(100),
    age INTEGER,
    course VARCHAR(50)
);
```

`INTEGER` stores whole numbers. `VARCHAR(100)` stores variable-length text up to 100 characters. This is a simple practice table; in a production design, an `id` primary key is recommended.

## 4. Insert Data

```sql
INSERT INTO students (id, name, age, course)
VALUES (1, 'Arun', 22, 'React');

INSERT INTO students (id, name, age, course)
VALUES
    (2, 'Rahul', 24, 'Python'),
    (3, 'Anu', 21, 'Django');
```

`INSERT 0 2` means two rows were inserted. You may omit the column list only when supplying values for all columns in their defined order; listing columns is safer.

## 5. Read Data with SELECT

```sql
SELECT * FROM students;
SELECT name, age FROM students;
```

`*` selects every column. Listing column names selects only the information you need.

## 6. Filter with WHERE

```sql
SELECT name, age FROM students WHERE age > 21;
SELECT * FROM students
WHERE age > 20 AND course = 'React';
SELECT * FROM students
WHERE age < 22 OR course = 'Python';
```

| Operator     | Meaning                           |
| ------------ | --------------------------------- |
| `=`          | Equal to                          |
| `<>` or `!=` | Not equal to                      |
| `>` / `<`    | Greater than / less than          |
| `>=` / `<=`  | Greater/less than or equal to     |
| `AND`        | Both conditions must match        |
| `OR`         | At least one condition must match |

Text equality is case-sensitive in these examples: `'React'` and `'react'` are different. `ILIKE` can be used for case-insensitive pattern matching (covered in a later lesson).

## 7. Update Existing Rows

```sql
UPDATE students
SET course = 'Django'
WHERE id = 10;

UPDATE students
SET course = 'FastAPI'
WHERE id = 10
RETURNING *;
```

`UPDATE 1` means one row was updated. `RETURNING *` displays the affected row. **Without `WHERE`, every row may be updated.**

## 8. Delete Rows Safely

```sql
-- Preview matching rows first
SELECT * FROM students WHERE age < 21;

-- Then delete only those rows
DELETE FROM students WHERE age < 21;

-- Optional: see deleted rows
DELETE FROM students WHERE age < 21 RETURNING *;
```

`DELETE 0` means the command succeeded but no rows matched. **`DELETE FROM students;` deletes every row in the table.**

## 9. Sort Results with ORDER BY

```sql
SELECT name, age FROM students ORDER BY age ASC;
SELECT name, age FROM students ORDER BY age DESC;
```

`ASC` sorts smallest to largest (or A–Z); `DESC` reverses the order. If several students share the same age, add a unique tie-breaker to make results predictable:

```sql
SELECT name, age FROM students
ORDER BY age ASC, id ASC;
```

This assumes `id` values are unique.

## 10. LIMIT and OFFSET

```sql
SELECT name, age FROM students
ORDER BY age DESC
LIMIT 2;

SELECT name, age FROM students
ORDER BY age ASC, id ASC
LIMIT 3 OFFSET 2;
```

`LIMIT 3` returns at most three rows; `OFFSET 2` skips the first two rows. This is useful for simple pagination. For consistent pages, use a deterministic `ORDER BY`.

| Page | LIMIT | OFFSET |
| ---- | ----: | -----: |
| 1    |    10 |      0 |
| 2    |    10 |     10 |
| 3    |    10 |     20 |

## 11. DISTINCT: Remove Duplicate Values

```sql
SELECT DISTINCT course
FROM students
ORDER BY course ASC;

SELECT DISTINCT LOWER(course)
FROM students
ORDER BY LOWER(course);
```

`DISTINCT` removes duplicate results. `LOWER(course)` also normalizes differences in letter case before comparing.

## 12. BETWEEN: Filter a Range

```sql
SELECT name, age, course
FROM students
WHERE age BETWEEN 21 AND 23
ORDER BY age DESC;
```

`BETWEEN 21 AND 23` **includes both endpoints**. It is equivalent to `age >= 21 AND age <= 23`.

## 13. IN: Match One of Several Values

```sql
SELECT name, course
FROM students
WHERE course IN ('React', 'Django');
```

This is equivalent to `course = 'React' OR course = 'Django'`. More `IN` and `NOT IN` practice is planned for the next lesson.

## 14. Common Mistakes

1. Forgetting the `;` at the end of an SQL statement.
2. Writing `React` instead of `'React'` for a text value.
3. Using `UPDATE` or `DELETE` without `WHERE` by accident.
4. Assuming `BETWEEN 21 AND 23` excludes 21 or 23 (it includes both).
5. Expecting a fixed order without `ORDER BY`.
6. Assuming `ORDER BY age` gives a fixed order for students with the same age.
7. Supplying fewer values than columns in `INSERT INTO students VALUES (...)`.

## 15. Quick Reference

| Goal                 | SQL            |
| -------------------- | -------------- |
| Create table         | `CREATE TABLE` |
| Add rows             | `INSERT INTO`  |
| Read rows            | `SELECT`       |
| Filter rows          | `WHERE`        |
| Change rows          | `UPDATE`       |
| Remove rows          | `DELETE`       |
| Sort                 | `ORDER BY`     |
| Limit count          | `LIMIT`        |
| Skip rows            | `OFFSET`       |
| Unique values        | `DISTINCT`     |
| Inclusive range      | `BETWEEN`      |
| Match several values | `IN`           |

## 16. Practice Exercises

1. Show the names and ages of students older than 22.
2. Show the three oldest students, including their names and ages.
3. List unique courses alphabetically.
4. Show students aged 21 through 24, sorted by age descending.
5. Show names and courses for students taking React or Django.
6. Skip the first two students in ascending age order and show the next three. Break ties using `id`.

### Answer Key

```sql
-- 1
SELECT name, age FROM students WHERE age > 22;
-- 2
SELECT name, age FROM students ORDER BY age DESC, id ASC LIMIT 3;
-- 3
SELECT DISTINCT course FROM students ORDER BY course ASC;
-- 4
SELECT * FROM students WHERE age BETWEEN 21 AND 24 ORDER BY age DESC;
-- 5
SELECT name, course FROM students WHERE course IN ('React', 'Django');
-- 6
SELECT name, age FROM students ORDER BY age ASC, id ASC LIMIT 3 OFFSET 2;
```

**Next lesson:** `IN`, `NOT IN`, `LIKE`, and `ILIKE`.
