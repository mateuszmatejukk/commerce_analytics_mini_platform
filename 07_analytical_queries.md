## daily net sales per country
```sql
select 
sales_date,
country,
sum(net_revenue) as total_revenue
from gold.daily_sales_summary
group by sales_date, country
order by sales_date
```

## top 5 customers by total revenue
```sql
select
customer_id,
customer_name,
total_net_revenue
from gold.customer_analytics
order by total_net_revenue desc
limit 5;
```

## top 3 products per every category and month
```sql
with numbered_products as(
	select
	product_id,
	product_name,
	category,
	sales_month,
	net_revenue,
	row_number() over(
	partition by category, sales_month
	order by net_revenue desc
	) as rn
	from gold.product_performance_monthly
)
select
product_id,
product_name,
category,
sales_month,
net_revenue
from numbered_products 
where rn <= 3 
order by category, sales_month, rn;
```
## monthly net revenue
```sql
select
date_trunc('month',sales_date) as sales_month,
sum(net_revenue) as total_revenue
from gold.daily_sales_summary 
group by date_trunc('month',sales_date)
order by date_trunc('month',sales_date);
```
## discount shares of sales
```sql
select
sum(gross_revenue) as total_gross_revenue,
sum(discount_value) as total_discount_value,
round(sum(discount_value)  / sum(gross_revenue ) *100,2) as discount_share
from gold.daily_sales_summary
```
## number of orders by payment status
```sql
select
count(order_id) as orders_count,
payment_status
from silver.orders_enriched 
group by payment_status
order by orders_count desc;
```
