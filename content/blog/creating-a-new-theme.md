---
author: "Alper Yildirim"
date: 2018-03-01
title: Weighted average in PostgreSQL
---


SQL is a very intuitive and straight-forward query language. It allows anyone to write queries from very simple ones like **`SELECT * FROM users WHERE id = 10`** to very complex and frustrating ones that contain joins, filters, aggregations, and even mathematical operations.

This post is on how to apply *weighted arithmetic average* in PostgreSQL, which is not a part of SQL's native aggregate functions like **avg()**, **max()** or **sum()**.

Let's assume that we have a dataset/table such as below which shows different products that our e-commerce company **A** is selling in the market. Attached to 




```
id         | product     | size       | color       | category     | material     | return_rate | nb_of_returns
-----------------------------------------------------------------------------------------------------------------------
1          | jacket      | M          | black       | men          | leather      | 0.70        | 7
2          | jacket      | S          | brown       | women        | cotton       | 0.40        | 40
3          | pants       | S          | blue        | men          | cotton       | 0.15        | 1
4          | pants       | L          | brown       | men          | polymer      | 0.05        | 45
5          | pants       | M          | black       | women        | polymer      | 0.30        | 13
6          | sweater     | L          | blue        | women        | cotton       | 0.02        | 100
7          | shirt       | S          | black       | men          | cotton       | 0.01        | 200
8          | jacket      | S          | blue        | women        | leather      | 0.11        | 1000
9          | pants       | M          | black       | men          | polymer      | 0.12        | 10
10         | sweater     | S          | black       | women        | cotton       | 0.19        | 50
```




Before we start, we need a bit of background [theory] on weighted arithmetic average. Let's assume that we have a set of variables:

\begin{align}
x\_{1}, \dots, x\_{n}
\end{align}

and a set of weights:
\begin{align}
w\_{1}, \dots, w\_{n}
\end{align}

Then the weighted arithmetic average is calculated as the sum of the products of these weights with the variables over the sum of these weights:
\begin{align}
\bar{x}=\frac{\sum_{i=1}^{n}{w_ix_i}}{\sum_i{w_i}}
\end{align}

The formulas are simplified when the weights are normalized such that they sum up to **1**,

So, how can we apply all this to a real world scenario? 




```sql
 SELECT 
    T.product,
    T.size,
    T.color,
    T.category,
    T.material,
    avg(success_rate),
    sum(T.w_i::double precision * T.success_rate) / COALESCE(NULLIF(sum(T.w_i), 0::numeric), 1::bigint::numeric)::double precision AS avg_score,
    sum(T.w_i::double precision * T.success_rate / COALESCE(NULLIF(T.w_j, 0), 1::bigint)::double precision) OVER (PARTITION BY T.os_version, T.marketing_name, T.model, T.event) AS weighted_average_score,
   FROM ( SELECT 
            product,
            size,
            color,
            category,
            material,
            success_rate,
            sum(total_events) OVER (PARTITION BY os_version, app_version, marketing_name, model, event) AS w_i,
            sum(total_events) OVER (PARTITION BY os_version, marketing_name, model, event) AS w_j,
           FROM device_table
        ) T
  GROUP BY T.os_version, T.app_version, T.marketing_name, T.model, T.event, T.success_rate, T.w_i, T.w_j
  ORDER BY T.marketing_name, T.model;
```

[theory]: https://en.wikipedia.org/wiki/Weighted_arithmetic_mean
[window functions]: https://en.wikipedia.org/wiki/Weighted_arithmetic_mean