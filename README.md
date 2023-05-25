# SQL-casestudy---Tiny-Shop-Sales-Analysis
#data in motion 

```
-- Case Study Questions

--1) Which product has the highest price? Only return a single row.

select product_name
from products
order by price desc
limit 1;


--2) Which customer has made the most orders?
select concat(first_name, ' ',last_name) as full_name, count(o.order_id)
from customers c
join orders o
on o.customer_id= c.customer_id
group by 1
having count(o.order_id) > 1;

--3) What’s the total revenue per product?
select p.product_id, sum(quantity) as prod_cnt, price , sum(p.price* oi.quantity) as Total_rev
from order_items oi
join products p
on p.product_id = oi.product_id
group by p.product_id
order by 1;


--4) Find the day with the highest revenue.
select distinct o.order_date, sum(price* quantity) as Total_rev
from order_items oi
join products p
on p.product_id = oi.product_id
join orders o
on o.order_id = oi.order_id
group by 1
order by 2 desc
limit 1;


--5) Find the first order (by date) for each customer.
with cte as 
(select c.customer_id,  concat(c.first_name, ' ', c.last_name) as full_name, o.order_date,
dense_rank() over(partition by c.customer_id order by o.order_date asc) as rnk
from customers c
join orders o 
on o.customer_id = c.customer_id)

select full_name, order_date
from cte 
where rnk = 1;


--6) Find the top 3 customers who have ordered the most distinct products
select c.customer_id, concat(c.first_name ,' ', c.last_name) as full_name , count(distinct oi.product_id) as distinct_product
from customers c
join orders o
on o.customer_id = c.customer_id
join order_items oi
on oi.order_id = o.order_id
group by 1, 2
order by 3 desc
limit 3;

--7) Which product has been bought the least in terms of quantity?
select sum(quantity) as product_quantity, product_id
from order_items
group by 2
order by 1;

--8) What is the median order total?
with cte as 
(select oi.order_id, sum(oi.quantity * p.price) AS total_order
 from order_items oi
 join products p
 on p.product_id = oi.product_id
 group by 1
)
select percentile_cont(0.5) within group (order by total_order) as median_order_total
from cte;

--9) For each order, determine if it was ‘Expensive’ (total over 300), ‘Affordable’ (total over 100), or ‘Cheap’.
with cte as 
(select oi.order_id, sum(price* quantity) as Total_rev
from order_items oi
join products p
on p.product_id = oi.product_id
join orders o
on o.order_id = oi.order_id
group by 1)

select order_id, total_rev,
(case when total_rev > 300 then 'expensive'
     when total_rev > 100 then 'affordable'
     else 'cheap'
     end) as price_range
from cte
order by 1 ;


--10) Find customers who have ordered the product with the highest price.
 with cte as 
(select c.customer_id,concat(c.first_name,' ',c.last_name) as full_name,
p.price,
dense_rank () over(order by price desc) as rnk
 from customers c
join orders o 
on c.customer_id = o.customer_id
join order_items oi
on oi.order_id = o.order_id
join products p 
on p.product_id = oi.product_id
order by 3 desc)

select full_name 
from cte 
where rnk = 1 ;
```
