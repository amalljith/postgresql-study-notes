# PostgreSQL — Lesson 02: Filtering Operators

**Beginner-friendly reference | SQL Shell (psql) | English**

## 1. Introduction

Filtering operators help retrieve only the rows that match a condition. This lesson covers `IN`, `NOT IN`, `LIKE`, `NOT LIKE`, `ILIKE`, `NOT ILIKE`, `IS NULL`, and `IS NOT NULL`.

## 2. Example Students Table

These examples use a simplified version of the `school_db.students` table.

| id | name | age | course | email |
|---:|---|---:|---|---|
| 1 | Arun | 22 | React | arun@gmail.com |
| 2 | Rahul | 24 | Python | NULL |
| 3 | Anu | 21 | Django | anu@gmail.com |
| 4 | Kiran | 23 | Django | NULL |
| 5 | Amal | 24 | React | NULL |

Your actual database may contain more rows and different results.

## 3. IN: Match Multiple Values

```sql
SELECT name, age, course
FROM students
WHERE age IN (21, 22, 24)
ORDER BY age ASC, id ASC;
```

Expected output:

| name | age | course |
|---|---:|---|
| Anu | 21 | Django |
| Arun | 22 | React |
| Rahul | 24 | Python |
| Amal | 24 | React |

`IN (21, 24)` means `= 21 OR = 24` for the selected column.

## 4. NOT IN: Exclude Values

```sql
SELECT name, age
FROM students
WHERE age NOT IN (21, 24)
ORDER BY age DESC;
```

Expected output:

| name | age |
|---|---:|
| Kiran | 23 |
| Arun | 22 |

**Important:** NULL values do not behave like ordinary values in `NOT IN`; be especially careful if the list contains NULL.

## 5. LIKE: Pattern Matching

`LIKE` performs case-sensitive pattern matching in ordinary PostgreSQL text searches.

```sql
SELECT name, course
FROM students
WHERE name LIKE 'A%'
ORDER BY name ASC;
```

Expected output:

| name | course |
|---|---|
| Amal | React |
| Anu | Django |
| Arun | React |

## 6. Wildcards: % and _

| Wildcard | Meaning | Example |
|---|---|---|
| `%` | Zero or more characters | `'A%'` starts with A |
| `_` | Exactly one character | `'A___'` is A plus three characters |

```sql
-- Ends with lowercase n
SELECT name FROM students WHERE name LIKE '%n';

-- Contains the sequence ra
SELECT name FROM students WHERE name LIKE '%ra%';

-- Exactly four characters beginning with uppercase A
SELECT name FROM students WHERE name LIKE 'A___';

-- Third character is lowercase n
SELECT name FROM students WHERE name LIKE '__n%';
```

**Common mistake:** `'A%___'` matches names with **at least** four characters beginning with A. `'A___'` matches exactly four characters.

## 7. NOT LIKE

```sql
SELECT name FROM students
WHERE name NOT LIKE 'A%';
```

Excludes names starting with uppercase A. NULL names are not matched.

## 8. ILIKE: Case-Insensitive Matching

`ILIKE` ignores upper/lowercase differences according to the active locale.

```sql
SELECT name FROM students
WHERE name ILIKE 'a%';

SELECT name, course FROM students
WHERE name ILIKE '%AN%'
ORDER BY name ASC;
```

The first query matches Amal, Anu, and Arun. The second matches Kiran in the example data.

## 9. NOT ILIKE

```sql
SELECT name, age FROM students
WHERE name NOT ILIKE 'a%'
ORDER BY age ASC;
```

Expected output:

| name | age |
|---|---:|
| Kiran | 23 |
| Rahul | 24 |

## 10. IS NULL and IS NOT NULL

`NULL` represents unknown or missing data. It is neither zero nor an empty string.

```sql
SELECT name, email FROM students
WHERE email IS NULL
ORDER BY name ASC;

SELECT name, email FROM students
WHERE email IS NOT NULL
ORDER BY name ASC;
```

The first query finds Amal, Kiran, and Rahul in the sample. The second finds Anu and Arun.

**Never use `email = NULL`.** It evaluates to UNKNOWN. Use `email IS NULL`.

## 11. Combining Filters

```sql
SELECT name, age, course
FROM students
WHERE age IN (22, 23, 24)
  AND course NOT IN ('Python')
  AND name NOT ILIKE 'a%'
ORDER BY age DESC;
```

`AND` requires all conditions to be satisfied; `OR` requires at least one.

## 12. Common Mistakes

1. Using `'react'` with `=` when the database stores `'React'`.
2. Forgetting to close a string quote in `ILIKE '%an%'`.
3. Using `%` when an exact number of characters is required.
4. Writing `= NULL` instead of `IS NULL`.
5. Assuming `IN` automatically sorts results.
6. Assuming `NOT IN` includes NULL values.

## 13. Quick Reference

| Goal | Pattern |
|---|---|
| Match a list | `IN (...)` |
| Exclude a list | `NOT IN (...)` |
| Starts with A | `LIKE 'A%'` |
| Ends with n | `LIKE '%n'` |
| Contains ra | `LIKE '%ra%'` |
| Exactly four characters starting with A | `LIKE 'A___'` |
| Case-insensitive match | `ILIKE 'a%'` |
| Exclude a case-insensitive pattern | `NOT ILIKE 'a%'` |
| Missing value | `IS NULL` |
| Known value | `IS NOT NULL` |

## 14. Practice Exercises

1. Find students aged 21, 22, or 24 and sort by age.
2. Exclude students aged 21 and 24.
3. Find names beginning with uppercase A.
4. Find names ending in lowercase n.
5. Find four-character names starting with A.
6. Find names containing `AN` without case sensitivity.
7. Exclude names starting with A or a.
8. Find students with missing email addresses.
9. Find students with known email addresses.

### Answer Key

```sql
-- 1
SELECT name, age, course FROM students WHERE age IN (21,22,24) ORDER BY age ASC;
-- 2
SELECT name, age FROM students WHERE age NOT IN (21,24) ORDER BY age DESC;
-- 3
SELECT name, course FROM students WHERE name LIKE 'A%' ORDER BY name ASC;
-- 4
SELECT name, age FROM students WHERE name LIKE '%n' ORDER BY age DESC;
-- 5
SELECT name, course FROM students WHERE name LIKE 'A___' ORDER BY name ASC;
-- 6
SELECT name, course FROM students WHERE name ILIKE '%AN%' ORDER BY name ASC;
-- 7
SELECT name, age FROM students WHERE name NOT ILIKE 'a%' ORDER BY age ASC;
-- 8
SELECT name, email FROM students WHERE email IS NULL ORDER BY name ASC;
-- 9
SELECT name, email FROM students WHERE email IS NOT NULL ORDER BY name ASC;
```

## 15. Final Quiz Review

**Score: 5/5.** Topics: `NOT IN`, four-character `ILIKE` patterns, `IS NULL`, names ending in n, and combining conditions using `AND`.

**Next lesson:** Aggregate Functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`).
