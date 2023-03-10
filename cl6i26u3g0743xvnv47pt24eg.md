---
title: "Case Study: Danny's Dinner"
datePublished: Sat Aug 06 2022 15:36:49 GMT+0000 (Coordinated Universal Time)
cuid: cl6i26u3g0743xvnv47pt24eg
slug: case-study-dannys-dinner
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1659787536519/EN33TFR6w.png
tags: postgresql, databases, sql, data-engineering

---

## Introduction

Danny seriously loves Japanese food so, at the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from its few months of operation but has no idea how to use their data to help them run the business.

## Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalized experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally, he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

- sales - The `sales` table captures all `customer_id` level purchases with a corresponding `order_date` and `product_id` information for when and what menu items were ordered.
- menu - The `menu` table maps the `product_id` to the actual `product_name` and `price` of each menu item.
- members - The final `members` table captures the `join_date` when a `customer_id` joined the beta version of the Danny’s Diner loyalty program.

You can inspect the entity relationship diagram and example data below.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659787693868/MAsvX9xIc.png align="left")

And here’s the sample data for all the tables:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659787797038/K6pfFONq4.png align="left")

> You can visit the [GitHub link here](https://github.com/Aditya-Gupta1/sql-case-study) to get your hands on the initialization script that’ll create all the required tables and populate them with the data, along with all the solutions.

## Questions

Let’s begin solving the case study:

**Ques-1: What is the total amount each customer spent at the restaurant?**

**Ans:** All the orders are present in the `sales` table and all the prices are present in the `menu` table. So we can join these 2 tables and group the result by `customer_id`. As the last step, we just need to add up the prices.
```SQL
SELECT prices.customer_id, SUM(prices.price) as total_amount_spent
FROM (sales s JOIN menu m ON s.product_id = m.product_id) prices
GROUP BY prices.customer_id
ORDER BY prices.customer_id;
```
*(ORDER BY isn't necessary. That just display's the data in a nice way!)*

Query Result:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659789734516/F0XwFsfop.png align="left")

**Ques-2: How many days has each customer visited the restaurant?**

**Ans:** We can group the `sales` table by `customer_id` and count the unique days on which the order was placed.
```SQL
SELECT customer_id, COUNT(DISTINCT order_date)
FROM sales
GROUP BY customer_id
ORDER BY customer_id;
```
Query Result:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659789839558/r0EdVx2Fy.png align="left")
**Ques-3: What was the first item from the menu purchased by each customer?**

**Ans:** To get the first item, we need to find the date of the first order placed by each customer. Once we get that, we just need to do a join with `menu` to get the product name.

Now, there a 2 ways that we can think of getting the earliest date. One is to group by `customer_id` and find the minimum `order_date` for each customer. And the second is to rank the orders **by** order date **for** each customer and then fetch the orders with rank 1. Let's see the second approach.

Note that the string aggregation used in the query is only to get all the products for a single user in a single column.
```SQL
WITH sale_rankings AS
	(
	SELECT customer_id, order_date, product_name,
	DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date) AS order_rank
	FROM sales JOIN menu ON sales.product_id = menu.product_id
	),
temp_view AS
	(
	SELECT customer_id, product_name
	FROM sale_rankings
	WHERE order_rank = 1
	GROUP BY customer_id, product_name
	)
SELECT customer_id, STRING_AGG(product_name, ', ') AS products
FROM temp_view
GROUP BY customer_id;
```
Query Result:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659790140639/BdlFcqDp0.png align="left")

**Ques-4: What is the most purchased item on the menu and how many times was it purchased by all customers?**

**Ans:** We can find the most purchased item by counting the occurrence of each product and picking the one which has maximum occurrence:
```SQL
SELECT product_id FROM sales GROUP BY product_id ORDER BY COUNT(*) DESC LIMIT 1;
```
Query Result:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659791041952/GOtfWZpei.png align="left")
Then we can use this to find the customers who purchased it:
```SQL
SELECT customer_id, COUNT(*)
FROM sales
WHERE product_id = 
  (SELECT product_id 
  FROM sales 
  GROUP BY product_id 
  ORDER BY COUNT(*) DESC LIMIT 1)
GROUP BY customer_id;
```
Query Result:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659790990425/V9KZ7ubwB.png align="left")

**Ques-5: Which item was the most popular for each customer?**

**Ans:** We have to find the number of times each product is purchased by each customer. This can be done by ranking each `customer_id` and `product_id` pair according to the frequency of each product.

Again, string aggregation is only for displaying the result consistently. Otherwise, there will be multiple rows for every customer if it has more than one popular item.
```SQL
WITH popular_products AS
(
	SELECT customer_id, product_id,
	DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(*) DESC) AS popularity
	FROM sales
	GROUP BY customer_id, product_id
)
SELECT pp.customer_id, 
STRING_AGG(m.product_name, ', ' ORDER BY m.product_name) AS popular_products_per_customer
FROM popular_products pp JOIN menu m USING(product_id)
WHERE pp.popularity = 1
GROUP BY pp.customer_id
```
Query Result:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659792018403/aSxKLHppt.png align="left")
**Ques-6: Which item was purchased first by the customer after they became a member?**

