# Avoid error of duplicate group_keys

### Prerequisite: get unique `group_keys`

Ensure that you understand how to [get group_keys from `dense_rank`](#how-to-get-unique-group_keys).

<aside class="success">
Why do we use <a href="#how-to-get-unique-group_keys"><code>DENSE_RANK()</code> over <a href=""><code>ROW_NUMBER()</code></a>.
</aside>

## How can there by duplicate groups

> Example of unintended duplicate groups

```
+------------+----+--------------+-------------------+-----------+--------+
| dates      | rn | rn_by_status | correct_group_key | group_key | status |
+------------+----+--------------+-------------------+-----------+--------+
| 2023-03-01 |  1 |            1 |                 0 |         0 |      0 |
| 2023-03-02 |  2 | ---------- 1 | -------------- 11 | ------- 1 | ---- 1 |
| 2023-03-03 |  3 | ---------- 2 | -------------- 10 | ------- 1 | ---- 0 |
| 2023-03-04 |  4 | ---------- 3 | -------------- 10 | ------- 1 | ---- 0 |
| 2023-03-05 |  5 |            2 |                31 |         3 |      1 |
| 2023-03-06 |  6 |            3 |                31 |         3 |      1 |
| 2023-03-07 |  7 |            4 |                31 |         3 |      1 | 
| 2023-03-08 |  8 |            4 |                40 |         4 |      0 |
| 2023-03-09 |  9 |            5 |                40 |         4 |      0 |
| 2023-03-10 | 10 |            6 |                41 |         4 |      1 |
+------------+----+--------------+-------------------+-----------+--------+
```

By subtracting `rn_by_status` from `rn`, the `group_key` should be unique to the island and group. This should work in most situations. 

Whoever, by random change, the order of the data may cause duplicate `group_keys`.

## Best practice to avoid duplicate group_keys

```sql
...
          (rn-rn_by_status) AS group_key,
status+10*(rn-rn_by_status) AS correct_group_key
...
```

To fix this, append the status column to your `correct_group_key`. Eeliang suggest adding the main identifier (`status` or `user_id`) in the ones place. This ensures that we always know where to identify the status.

References:

- [Resolving duplicate group_keys](https://binhhoang.io/blog/gaps-and-islands/)
