---
author: "Alper Yildirim"
date: 2018-03-01
title: How to calculate weighted average in PostgreSQL like a boss
---

As the title says it all, this post is on how to perform weighted average in SQL, which is not supported natively, by exploiting some math and window functions in a cunning 😈 yet logical way 🤓
than trying to pull out a physical solution. 

In normal data engineering, this approach should be avoided at all costs as it will can be super hard to maintain
but nevertheless this is a very useful trick for quick data analysis.

Let's use a simple example and imagine that there is we have a products table where we are book-keeping all the products
we are buying. 

```
id         | product     | quantity   | month        | price
-----------------------------------------------------------
1          | lemon       | 1000       | august      | 1000 
2          | apples      | 13          | march       | 50   
3          | oranges     | 125          | march       | 300    
4          | potatoes    | 45          | february    | 100   
5          | lemon       | 78          | april       | 125
6          | jumper      | 400          | september   | 1000
7          | shirt       | 240          | august      | 300
8          | jacket      | S          | july        | wo
9          | pants       | M          | may         | m
10         | sweater     | S          | june        | 
11         | shirt       | M          | october     |
....
....
```



```
id         | disease     | age_group  | color       | gender       | material     | total_case  | deaths 
-----------------------------------------------------------------------------------------------------------------------
1          | jacket      | M          | black       | men          | leather      | 10          | 7
2          | jacket      | S          | brown       | women        | cotton       | 100         | 40
3          | pants       | S          | blue        | men          | cotton       | 10          | 1
4          | pants       | L          | brown       | men          | polymer      | 125         | 45
5          | pants       | M          | black       | women        | polymer      | 300         | 13
6          | jumper      | L          | blue        | women        | cotton       | 1000        | 100
7          | shirt       | S          | black       | men          | cotton       | 2000        | 100
8          | jacket      | S          | blue        | women        | leather      | 400         | 100
9          | pants       | M          | black       | men          | polymer      | 15          | 10
10         | sweater     | S          | black       | women        | cotton       | 78          | 50
11         | shirt       | M          | orange      | unisex       | cotton       | 45          | 33
```
and we want to know

What is the average return ratio of the shirts that the company is selling?

`SELECT avg(return_rate) FROM product_returns WHERE product = 'shirt'`

Easy right?

But, this is sometimes not quite right, because averaging means that the details will be lost. The more details we have for the data,
the more 

So, an example to this will be
 



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
    T.product_size,
    T.color,
    T.category,
    T.material,
    avg(return_rate) as avg_return_rate,
    sum(T.w_i::double precision * T.return_rate) / COALESCE(NULLIF(sum(T.w_i), 0::numeric), 1::bigint::numeric)::double precision AS avg_return,
    sum(T.w_i::double precision * T.return_rate / COALESCE(NULLIF(T.w_j, 0), 1::bigint)::double precision) OVER (PARTITION BY T.product, T.product_size, T.color, T.categoryy) AS weighted_average_return,
   FROM ( SELECT 
            product,
            size,
            color,
            category,
            material,
            return_rate,
            sum(nb_of_returns) OVER (PARTITION BY product, product_size, color, category, material) AS w_i,
            sum(nb_of_returns) OVER (PARTITION BY product, product_size, color, category) AS w_j,
            --sum(nb_of_returns) OVER (PARTITION BY os_version, app_version, marketing_name, model, event) AS w_i,
            --sum(nb_of_returns) OVER (PARTITION BY os_version, marketing_name, model, event) AS w_j,
           FROM products
        ) T
  GROUP BY T.product, T.product_size, T.color, T.category, T.material, T.avg_return_rate, T.w_i, T.w_j
  ORDER BY T.product, T.product_size;
```

[theory]: https://en.wikipedia.org/wiki/Weighted_arithmetic_mean
[window functions]: https://en.wikipedia.org/wiki/Weighted_arithmetic_mean