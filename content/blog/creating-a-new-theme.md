---
author: "Alper Yildirim"
date: 2018-03-01
title: Weighted average in PostgreSQL
---

In this post, I will demonstrate how to do a weighted arithmetic mean or average using only sql in postgres.

Sql is an intuitive, straight-forward and yet a very powerful query language. 
It allows simple operations such as **`SELECT name FROM users WHERE id = 10`** to
complex and frustrating ones that contain many joins, aggregations, and even mathematical operations.
 
Postgres also supports many functions such as **avg()**, **max()** or **sum()**. 
However, when it comes to more complex aggregations such as weighted arithmetic average, it does not have a native support.

In this post I will demonstrate how to tackle weighted average problem with the use of a logical method than doing any
of these. In doing so, I will exploit the rule of normalization when doing weighted arithmetic average.

Let's assume that we have this table below, which shows different products that our company sells. 

Business question:

What is the average return ratio of the shirts that the company is selling?

`SELECT avg(return_rate) FROM product_returns WHERE product = 'shirt'`



Easy right?

But 


```
id         | product     | size       | color       | category     | material     | return_rate | nb_of_returns
-----------------------------------------------------------------------------------------------------------------------
1          | jacket      | M          | black       | men          | leather      | 0.70        | 7
2          | jacket      | S          | brown       | women        | cotton       | 0.40        | 40
3          | pants       | S          | blue        | men          | cotton       | 0.15        | 1
4          | pants       | L          | brown       | men          | polymer      | 0.05        | 45
5          | pants       | M          | black       | women        | polymer      | 0.30        | 13
6          | jumper      | L          | blue        | women        | cotton       | 0.02        | 100
7          | shirt       | S          | black       | men          | cotton       | 0.01        | 100
8          | jacket      | S          | blue        | women        | leather      | 0.11        | 1000
9          | pants       | M          | black       | men          | polymer      | 0.12        | 10
10         | sweater     | S          | black       | women        | cotton       | 0.19        | 50
...          ...           ...          ...           ...            ...            ...           ...
100000     | shirt       | M          | orange      | unisex       | cotton       | 0.33        | 3300
```




Before we start, we need a bit of background [theory] on weighted arithmetic average. 

Let's assume that we have a set of variables:

\begin{equation}
x\_{1}, \dots, x\_{n} 
\end{equation}

and a set of weights:
\begin{equation}
w\_{1}, \dots, w\_{n}
\end{equation}

Then the weighted arithmetic average is calculated as the sum of the products of these weights with the variables over the sum of these weights:
\begin{align}
\bar{x}=\frac{\sum_{i=1}^{n}{w_ix_i}}{\sum_{i=1}^{n}{w_i}}
\end{align}
z
The formulas are simplified when the weights are normalized such that they sum up to 1,

\begin{align}
\sum_{i-1}^{n}{w'_i}=1
\end{align}

For such normalized weights the weighted mean is then:
\begin{align}
\bar{x}=\sum_{i-1}^{n}{w'_i}{x_i}
\end{align}

Note that one can always normalize the weights by making the following transformation on the original weights:
\begin{align}
w'\_{i}=\frac{{w_i}}{\sum_{j-1}^{n}{w_j}}
\end{align}

Using the normalized weight yields the same results as when using the original weights:

\begin{align}
\bar{x}=\sum_{i=1}^{n}{w'_i}{x_i}=\bbox[yellow]{\sum_{i=1}^{n}{\frac{{w_i}}{\sum_{j=1}^{n}{w_j}}x_i}}=\frac{\sum_{i=1}^{n}{w_i}{x_i}}{\sum_{j=1}^{n}{w_j}}=\frac{\sum_{i=1}^{n}{w_ix_i}}{\sum_{i=1}^{n}{w_i}}
\end{align}

i highlighted the above formula because that is the part we will tap in and logically hack
postgreSql to have a weighted arithmetic mean using the beatiful and powerful window functions. Window functions
are just one of the reasons I love Postgres.

Ok, but how do we apply this to our problem then? 




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