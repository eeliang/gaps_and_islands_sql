# Single column

> Sample data

```sql
CREATE TABLE IF NOT EXISTS RAINY_DAYS (
  dates date NOT  NULL
);
INSERT INTO RAINY_DAYS VALUES 
('2023-03-01'),
('2023-03-02'),
('2023-04-01'),
('2023-04-05'),
('2023-04-06'),
('2023-04-07'),
('2023-04-09');

```
In this version, there is only 1 column. The rows are not guaranteed to be in sequence or consecutive.

For example, table **rainy_days** contains 1 column: `dates`. This table records if it rained every day. Days with no rain will not be reflected in the table.

References:

- [Single column gap and island](https://www.mssqltips.com/sqlservertutorial/9130/sql-server-window-functions-gaps-and-islands-problem/).
- [Dense_rank vs row_number](https://codingsight.com/similarities-and-differences-among-rank-dense_rank-and-row_number-functions/).

## Task objective: Split gap and island, count sequence_length

> Desired output

```md
+------------+------------+-----------------+--------+
| start_date | end_date   | sequence_length | type   |
+------------+------------+-----------------+--------+
| 2023-03-01 | 2023-03-02 |               2 | island |
| 2023-03-03 | 2023-03-31 |              29 | gap    |
| 2023-04-01 | 2023-04-01 |               1 | island |
| 2023-04-02 | 2023-04-04 |               3 | gap    |
| 2023-04-05 | 2023-04-07 |               3 | island |
| 2023-04-08 | 2023-04-08 |               1 | gap    |
| 2023-04-09 | 2023-04-09 |               1 | island |
+------------+------------+-----------------+--------+
```

The goal of this challenge is to find (1) consecutive raining days and (2) the gaps between rainy days.

The output should contain all consecutive dates, populating the missing dates in the input table as **gaps**. 

## Solution: calculate Gaps and Island separately

```sql
with
island_details as (
  SELECT 
    MIN(r.dates) as start_date,
    MAX(r.dates) as end_date,
    COUNT(r.dates) + 1 AS sequence_length,
    'island' as type
  FROM (
    SELECT
      dates,
      DATE_ADD(dates, INTERVAL -(DENSE_RANK() over (ORDER BY dates)) DAY) AS group_key
    FROM rainy_days
  ) r
  GROUP BY group_key
),
gap_details as (
  SELECT
    DATE_ADD(t1.dates, INTERVAL 1 DAY) AS start_date,
    DATE_ADD(MIN(t2.dates), INTERVAL - 1 DAY) AS end_date,
    DATEDIFF(MIN(t2.dates), t1.dates) - 1 AS sequence_length,
    'gap' AS type
  FROM rainy_days t1
  LEFT JOIN rainy_days t2 ON t2.dates > t1.dates
  GROUP BY t1.dates
  HAVING DATEDIFF(MIN(t2.dates), t1.dates) > 1
),
final_output AS (
  SELECT * FROM ISLAND_DETAILS
  UNION ALL
  SElecT * FROM GAP_DETAILS
)
SELECT * FROM final_output
ORDER BY start_date, end_date;
```

The process to get the gaps and islands are distinct as there is no reliable index column - in sequence and consecutive.

Therefor, we calculate them separately and combine the results afterwards.

### `island_details`
This solution requires us to understand how consecutive data points work in conjunction with the `DENSE_RANK()` function. When consecutive data points (e.g. consecutive dates) increment step by step with the index column `rn`, it returns a unique `group_key` for that series of consecutive rows.

Next, we calculate MIN(), MAX(), and count() of each `group_key` which gives us the `start_date`, `end_date`, and `sequence_length`.

<aside class="success">
Why do we use <a href="#how-to-get-unique-group_keys"><code>DENSE_RANK()</code> over <a href=""><code>ROW_NUMBER()</code></a>.
</aside>

### `gap_details`
We use the difference between each date and the next earliest date to find gaps. 

This is done by joining `rainy_days` onto itself and aggregating the next MIN() date. The result is the next earliest date for each row, which is related to the `start_date` and `end_date` of a gap.

We define a gap as 2 active dates that are >1 days apart. `DATEDIFF(MIN(t2.dates), t1.dates) > 1`.

Lastly, we increment +1 from the last active day to get the gap `start_date`, and -1 from the next active day to get the gap `end_date`.

### `final_output`
To present the results in sequence, we combine the `island_details` and `gap_details` using the `union` operator.
