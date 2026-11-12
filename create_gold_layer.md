```sql
create table if not exists gold.daily_sales_summary(
sales_date date not null,
country varchar(2) not null,
sales_channel varchar(30) not null,
orders_count int not null,
paid_orders_count int not null,
cancelled_orders_count int not null,
gross_revenue numeric(10,2) not null,
discount_value numeric(10,2) not null,
net_revenue numeric(10,2) not null,
	constraint pk_daily_sales_summary
	primary key(sales_date,country,sales_channel)
);
```
```sql
create table if not exists gold.customer_analytics(
customer_id int primary key,
customer_name varchar(100) not null,
country varchar(2) not null,
first_order_date date,
last_order_date date,
orders_count int not null,
total_net_revenue numeric(10,2) not null,
average_order_value numeric(10,2) not null,
customer_lifetime_rank int not null
);
```
```sql
create table if not exists gold.product_performance_monthly(
sales_month date not null,
category varchar(50) not null,
product_id int not null,
product_name varchar(100) not null,
units_sold int not null,
gross_revenue numeric(10,2) not null,
net_revenue numeric(10,2) not null,
category_rank_by_revenue int not null,
	constraint pk_product_performance_monthly
	primary key(sales_month, product_id)
);
```
```sql
insert into gold.daily_sales_summary(
sales_date,
country,
sales_channel,
orders_count,
paid_orders_count,
cancelled_orders_count,
gross_revenue,
discount_value,
net_revenue
)
select
oe.order_date as sales_date,
cc.country as country,
oe.sales_channel as sales_channel,
count(distinct oe.order_id) as orders_count,
count(case when oe.payment_status = 'paid' then oe.order_id end) as paid_orders_count,
count(case when oe.payment_status = 'failed' then oe.order_id end) as cancelled_orders_count,
round(sum(oie.gross_value),2) as gross_revenue,
round(sum(oie.discount_value),2) as discount_value,
round(sum(oie.net_value),2) as net_revenue
from silver.orders_enriched oe 
left join silver.customers_clean cc
on cc.customer_id = oe.customer_id 
left join silver.order_items_enriched oie
on oe.order_id = oie.order_id
group by oe.order_date, cc.country, oe.sales_channel
```
^ do dokonczenia
```sql
insert into gold.customer.analytics(
customer_id,
customer_name,
country,
first_order_date,
last_order_date,
orders_count,
total_net_revenue,
average_order_value,
customer_lifetime_rank
)
select
cc.customer_id,
cc.customer_name,
cc.country,
first_value(oe.order_date) over(
	partition by oe.customer_id
	order by oe.order_date, oe.order_id
	) as first_order_date,
last_value(oe.order_date) over(
	partition by oe.customer_id
	order by oe.order_date, oe.order_id
	ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
	) as last_order_date,
count(distinct oe.order_id) as orders_count,
```
^ do dokonczenia
```sql
insert into gold.product_performance_monthly(
sales_month,
category,
product_id,
product_name,
units_sold,
gross_revenue,
net_revenue,
category_rank_by_revenue
)
select
date_trunc('month',oe.order_date) as sales_month,
pc.category as category,
pc.product_id as product_id,
pc.product_name as product_name,
sum(oie.quantity) as units_sold,
sum(oie.gross_value) as gross_revenue,
sum(oie.net_value) as net_revenue,
dense_rank() over(
	partition by date_trunc('month',oe.order_date), pc.category
	order by sum(oie.net_value) desc
	) as category_rank_by_revenue
from silver.products_clean pc
join silver.order_items_enriched oie
on pc.product_id = oie.product_id
join silver.orders_enriched oe
on oie.order_id = oe.order_id 
where oe.payment_status = 'paid'
group by date_trunc('month',oe.order_date), pc.category,pc.product_id,pc.product_name
```
