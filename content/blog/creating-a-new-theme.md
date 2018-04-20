---
author: "Alper Yildirim"
date: 2018-03-01
title: Weighted average with Sql
---


## Introduction

For some, SQL is only some **`SELECT * FROM WHERE table`** code for getting some information from a database. However, apart from having a simple and intuitive syntax, the language allows to do much more powerful, and complex stuff.

This post shows how to manipulate the data using weighted arithmetic mean, which is not a native aggregation method like **avg(), max() or sum()**

First off, let's start with some [definition].

So, let's suppose we have a set of variables:

\begin{align}
x\_{1}, \dots, x\_{n}
\end{align}

and a set of weights:
\begin{align}
w\_{1}, \dots, w\_{n}
\end{align}

Then the weighted average is:
\begin{align}
\bar{x}=\frac{\sum_{i=1}^{n}{w_ix_i}}{\sum_i{w_i}}
\end{align}

The formulas are simplified when the weights are normalized such that they sum up to **1**,


So, how can we apply all this to a real world scenario? 

   Name | Age  | Subject| Note | Track
--------|------|--------|------|---------
    Bob | 27   | Math   | 10   | Science
  Alice | 23   | Bio    | 20   | Science
   Luke | 27   | Chem   | 30   | Science
  Alice | 23   | Bio    | 50   | Social
    Bob | 27   | Math   | 10   | Science
  Alice | 23   | Bio    | 20   | Social
   Luke | 27   | Chem   | 30   | Social
  Alice | 23   | Bio    | 50   | Social


```sql
 SELECT 
    T.os_version,
    T.app_version,
    T.marketing_name,
    T.model,
    T.event,
    avg(success_rate),
    sum(T.w_i::double precision * T.success_rate) / COALESCE(NULLIF(sum(T.w_i), 0::numeric), 1::bigint::numeric)::double precision AS avg_score,
    sum(T.w_i::double precision * T.success_rate / COALESCE(NULLIF(T.w_j, 0), 1::bigint)::double precision) OVER (PARTITION BY T.os_version, T.marketing_name, T.model, T.event) AS weighted_average_score,
   FROM ( SELECT os_version,
            app_version,
            marketing_name,
            model,
            event,
            success_rate,
            sum(total_events) OVER (PARTITION BY os_version, app_version, marketing_name, model, event) AS w_i,
            sum(total_events) OVER (PARTITION BY os_version, marketing_name, model, event) AS w_j,
           FROM device_table
        ) T
  GROUP BY T.os_version, T.app_version, T.marketing_name, T.model, T.event, T.success_rate, T.w_i, T.w_j
  ORDER BY T.marketing_name, T.model;
```

[definition]: https://en.wikipedia.org/wiki/Weighted_arithmetic_mean