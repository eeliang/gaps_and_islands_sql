# Avoid error of overalapping groups

```
+------------+----+--------------+-----------+--------+-------------------+
| dates      | rn | rn_by_status | group_key |  status |correct_group_key |
+------------+----+--------------+-----------+--------+-------------------+
| 2023-03-01 |  1 |            1 |         0 |      0 |                 0 |
| 2023-03-02 |  2 | ---------- 1 | ------- 1 | ---- 1 | -------------- 11 |
| 2023-03-03 |  3 | ---------- 2 | ------- 1 | ---- 0 | -------------- 10 |
| 2023-03-04 |  4 | ---------- 3 | ------- 1 | ---- 0 | -------------- 10 |
| 2023-03-05 |  5 |            2 |         3 |      1 |                31 |
| 2023-03-06 |  6 |            3 |         3 |      1 |                31 |
| 2023-03-07 |  7 |            4 |         3 |      1 |                31 |
| 2023-03-08 |  8 |            4 |         4 |      0 |                40 |
| 2023-03-09 |  9 |            5 |         4 |      0 |                40 |
+------------+----+--------------+-----------+--------+-------------------+
```

By subtracting `rn_by_status` from `rn`, the `group_key` should belong in the same grouping only if they are consecutive and belong to the same status (at least that is what you want if there were no bug). In other words, by using PARTITION BY status, the rankings are incremented for each status; this creates the incremental offset that we need so that when ranking is subtracted from id, we get our desired groupings.

But by chance, the order of the data may cause overlapping `group_keys`.

## Best practice to avoid duplicate group_keys

```sql
status+10*(...) AS group_key
```

To fix this, append the status column to your `correct_group_key`. Eeliang suggest adding the main identifier (`status` or `user_id`) in the ones place. This ensures that we always know where to identify the status.

References:

- [Resolving duplicate group_keys](https://binhhoang.io/blog/gaps-and-islands/)