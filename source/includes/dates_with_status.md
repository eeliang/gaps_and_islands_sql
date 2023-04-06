# Date column with Status

> Sample data

```sql
CREATE TABLE IF NOT EXISTS park_visitors (
  dates date NOT  NULL,
  visitors integer NOT NULL
);
INSERT INTO park_visitors VALUES 
('2023-03-01', 10),
('2023-03-02', 109),
('2023-03-03', 89),
('2023-03-04', 76),
('2023-03-05', 150),
('2023-03-06', 500),
('2023-03-07', 1455),
('2023-03-08', 54),
('2023-03-09', 98);
```

In this version, there are 2 columns. The rows are not guaranteed to be in sequence, but is guaranteed to contain all consecutive days.

For example, table **park_visitors** contains 2 columns: `dates`, and `visitors`. This table records the number of visitors every day, recording 0 if there are no visitors.

References: 

- [Analyzing total downtime of a IoT device](https://towardsdatascience.com/gaps-and-islands-with-mysql-b407040d133d).

## Task objective: Group by status, count sequence_length

> Desired output

```md
+------------+------------+-----------+--------+-----------------+
| start_date | end_date   | group_key | status | sequence_length |
+------------+------------+-----------+--------+-----------------+
| 2023-03-01 | 2023-03-01 |         0 |      0 |               1 |
| 2023-03-02 | 2023-03-02 |        11 |      1 |               1 |
| 2023-03-03 | 2023-03-04 |        10 |      0 |               2 |
| 2023-03-05 | 2023-03-07 |        31 |      1 |               3 |
| 2023-03-08 | 2023-03-09 |        40 |      0 |               2 |
+------------+------------+-----------+--------+-----------------+
```

The goal of this challenge is to find the (1) streaks where there are more than <100> visitors each day and (2) the gaps between these busy days. 

<aside class="success">
Note, the <strong>gaps</strong> are represented AS <code>status=0</code>.
</aside>

## Solution: rank() with partition by status

```sql
WITH
classify_into_status AS (
  SELECT 
    dates, 
    CASE WHEN visitors >= 100 THEN 1 ELSE 0 END AS status
  FROM park_visitors
  ORDER BY dates
),
sort_and_rank AS (
  SELECT 
    s.*,
    DENSE_RANK() over (ORDER BY dates) AS rn,
    DENSE_RANK() over (PARTITION BY status ORDER BY dates) AS rn_by_status
  FROM classify_into_status s
  ORDER BY dates
),
consecutive_dates_will_share_group_key AS (
  SELECT 
    r.*, 
    status+10*(rn-rn_by_status) AS group_key
  FROM sort_and_rank r
),
final_output AS (
  SELECT 
    group_key, 
    status,
    MIN(dates) AS start_date, 
    MAX(dates) AS end_date,  
    COUNT(*) AS sequence_length
  FROM consecutive_dates_will_share_group_key
  WHERE status in (1,0)
  GROUP BY group_key, status
  ORDER BY start_date, end_date
)
SELECT * FROM final_output
WHERE sequence_length > 0;
```

We will go though each CTE step-by-step.

### `classify_into_status`
We apply a conditional statement to band the number columns into discrete statuses - Changing number of visitors to "high_traffic" / "low_traffic".

### `sort_and_rank`
This CTE orders the table by the `dates` column. We create an index column `rn` and a separate rank column `rn_by_status` for each status.

### `consecutive_dates_will_share_group_key`
Subtracting `rn_by_status` from `rn` will return a unique `group_key` for consecutive rows. This works because the `rn_by_status` column increments at the same pace as the `rn` column. This allows us to have a unqiue `group_key` for each island.

<aside class="notice">
There may be rare instance where the <code>group_key</code> is  duplicated across statuses. See <a href="#avoid-error-of-overalapping-groups">how to address this issue</a>.
</aside>

### `final_output`
Using the `group_key`, find the min(), max(), and count().
