# How to get unique `group_keys`

```
+------------+----+--------------+-----------+--------+
| dates      | rn | rn_by_status | group_key | status |
+------------+----+--------------+-----------+--------+
| 2023-03-01 |  1 |            1 |         0 |      0 | (1st instance of status=0)
| 2023-03-02 |  2 |            2 |         0 |      0 |
| 2023-03-03 |  3 |            3 |         0 |      0 |
| 2023-03-04 |  4 |            1 |         3 |      1 | (1st instance of status=1)
| 2023-03-05 |  5 |            2 |         3 |      1 |
| 2023-03-06 |  6 |            4 |         2 |      0 | (4th instance of status=0)
| 2023-03-07 |  7 |            5 |         2 |      0 |
| 2023-03-08 |  8 |            4 |         4 |      1 |
| 2023-03-09 |  9 |            5 |         4 |      1 |
| 2023-03-10 | 10 |            6 |         4 |      1 |
+------------+----+--------------+-----------+--------+
```

We must first understand how consecutive data points work in conjunction with the `DENSE_RANK()` function. When consecutive data points (e.g. consecutive dates) increment step by step with the index column `rn`, it returns a unique `group_key` for that series of consecutive rows.

Looking at the last 3 row from the example table. When we subtract `rn_by_status` from `rn` and there is a consecutive sequence (8 minus 4, 9 minus 5, 10 minus 6) we get the same `group_key=4`. 

References:

- [Dense_rank vs row_number](https://codingsight.com/similarities-and-differences-among-rank-dense_rank-and-row_number-functions/).
