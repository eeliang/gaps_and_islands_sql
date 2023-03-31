# Range with status

> Sample data

```sql
DROP TABLE library_books;
CREATE TABLE IF NOT EXISTS library_books (
  borrower integer NOT NULL,
  book_id integer NOT NULL,
  checkout_date date NOT  NULL,
  return_date date NOT  NULL
);
INSERT INTO library_books VALUES
(2, 03, '2023-03-01', '2023-03-03'), 
(1, 01, '2023-03-01', '2023-03-03'),
(1, 02, '2023-03-02', '2023-03-03'),
(2, 04, '2023-03-02', '2023-03-04'),
(2, 02, '2023-03-03', '2023-03-06'),
(1, 03, '2023-03-04', '2023-03-04'),
(1, 01, '2023-03-09', '2023-03-10');
```

In this version, there are 2 columns that signify the range, and more columns to describe unique attributes like groups or product ID. The ranges are not guaranteed to be in sequence, mutually exclusive, or does it cover the entire range. 

Table **library_rentals** contains 4 columns: `borrower`, `book_id`,`checkout_date`, and `return_date`. They record which book was borrowed and for how long.

For example, ranges [(1,6),(2,5),(3,4)] should return (1,6).

References:

- [Resolving overlapping ranges](https://medium.com/analytics-vidhya/sql-classic-problem-identifying-gaps-and-islands-across-overlapping-date-ranges-5681b5fcdb8)
- [Merging contiguous ranges](https://peterevans.dev/posts/gaps-and-islands-merging-contiguous-ranges/).


## Task objective: Group ranges that overlap

> Desired output

```
+------------+------------+-----------+-----------+-----------------+
| start_date | end_date   | group_key | num_books | sequence_length |
+------------+------------+-----------+-----------+-----------------+
| 2023-03-01 | 2023-03-05 |        11 |         2 |               4 |
| 2023-03-01 | 2023-03-06 |        12 |         3 |               5 |
| 2023-03-09 | 2023-03-10 |        21 |         1 |               1 |
+------------+------------+-----------+-----------+-----------------+
```

The goal of this challenge is to find the (1) overlapping range and report the the earliest and latest data points, group by borrower, ignoring the book_id and (2) the gaps where no books where borrowed.

## Solution

```sql
with
last_return_date AS (
  SELECT borrower, book_id, checkout_date, return_date,
    DENSE_RANK() over (ORDER BY checkout_date) AS rn,
    MAX(return_date) OVER (PARTITION BY borrower ORDER BY checkout_date, return_date ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING) AS last_return_date
  FROM library_books
  ORDER BY checkout_date, return_date
),
start_of_island AS (
  SELECT l.*, CASE WHEN checkout_date <= last_return_date THEN 0 ELSE 1 END AS is_start
  FROM last_return_date l
  ORDER BY checkout_date, return_date
),
islands_will_share_group_key AS (
  SELECT s.*, borrower+10*SUM(is_start) OVER ( PARTITION BY borrower ORDER BY checkout_date, return_date) AS group_key
  FROM start_of_island s
  ORDER BY checkout_date, return_date
),
consolidated_range_for_each_group AS (
  SELECT MIN(checkout_date) AS start_date, MAX(return_date) AS end_date, group_key, COUNT(book_id) AS num_books, datediff(MAX(return_date),MIN(checkout_date)) AS sequence_length
  FROM islands_will_share_group_key
  GROUP BY group_key
  HAVING COUNT(*) > 0
  ORDER BY start_date, end_date
)
select * from consolidated_range_for_each_group;
```

Go through step by step.

### `last_return_date`
The first step is to order all event by `checkout_date` and `return_date`. For each row, find the most recent `return_date` (`last_return_date`). This allow us to check if the end points of the range is extended by an earlier row.

### `start_of_island as (`
This checks if there is a consecutive streak from the `last_return_date`. If not, this signifies that there is a new **island**.

### `islands_will_share_group_key`
Using the `SUM()` window function, the `group_key` counts all the exclusive new islands of the group - partitioned by unique attributes like user_id or borrower.

<aside class="notice">
There may be rare instance where the <code>group_key</code> is  duplicated across statuses. See <a href="#avoid-error-of-overalapping-groups">how to address this issue</a>.
</aside>

### `consolidated_range_for_each_group`
Finally, use the group_key to find the min(), max(), and count(). This will merge and reduce the initial ranges to distinct ranges with overlaps. Each range will specific the unique identified (e.g. user_id, borrower).