**Ans:** We will Join `members` with `sales` for orders whose `order_date` is greater than the `join_date`. This will give us all the orders that were placed after the members joined. Then we will rank the orders for each customer by `order_date`. This can be thought of as the number of days after the joining, that the order was placed. Hence, all the orders with rank 1 are our answer.
```SQL
SELECT customer_id, product_name FROM
(
	SELECT s.customer_id, s.product_id, 
	DENSE_RANK() OVER (PARTITION BY s.customer_id 
		ORDER BY s.order_date) AS days_after_joining
	FROM sales s JOIN members m 
	ON s.customer_id = m.customer_id 
	AND s.order_date >= m.join_date
) orders_after_joining 
JOIN menu using(product_id)
WHERE days_after_joining = 1
```
Query Result:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659792066569/lOn_JKQX4.png align="left")

**Ques-7: Which item was purchased just before the customer became a member?**

**Ans:** This question is similar to question 6. Now we will join the `member` and `sales` table on the condition that the `order_date` should be less than the `join_date`.
```SQL
select customer_id, string_agg(product_name, ', ') as item_purchased_before_membership
from (
select s.customer_id, s.product_id, 
dense_rank() over (partition by s.customer_id order by s.order_date desc) as days_after_joining
from sales s join members m 
on s.customer_id = m.customer_id 
and s.order_date < m.join_date
) orders_after_joining join menu using(product_id)
where days_after_joining = 1
group by customer_id
```
Query Result:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659792319845/Rbb5Ri4aT.png align="left")

**Ques-8: What are the total items and amount spent for each member before they became a member?**

**Ans:** Similar to the above questions, take a join between `sales` and `members` based on the order and the join date. And then join the resulting table with `menu` to get the prices. And then just perform the aggregations as and when needed.
```SQL
SELECT  s.customer_id, 
COUNT(product_id) AS no_of_products, 
SUM(price) AS total_amount_spent
FROM sales s JOIN members m 
ON s.customer_id = m.customer_id 
AND s.order_date < m.join_date
JOIN menu using(product_id)
GROUP BY s.customer_id
```
Query Result:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659792956802/OQNfnXSY3.png align="left")

**Ques-9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier, how many points would each customer have?**

**Ans:** We will use `CASE WHEN` statements while doing the summation on the price.
```SQL
SELECT customer_id,
SUM(CASE WHEN product_id = 1 THEN 20*price ELSE 10*price END) AS points
FROM sales s JOIN menu m USING(product_id)
GROUP BY customer_id 
```
Query Result:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659793271178/IXcUni9LN.png align="left")

**Ques-10: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customers A and B have at the end of January?**

**Ans:** 
1. Join the `sale` and `member` table to get the orders for all the members.
2. Filter them to get all the orders by the end of January.
3. Apply the conditions as given in the question while summation of the prices.

```SQL
SELECT customer_id,
SUM(
CASE
	WHEN order_date BETWEEN join_date AND join_date + INTERVAL '7 day' 
		THEN 20*price
	WHEN product_id = 1 THEN 20*price
	ELSE 10*price
END
) AS points
FROM sales s JOIN members m USING(customer_id) JOIN menu USING(product_id)
WHERE s.order_date <= '2021-01-31'
GROUP BY customer_id
```
Query Results:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659793873183/7k6NlujEt.png align="left")

---

## Bonus Questions

**Ques-1: Write a SQL query to recreate the below table**

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659794018822/_c-_xJ32Q.png align="left")
**Ans:** So the table given in the question has 2 columns:
1. `price`: it contains the price of the product purchased.
2. `member`: it indicates whether the customer is a member *at the time of placing that order*.

Price can be fetched from the `menu` table directly. As far as the `member` column is concerned, there can be 3 cases:

1. The customer is not a member.
2. The customer is not a member at the time of purchase. They will be in the future.
3. The customer is the member.

We can use these 3 conditions to calculate the `member` column.
```SQL
SELECT customer_id, 
order_date, 
product_name, 
price,
(CASE 
	WHEN join_date IS NULL THEN 'N'
	WHEN order_date >= join_date THEN 'Y'
	ELSE 'N'
END) AS member
FROM sales s JOIN menu USING(product_id) LEFT JOIN members USING(customer_id)
```
Query Result:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659794455158/CCjIyCduI.png align="left")

**Ques-2: Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.**
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659794649402/XmUu2FMuM.png align="left")
**Ans:** Things may get messy here😅

So as we can see from the image, all the things we did in the previous question will remain the same, we just have one extra column of ranking. The ranking can be calculated for each customer who is a member, otherwise, it will be null.
```SQL
SELECT customer_id, 
order_date, 
product_name, 
price,
(CASE 
	WHEN join_date IS NULL THEN 'N'
	WHEN order_date >= join_date THEN 'Y'
	ELSE 'N'
END) AS member,
CASE 
	WHEN join_date IS NULL OR order_date < join_date THEN NULL
	ELSE DENSE_RANK() OVER ( PARTITION BY 
	CASE 
		WHEN join_date IS NULL OR order_date < join_date THEN NULL 
		ELSE customer_id 
    END ORDER BY order_date)
END AS ranking
FROM sales s JOIN menu USING(product_id) 
LEFT JOIN members USING(customer_id)
```

---

And this brings us to the end of the case study. I hope you learned something and enjoyed it. Thanks for reading!

Until next time👋

## References

- Case Study is taken from here: [“Danny’s Dinner”](https://8weeksqlchallenge.com/case-study-1/)



























































`