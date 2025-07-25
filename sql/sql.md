# SQL Study Guide

## `WHERE`, `GROUP BY`, and `HAVING`

- `WHERE`: specifies criteria that field values must meet to be included in the query results.
- `GROUP BY`: Groups result rows sharing one or more values for aggregation.
- `HAVING`: Filters groups created by `GROUP BY` (like `WHERE` but for groups).

```
SELECT department, COUNT(*) AS ct
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

You don't actually need `COUNT(*) AS ct` to still use HAVING.

### Subquery Expressions

All of the expression forms return boolean (true/false) results.

- `WHERE EXISTS (subquery)`: If the subquery returns at least one row, the result of EXISTS is “true”; if the subquery returns no rows, the result of EXISTS is “false”. 
- `WHERE col IN (subquery or list)`: The IN operator is a filtering condition in SQL that checks whether a value exists in a list or subquery result.
- `WHERE col NOT IN (subquery or list)`: Use the IN operator to check whether a value does not exist in a list or subquery result.

`EXISTS` `SELECT` example
```
SELECT
  name,
  EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
  ) AS has_orders
FROM customers c;
```

`IN` `SELECT` example
```
CASE
    WHEN id IN (1, 2) THEN 'Priority'
    ELSE 'Regular'
END AS customer_type
```

### Efficiency Tips

- Filter on indexed columns.
- Filter early with `WHERE` clauses.
- Filter before joining in a subquery or CTE.

## Joins

Joins combine rows from two or more tables based on a related column.
- `INNER JOIN`: Returns only matching rows. This is the default `JOIN`.
- `LEFT JOIN` / `RIGHT JOIN`: Returns all rows from one table, along with matched rows from the other.
- `FULL (OUTER) JOIN`: Returns rows from both tables; NULL in columns where there's no match.
- `CROSS JOIN`: Cartesian product; every row of one table with every row of the other.

![](img/cross-join.png)

```
SELECT *
FROM Customers
CROSS JOIN Shopping_Details;
```

- Self join: A table joined to itself. Common for hierarchical data (product categories and subcategories, employees and their managers, etc).

### Join Efficiency Tips

- Join on indexed columns.
    - Boosts the efficiency and speed of searches, filtering, and joins (`WHERE`, `JOIN`, `ORDER BY`, `GROUP BY`). Think of an index like a book’s table of contents: instead of flipping through every page (row), you can jump directly to where the data lives.
    - Should be used strategically because it takes up disk space and leads to slower `INSERT`, `UPDATE`, `DELETE` operations.
- `INNER JOIN`s are generally faster than `OUTER JOIN`s (`LEFT`, `RIGHT`, `FULL`) because they return only matching records. `OUTER JOIN`s require handling unmatched rows, which increases processing time.
- Place the smaller, filtered tables earlier in a multi-way join. Join order matters. PostgreSQL’s planner optimizes join order, but you can help it by joining already-filtered or smaller tables first in join clauses or subqueries.
- Avoid `CROSS JOIN`s unless necessary.

## Aggregate Functions

### General Purpose

- `COUNT(col)`: Number of items, `COUNT(*)`: Number of rows, `COUNT(DISTINCT col)`: Number of distinct items
- `SUM(col)`: Total value
- `AVG(col)`: Average value
- `MIN(col)` / `MAX(col)`: Minimum / maximum
- `STRING_AGG(col, delimiter string)`: Concatenates non-null values into a string
- `BOOL_AND(col)`: Returns true if all non-null input values are true, otherwise false
- `BOOL_OR(col)`: Returns true if any non-null input value is true, otherwise false

### Statistics

- `CORR(col1, col2)`: Computes the correlation coefficient
- `COVAR_SAMP(col1, col2)`: Computes the sample covariance
- `STDDEV(col)` / `STDDEV_SAMP(col)`: Computes the sample standard deviation of the input values
- `VARIANCE(col)` / `VAR_SAMP(col)`: Computes the sample variance of the input values (square of the sample standard deviation)
- Where applicable, replace `SAMP` with `POP` for the population version of the metric

All of the above can be used as a window function with the exception of `COUNT(DISTINCT)`. Use a subquery or CTE instead.

## Window Aggregation Functions

Window functions apply calculations across sets of rows related to the current row without collapsing them.

- `ROW_NUMBER()`: Assigns a unique, sequential integer to each row within a partition, starting at 1 for the first row.
- `RANK()`: Assigns a rank to each row within a partition, with gaps in the ranking if there are ties.
- `DENSE_RANK()`: Assigns a rank to each row within a partition, like RANK(), but without gaps for tied values.

![](img/rank.png)

- `LAG(col[, offset integer [, default anycompatible ]])`: Returns the value of a specified column from a previous row within the current window partition. Returns value evaluated at the row that is offset rows before the current row within the partition; if there is no such row, instead returns default (which must be of a type compatible with value). Both offset and default are evaluated with respect to the current row. If omitted, offset defaults to 1 and default to NULL.
- `LEAD(col[, offset integer [, default anycompatible ]])`: Returns the value of a specified column from a following row within the current window partition. Returns value evaluated at the row that is offset rows after the current row within the partition; if there is no such row, instead returns default (which must be of a type compatible with value). Both offset and default are evaluated with respect to the current row. If omitted, offset defaults to 1 and default to NULL.

![](img/lag-lead.webp)

- `FIRST_VALUE(col)`: Retrieves the first value in an ordered window partition for each row.
- `LAST_VALUE(col)`: Retrieves the last value in an ordered window partition for each row.
- `NTH_VALUE(col, n integer)`: Retrieves the n-th value in an ordered window partition for each row.
- `PERCENT_RANK()`: Calculates the relative rank of each row as a percentage between 0 and 1 within a partition.
- `CUME_DIST()`: Computes the cumulative distribution of a value, showing the proportion of rows less than or equal to the current row within a partition.
- `NTILE(num_buckets integer)`: Divides the ordered partition into a specified number of ranked groups (tiles) and assigns each row to a group.

```
SELECT
    employee,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS depart_sal_rank
