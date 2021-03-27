---
author: "Alper Yildirim"
date: 2018-03-01
title: Weighted Average with Sql
---

This post is on how to calculate weighted average on a particular dataset in Sql.

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

```sql
SELECT product, 
sum(price * quantity) / sum(quantity)
FROM products
GROUP BY products
```

and that is it!


> Output 

```
| product |   weighted_average |
|---------|--------------------|
|      🍎 | 0.4079641350210971 |
|      🍊 |  2.210526315789474 |
|      🍋 | 0.8441578148710167 |
```


## ⛱ Sandbox Code

With the code below, you can build the schema and the data set and try this yourself on [SqlFiddle]!

```sql
CREATE TABLE products
(
  id serial primary key,
  product varchar,
  quantity int,
  price decimal(4,2)
);

INSERT INTO products(product, quantity, price)
VALUES
('🍋', 1000, 0.5),
('🍋', 78, 1.7),
('🍋', 240, 2),
('🍎', 13, 1),
('🍎', 45, 0.75),
('🍎', 800, 0.2),
('🍎', 90, 2),
('🍊', 125, 1.2),
('🍊', 400, 3),
('🍊', 900, 2);
```

Thank you!


[SqlFiddle]: http://sqlfiddle.com/