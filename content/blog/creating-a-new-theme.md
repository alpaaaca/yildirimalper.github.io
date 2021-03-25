---
author: "Alper Yildirim"
date: 2018-03-01
title: How to calculate weighted average in PostgreSQL like a boss
---

This post is on how to use **weighted average** and some math to perform weighted average on a particular dataset in Sql.

## 🍊 Let’s tackle this with a simple and tangible example.
Imagine we are some small shop that buys fruits at different prices (due to fluctuations and different retailers).


We do the book-keeping for the items that we buy from different retailers with the following arbitrary table. We have the information about the product, quantity and the unit price.

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

We want to calculate the price we are paying for one unit of each different fruit.

To do so for one unit of 🍋 , we would use the weighted average mean such as:

\begin{align}
\frac{1000 * 0.5 + 78 * 1.7 + 240 * 2}{1000 + 78 + 240} = 0.844 \space \space \space \space \space \space \space \space \space \space (1)
\end{align}

---------------------------

## 𝌣 Solution — Code
In order to use window functions simply for this purpose, an easy way will be to use the normalised weighted average formula below!

If you are interested in how this is derived, check the “Hacking the math” below for the steps.
* We pre-calculate `w_i` and `w_j` in subquery
* and multiply the unit price (`x_i`) with `w_i` and divide this by `w_j` over a sum!

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

and that is it!

_Window functions are just one of the reasons why I love Postgres!_

> Output (native avg. is for comparison):

```
| product |        native_avg |   weighted_average |
|---------|-------------------|--------------------|
|      🍊 | 2.066666666666667 | 2.2105263157894735 |
|      🍋 |               1.4 | 0.8441578148710167 |
|      🍎 |            0.9875 | 0.4079641350210971 |
```

This time our calculation as the unit price for one 🍋 is in-line with the above result **0.8441578148710167** and we can be assured that our calculations are correct!

For people who want to dig deeper into a bit of [theory] on weighted arithmetic mean and how the math was hacked for this particular solution, check it below.

--------------------

## 🧮 Hacking the math

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

The highlighted part of the above formula is the part we tapped in and logically hacked PostgreSql!

--------------------

## 🤯 Or in other words…

The total number of lemons is **1318**, so our normalization would be

\begin{align}
\frac{100}{1318}
+
\frac{78}{1318}
+
\frac{240}{1318} = 1
\end{align}

---------------

## ⛱ Sandbox Code

With the code below, you can build the schema and the data set and try this yourself on [SqlFiddle]!

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

There are also other ways to go about this in the Sql way (some simpler and some more complicated) but using this formula and window functions in this type of dataset allows more control over the columns and lets one to come up with any sort of normalisation using different columns.

Please leave your comments right below for any feedback, questions or discussions!

Thank you!


[theory]: https://en.wikipedia.org/wiki/Weighted_arithmetic_mean
[window functions]: https://en.wikipedia.org/wiki/Weighted_arithmetic_mean
[SqlFiddle]: http://sqlfiddle.com/