FROM employees;
```

## Subqueries and Common Table Expressions (CTEs)

- Subquery: A query nested inside another query.
- CTE (`WITH` clause): Names a subquery for reuse in later query parts, improves readability and reuse.

### Correlated Subquery

The subquery depends on the outer query (c.id) — it runs once per row in the outer query. Useful for row-by-row calculations.

```
SELECT
  name,
  (
    SELECT COUNT(*)
    FROM orders o
    WHERE o.customer_id = c.id
  ) AS order_count
FROM customers c;
```

### Inline, Uncorrelated Subquery

A subquery that runs independently of the outer query, executed just once, and does not reference the outer query. The subquery runs independently and returns customer IDs.

Example 1
```
SELECT name
FROM customers
WHERE id IN (
  SELECT customer_id
  FROM orders
  WHERE total > 60
);
```

Example 2
```
SELECT
  c.name,
  co.order_count
FROM customers c
LEFT JOIN (
  SELECT customer_id, COUNT(*) AS order_count
  FROM orders
  GROUP BY customer_id
) co ON c.id = co.customer_id;
```

### CTE

A named temporary result set, defined with `WITH`, that can be referenced like a table in the main query. Makes queries more readable and reusable. Often better for performance than correlated subqueries if reused or complex. The CTE customer_orders is defined first, like a temporary named result.

```
WITH customer_orders AS (
  SELECT customer_id, COUNT(*) AS order_count
  FROM orders
  GROUP BY customer_id
)
SELECT
  c.name,
  co.order_count
FROM customers c
LEFT JOIN customer_orders co ON c.id = co.customer_id;
```

| Feature | Correlated Subquery | Uncorrelated Subquery | CTE |
| -- | -- | -- | -- |
| Runs Once or Per Row? | Once per row in outer query | Once, regardless of outer query | Once per reference (can be optimized by planner) |
| Clarity / Readability | Harder to read | Simple for small cases | Best for complex logic or reuse |
| Performance | Can be slow on large datasets | Efficient if small / indexed | Good with indexing, but can be inlined or materialized depending on query |
| Use Case | Row-wise filtering/aggregation | Independent filters or joins | Complex queries, modular logic, recursion |
| Where used? | Often in `SELECT`, `WHERE`, `EXISTS` | Often in `IN`, `WHERE`, `FROM` | With `WITH`, anywhere in query |

## Unions

`UNION` combines results from multiple `SELECT` queries into one result set.
- `UNION ALL`: Includes duplicates.
- `UNION`: Removes duplicates.

## Conditional Expressions

- `CASE`: Used to add `IF`/`THEN`/`ELSE` logic inline.
```
SELECT name,
  CASE
    WHEN salary > 100000 THEN 'High'
    ELSE 'Regular'
  END AS salary_group
