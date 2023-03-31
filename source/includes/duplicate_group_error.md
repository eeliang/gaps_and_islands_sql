# Avoid error of duplicate group_keys


### Prerequites: how to get unique group_keys

```
+------------+----+--------------+--------+-----------+
| dates      | rn | rn_by_status | status | group_key |
+------------+----+--------------+--------+-----------+
| 2023-03-01 |  1 |            1 |      0 |         0 | (1st instance of status=0)
| 2023-03-02 |  2 |            2 |      0 |         0 |
| 2023-03-03 |  3 |            3 |      0 |         0 |
| 2023-03-04 |  4 |            1 |      1 |         3 | (1st instance of status=1)
| 2023-03-05 |  5 |            2 |      1 |         3 |
| 2023-03-06 |  6 |            4 |      0 |         2 | (4th instance of status=0)
| 2023-03-07 |  7 |            5 |      0 |         2 |
| 2023-03-08 |  8 |            4 |      1 |         4 |
| 2023-03-09 |  9 |            5 |      1 |         4 |
| 2023-03-10 | 10 |            6 |      1 |         4 |
+------------+----+--------------+--------+-----------+
```

We must first understand how consecutive data points work in conjunction with the `DENSE_RANK()` function. When consecutive data points (e.g. consecutive dates) increment step by step with the index column `rn`, it returns a unique `group_key` for that series of consecutive rows.

Looking at the last 3 row from the example table. When we subtract `rn_by_status` from `rn` and there is a consecutive sequence (8 minus 4, 9 minus 5, 10 minus 6) we get the same `group_key=4`. 

## How can there by duplicate groups

> Example of unintended duplicate groups

```
+------------+----+--------------+--------+-----------+-------------------+
| dates      | rn | rn_by_status | status | group_key | correct_group_key |
+------------+----+--------------+--------+-----------+-------------------+
| 2023-03-01 |  1 |            1 |      0 |         0 |                 0 |
| 2023-03-02 |  2 | ---------- 1 | ---- 1 | ------- 1 | -------------- 11 |
| 2023-03-03 |  3 | ---------- 2 | ---- 0 | ------- 1 | -------------- 10 |
| 2023-03-04 |  4 | ---------- 3 | ---- 0 | ------- 1 | -------------- 10 |
| 2023-03-05 |  5 |            2 |      1 |         3 |                31 |
| 2023-03-06 |  6 |            3 |      1 |         3 |                31 |
| 2023-03-07 |  7 |            4 |      1 |         3 |                31 |
| 2023-03-08 |  8 |            4 |      0 |         4 |                40 |
| 2023-03-09 |  9 |            5 |      0 |         4 |                40 |
| 2023-03-10 | 10 |            6 |      1 |         4 |                41 |
+------------+----+--------------+------ -+-----------+-------------------+
```

By subtracting `rn_by_status` from `rn`, the `group_key` should be unique to the island and group. This should work in most situations. 

Whoever, by random change, the order of the data may cause duplicate `group_keys`.

## Best practice to avoid duplicate group_keys

```sql
status+10*(...) AS group_key
```

To fix this, append the status column to your `correct_group_key`. Eeliang suggest adding the main identifier (`status` or `user_id`) in the ones place. This ensures that we always know where to identify the status.

References:

- [Resolving duplicate group_keys](https://binhhoang.io/blog/gaps-and-islands/)