# 8-Week-SQL-Challenge

## Week 1
*Dannys Diner*

**Introduction**

[<img src="images/intro.png" alt="missing">](https://8weeksqlchallenge.com/case-study-1/)

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.
[Link](https://8weeksqlchallenge.com/case-study-1/)

---

*DATA*

```sql
    CREATE SCHEMA dannys_diner;
    SET search_path = dannys_diner;
    
    CREATE TABLE sales (
      "customer_id" VARCHAR(1),
      "order_date" DATE,
      "product_id" INTEGER
    );
    
    INSERT INTO sales
      ("customer_id", "order_date", "product_id")
    VALUES
      ('A', '2021-01-01', '1'),
      ('A', '2021-01-01', '2'),
      ('A', '2021-01-07', '2'),
      ('A', '2021-01-10', '3'),
      ('A', '2021-01-11', '3'),
      ('A', '2021-01-11', '3'),
      ('B', '2021-01-01', '2'),
      ('B', '2021-01-02', '2'),
      ('B', '2021-01-04', '1'),
      ('B', '2021-01-11', '1'),
      ('B', '2021-01-16', '3'),
      ('B', '2021-02-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-07', '3');
     
    
    CREATE TABLE menu (
      "product_id" INTEGER,
      "product_name" VARCHAR(5),
      "price" INTEGER
    );
    
    INSERT INTO menu
      ("product_id", "product_name", "price")
    VALUES
      ('1', 'sushi', '10'),
      ('2', 'curry', '15'),
      ('3', 'ramen', '12');
      
    
    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );
    
    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');
```

<img src="images/w1ERD.png" alt="missing">

---

*Solution*

1. What is the total amount each customer spent at the restaurant?

```sql
SELECT
  	s.customer_id,
    SUM(m.price) as Total
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id
```
---

2. How many days has each customer visited the restaurant?

```sql
SELECT 
	customer_id,
    COUNT(DISTINCT order_date) AS "Visited times"
FROM dannys_diner.sales
GROUP BY customer_id
```

---

-- 3. What was the first item from the menu purchased by each customer?

```sql
SELECT 
	DISTINCT sales.customer_id,
    menu.product_name
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id
WHERE sales.order_date = '2021-01-01'
limit 3
```

---

-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT 
	m.product_name,
    COUNT(s.product_id) as times
FROM dannys_diner.sales AS S
INNER JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
GROUP BY m.product_name, s.product_id
ORDER by times DESC
LIMIT 1
```

---

-- 5. Which item was the most popular for each customer?
```sql
WITH temp as (SELECT customer_id, product_name
FROM dannys_diner.sales as s
JOIN dannys_diner.menu as m ON s.product_id = m.product_id)
SELECT customer_id, product_name,  count(product_name) 
FROM temp
GROUP BY customer_id, product_name
ORDER BY count(product_name) DESC
```

---

-- 6. Which item was purchased first by the customer after they became a member?
```sql
SELECT DISTINCT ON (order_date) s.customer_id, order_date, product_name, join_date
FROM dannys_diner.sales as s
JOIN dannys_diner.members as m ON s.customer_id = m.customer_id
JOIN dannys_diner.menu as men ON s.product_id = men.product_id
WHERE join_date <= order_date
ORDER BY order_date
LIMIT 2
```

---

-- 7. Which item was purchased just before the customer became a member?
```sql
SELECT DISTINCT ON (s.customer_id) s.customer_id, order_date, product_name, join_date 
FROM dannys_diner.sales as s
Join dannys_diner.members as mem ON s.customer_id = mem.customer_id
JOIN dannys_diner.menu as m ON s.product_id = m.product_id
WHERE join_date > order_date
```

---

-- 8. What is the total items and amount spent for each member before they became a member?
```sql
WITH temp as (SELECT s.customer_id, s.product_id, price
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
WHERE order_date < join_date)

SELECT customer_id as Members, count(product_id) as "Products bought", sum(price)
FROM temp
GROUP BY customer_id
```

---

-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
WITH temp AS (
SELECT customer_id, s.product_id, product_name, price
FROM dannys_diner.sales as s
JOIN dannys_diner.menu as m ON s.product_id = m.product_id)

SELECT DISTINCT ON (customer_id) customer_id, 
SUM(CASE WHEN product_id = 1 THEN 20*price ELSE 10*price END) AS point
FROM temp
GROUP BY temp.customer_id
```

---

-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
SELECT s.customer_id, SUM(CASE WHEN ABS(order_date-join_date) <= 7 THEN price*20 ELSE price*10 END) AS "points earned"
FROM dannys_diner.sales AS s
Join dannys_diner.menu AS m ON s.product_id = m.product_id
JOIN dannys_diner.members as mem ON s.customer_id = mem.customer_id
WHERE EXTRACT(MONTH FROM order_date) < 2
GROUP BY s.customer_id
```

---

*Bonus Question*

```sql
SELECT s.customer_id, order_date, product_name, price, 
	CASE 
    	WHEN mem.join_date > s.order_date THEN 'N'
    	WHEN mem.join_date <= s.order_date THEN 'Y'
    	ELSE 'N'
   	END AS members
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members AS mem ON mem.customer_id = s.customer_id
ORDER BY s.customer_id, order_date


WITH temp AS (SELECT s.customer_id, order_date, product_name, price, 
	CASE 
    	WHEN mem.join_date > s.order_date THEN 'N'
    	WHEN mem.join_date <= s.order_date THEN 'Y'
    	ELSE 'N'
   	END AS members
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members AS mem ON mem.customer_id = s.customer_id
ORDER BY s.customer_id, order_date)

SELECT *, 
	CASE
    WHEN members = 'N' then NULL
    ELSE
    	RANK () OVER(PARTITION BY customer_id, members
      	ORDER BY order_date) END AS ranking
FROM temp;
```
[Back to Top](## Week 1)


---