FROM employees;
```
- `COALESCE(*col*[, ...])`: Returns the first non-null value in a list.
- `NULLIF(*col1*, *col1*)`: Returns null if col1 equals col2; otherwise it returns col1.
- `GREATEST()`: ...
- `LEAST()`: ...

## String Functions & Pattern Matching

- `LIKE` / `ILIKE`: Simple wildcard pattern matching
    - `LIKE`: case-sensitive (case matters)
    - `ILIKE`: case-insensitive (case doesn't matter)
    - `%` for any sequence of characters, `_` for a single char
- `~*`: Case-insensitive POSIX regular expression match. `WHERE email ~* '^[a-z0-9._%+-]+@gmail\.com$'`
    - MORE TO LEARN ON POSIX REGULAR EXPRESSION
- S`IMILAR TO`: Advanced pattern matching using regex. `WHERE code SIMILAR TO '[A-Z]{2}[0-9]{4}'`
    - MORE TO LEARN ON SQL REGULAR EXPRESSION
- Substring: Extracts part of a string by pattern or position.
    - `SUBSTRING(string [FROM int] [FOR int])`: Extract substring from position to position.
    - `SUBSTRING(string FROM pattern)`: Extract substring matching POSIX regular expression.
    - `SUBSTRING(string FROM pattern FOR escape)`: Extract substring matching SQL regular expression.
- `LENGTH(str)`: Returns the number of characters in a string.
- `LOWER(str)` / `UPPER(str)`: Converts all characters to lower or upper case.
- `TRIM([LEADING | TRAILING | BOTH] [characters] FROM str)`: Removes leading/trailing characters (default is whitespace). `TRIM('  hello  ')` OR `TRIM(BOTH 'x' FROM 'xTomxx')`
- `CONCAT(str1, str2, ...)`: Joins multiple strings together. Is null-safe so it ignores nulls.
- `POSITION(substring IN string)`: `position('om' in 'Thomas')` --> 3 OR `STRPOS('high', 'ig')` --> `2`
- `REGEXP_MATCHES(string text, pattern text)`: Return all captured substrings resulting from matching a POSIX regular expression against the string. `REGEXP_REPLACE('foobarbequebaz', '(bar)(beque)')` --> `{bar,beque}`
- `REGEXP_REPLACE(string text, pattern text, replacement text)`: Replace substring(s) matching a POSIX regular expression. `REGEXP_REPLACE('Thomas', '.[mN]a.', 'M')` --> `ThM`
- `REPLACE(string text, from text, to text)`: Replace all occurrences in string of substring from with substring to `REPLACE('abcdefabcdef', 'cd', 'XX')` --> `abXXefabXXef`
- `REVERSE(str)`: Return reversed string.

## Date/Time Functions and Operators

- `CURRENT_DATE` / `NOW()`: Returns the current date (without time), based on the server's time zone.
- `CURRENT_TIMESTAMP`: Returns the current date and time (with time zone awareness).
- `DATE_PART(part, [ts | interval])` / `EXTRACT(part FROM [ts|interval])`: Extracts a specific part (like year, month, day, etc.) from a date or timestamp.
- `DATE_TRUNC(part, ts)`: Truncates a timestamp to a specified precision (e.g., to the nearest day, month, etc.)
- Interval arithmetic: Adds or subtracts an interval (like days, months, hours) to/from a date or timestamp.
    - `'2020-01-01'::date + INTERVAL '1 day'`
    - `DATE_ADD(ts, interval)` / `DATE_SUBTRACT(ts, interval)`
        - interval can look like `'1 day'::interval`, ...
- Making a date/time
    - `MAKE_DATE(year int, month int, day int)`
    - `MAKE_INTERVAL([years int [, months int [, weeks int [, days int [, hours int [, mins int [, secs double precision]]]]]]])`
    - `MAKE_TIME(hour int, min int, sec double precision)`
    - `MAKE_TIMESTAMP(year int, month int, day int, hour int, min int, sec double precision)`
        - ADD `TZ` to add a timezone
    - `DATE '2006-01-02'`
    - `TIMESTAMP '2001-02-18 20:38:40'`

- NEED LIST OF PARTS. IS IT DIFFERENT FOR INTERVAL ARITHMETIC? INTERVAL?

## Array Functions, Operations, and Comparisons

- `ARRAY_AGG(col)`: Aggregates values, including nulls, into an array.
- `ARRAY_LENGTH(arr, dimension)`: Returns the length of the requested array dimension. Produces NULL for empty or missing array dimensions.
    - `array_length(ARRAY[[1,2],[3,4],[5,6]], 1) --> 3` row count
    - `array_length(ARRAY[[1,2],[3,4],[5,6]], 2) --> 2` col count
- `UNNEST(arr)`: Expands an array into a set of rows, which can be joined back to a table.
- `ARRAY[]` literals are used to define an array explicitly, which can be nested.
- Array column access at a given position in an array: `col[1]`
- `ANY`, `ALL`: Compare a scalar value to each element in an array.
    - `ANY`: True if at least one value matches
    - `ALL`: True if all comparisons pass

`ANY` example
```
SELECT 3 = ANY(ARRAY[1, 2, 3]);  -- TRUE
...
WHERE status = ANY(ARRAY['shipped', 'processing']);
```

`ALL` example
```
SELECT 3 < ALL(ARRAY[4, 5, 6]);  -- TRUE
...
WHERE score > ALL (ARRAY[70, 75, 85]);
```

- `ARRAY_APPEND(arr, value)`: Appends an element to the end of an array.
- `ARRAY_PREPEND(value, arr)`: Prepends an element to the beginning of an array.
- `ARRAY_CAT(arr1, arr2)`: Concatenates two arrays.
- `ARRAY_DIMS(arr)`:...
- `ARRAY_FILL(value, arr[rows int, cols int])`: Returns an array filled with copies of the given value, having dimensions of the lengths specified by the second argument.
- `ARRAY_NDIMS(arr)`: Returns the number of dimensions of the array. `ARRAY_NDIMS(ARRAY[[1,2,3], [4,5,6]]) -> 2`
- `ARRAY_POSITIONS(arr, value)`: Returns an array of the subscripts of all occurrences of the second argument in the array given as first argument. The array must be one-dimensional. If the value is not found in the array, an empty array is returned. It is possible to search for NULLs.
- `ARRAY_REMOVE(arr, value)`: Removes all elements equal to the given value from the array. The array must be one-dimensional. It is possible to remove NULLs.
- `ARRAY_REPLACE(arr, to_replace, replace_with)`: Replaces each array element equal to the second argument with the third argument.
- `ARRAY_SAMPLE(arr, n integer)`: Returns an array of n items randomly selected from array.
- `ARRAY_TO_STRING(arr, delimiter text [, null_string text ])`: Converts each array element to its text representation, and concatenates those separated by the delimiter string. If null_string is given, then NULL array entries are represented by that string.
- `CARDINALITY(arr)`: Returns the total number of elements in the array, or 0 if the array is empty. `CARDINALITY(ARRAY[[1,2],[3,4]]) --> 4`
- `TRIM_ARRAY(arr, n integer)`: Trims an array by removing the last n elements. `TRIM_ARRAY(ARRAY[1,2,3,4,5,6], 2) --> {1,2,3,4}`


### Array Operators

- `@>`, `<@`: Array comparison, checks if one array contains all elements of another.
    - `arr1 @> arr2` means "arr1 contains arr2"
```
SELECT ARRAY[1, 2, 3] @> ARRAY[2];     -- TRUE
SELECT ARRAY['a','b'] @> ARRAY['c'];  -- FALSE
```

- `&&`: Do the arrays have any elements in common, returns true or false.
- `||`: Concatenates the two arrays. The arrays must have the same number of dimensions. Can also concatenate one element with one array.

```
ARRAY[1,2,3] || ARRAY[4,5,6,7] → {1,2,3,4,5,6,7}
ARRAY[1,2,3] || ARRAY[[4,5,6],[7,8,9.9]] → {{1,2,3},{4,5,6},{7,8,9.9}}
```

## Additional Efficiency & Other Tips

- Use alias' for more interpretable code.
- Retrieve only the columns you need. Avoid `SELECT *`.
- Partition Large Tables. Partition Pruning: For especially large tables, horizontal partitioning (table inheritance or declarative partitioning) allows PostgreSQL to scan only relevant partitions during joins.
- Analyze and Optimize Plans
    - Use `EXPLAIN (ANALYZE)`: Always analyze your queries with `EXPLAIN` to understand how joins are executed. Look for full table scans or hash joins on large datasets; these may need rewriting or more indexes.
    - Review Execution Strategy: Pay attention to "Nested Loop," "Hash Join," and "Merge Join" and adjust your data or indexes if a suboptimal strategy is selected.
- Minimize Data Skew. Evenly Distributed Data: Joins can be slower if keys are very unevenly distributed (many-to-one or many-on-same-key). If possible, partition, cluster, or redesign for more even distribution.
- Use Proper Data Types. Avoid Implicit Casts: When join columns have mismatched types, PostgreSQL will perform implicit casts, slowing joins. Align data types for better performance.
- Monitor and Maintain Statistics. Run `VACUUM` and `ANALYZE`: Keep your table and index statistics up to date so the query planner can make optimal decisions.