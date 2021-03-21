---
author: "Alper Yildirim"
date: 2018-03-01
title: How to calculate weighted average in PostgreSQL like a boss
---

As the title says it all, this post is on how to perform weighted average in SQL, which is not a natively-supported aggregation function like `sum()`, `avg()` and so on. In order to do so, we will exploit some math and window functions 
in a cunning 😈, yet logical way 🤓, than trying to pull out a physical solution. 

In normal data engineering, this approach should **not** be preferred as it can be super hard to maintain, not scalable and confusing to other people
but nevertheless this is a very useful, simple and intuitive hack I figured out myself for some quick data analysis and getting insights.

Let's tackle this with a simple and tangible example. Imagine we are some small shop that buys fruits at different prices (due to fluctuations and different retailers) and we are interested in knowing
what we are paying for one unit of each different fruit or item in average so that we can figure out what price that we should be selling the item at. 

Let's say we bought **1000** lemons from retailer **A** at price **$0.5** each, **78** lemons from retailer **B** at **$1.7** each and **240** lemons from retailer **C** at **2$** each, then what we have paid for **one**
lemon in average would be calculated using **weighted average mean** such as:

\begin{align}
\frac{1000 * 0.5 + 78 * 1.7 + 240 * 2}{1000 + 78 + 240} = 0.844 \space \space \space \space \space \space \space \space \space \space (1)
\end{align}

Coming back to our shop, assume we have the following arbitrary table where we have the information about the product, quantity and the unit price of the item bought from the retailer.

```
| id | product | quantity | price |
|----|---------|----------|-------|
|  1 |      🍋 |     1000 |   0.5 |
|  2 |      🍋 |       78 |   1.7 |
|  3 |      🍋 |      240 |     2 |
|  4 |      🍎 |       13 |     1 |
|  5 |      🍎 |       45 |  0.75 |
|  6 |      🍎 |      800 |   0.2 |
|  7 |      🍎 |       90 |     2 |
|  8 |      🍊 |      125 |   1.2 |
|  9 |      🍊 |      400 |     3 |
| 10 |      🍊 |      900 |     2 |

```

and we want to know what is the average price we are paying for the individual items. 

We open our sql ide and quickly type the following query

```
SELECT product, avg(price) FROM products GROUP BY product
``` 

and we get this result

```
| product |               avg |
|---------|-------------------|
|      🍋 |               1.4 |
|      🍎 |            0.9875 |
|      🍊 | 2.066666666666667 |
```

But wait, this is not right at all! 

Because what we calculated in **(1)** as the unit price for one 🍋 from different retailers was **0.844** and it does not match with **1.4** from the above aggregated result which used
the natively implemented `avg()`.

This happened, because the unit prices for the same items were just averaged based on the number of rows but instead they should have been **weight averaged**.

Ok, but how are we going to weight-average then in sql way? Bunch of sub-queries? Some crazy joins? Moving the data to some excel sheet? or a jupyter notebook or python code? 

No! There is a simpler way to do this for the very first hand analysis.
But before we get our hands dirty, we need to delve into a bit of background [theory] on weighted arithmetic mean. (Don't worry we will simplify it later!) 

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

I highlighted the above formula because that is the part we will tap in and logically hack
PostgreSql to have a weighted arithmetic mean using the power of beautiful window functions. (Window functions are just one of the reasons I love Postgres!)

This seems a lot of math at the first sight and may seem frustrating but all the steps until the highlight are nothing but just normalizing and scaling of individual components over the total to 1.
In other words or in terms of our lemons, this is saying, "

Ok, but how do we convert all of this to SQL in order to solve our problem then? 



```sql
SELECT DISTINCT 
T.product,
avg(price) OVER (PARTITION BY T.product) as native_avg,
sum(T.w_i::double precision * T.price / T.w_j) OVER (PARTITION BY T.product) AS weighted_average
FROM (
SELECT 
  product,
  quantity,
  price,
  sum(quantity) OVER (PARTITION BY product, quantity) AS w_i,
  sum(quantity) OVER (PARTITION BY product) AS w_j
FROM products
) T
```

As you can see, we have created one subquery in which we pre-calculate the `w_i` and `w_j` and then in the upper part we are multiplying our value (x) with `w_i` and divide it by `w_j` and run it for the whole data set 
and this is how we achieve the weighted arithmetic mean

and after we run the query we get (normal avg. is for comparision):


```
| product |        native_avg |   weighted_average |
|---------|-------------------|--------------------|
|      🍊 | 2.066666666666667 | 2.2105263157894735 |
|      🍋 |               1.4 | 0.8441578148710167 |
|      🍎 |            0.9875 | 0.4079641350210971 |
```

In real-case scenarios, you may need to deal with `NULL` or `zero` values, therefore you might need to use functions like `COALESCE` and `NULLIF`: 

```sql
sum(T.w_i::double precision * T.price / COALESCE(NULLIF(T.w_j, 0), 1::bigint)::double precision) OVER (PARTITION BY T.product) AS weighted_average,
```

With the code below, you can build the schema and the data set and try this yourself and play around with the idea on [SqlFiddle]! Please make sure you choose Postgres as the database engine there!

```sql
CREATE TABLE products
(
  id serial primary key,
  product varchar,
  quantity int,
  price numeric(6,2)
);

INSERT INTO products(product, quantity, price)
VALUES
('🍋', 1000, 0.5),
('🍋', 78, 1.7),
('🍋', 240, 2),
('🍎', 13, 1);
('🍎', 45, 0.75),
('🍎', 800, 0.2),
('🍎', 90, 2),
('🍊', 125, 1.2),
('🍊', 400, 3),
('🍊', 900, 2);
```

Pretty awesome, right? 

Please leave your comments right below for any feedback, questions or discussions!

[theory]: https://en.wikipedia.org/wiki/Weighted_arithmetic_mean
[window functions]: https://en.wikipedia.org/wiki/Weighted_arithmetic_mean
[SqlFiddle]: http://sqlfiddle.com/