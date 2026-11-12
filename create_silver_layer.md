```sql
create table if not exists silver.customers_clean(
	customer_id int primary key,
	customer_name varchar(100) not null,
	email varchar(150),
	country varchar(2) not null,
	signup_date date not null,
	acquisition_channel varchar(30)
);
```
```sql
create table if not exists silver.products_clean(
	product_id int primary key,
	product_name varchar(100) not null,
	category varchar(50) not null,
	base_price numeric(10,2) not null
);
```
```sql
create table if not exists silver.orders_enriched(
	order_id int primary key,
	customer_id int not null,
	order_date date not null,
	order_timestamp timestamp not null,
	order_month date not null,
	order_status varchar(30) not null,
	currency varchar(3) not null,
	sales_channel varchar(30) not null,
	payment_status varchar(30),
	payment_method varchar(30),
	is_paid boolean not null,
	first_order_date date not null,
	constraint fk_orders_customers
	foreign key(customer_id)
	references silver.customers_clean(customer_id)
	);
```
```sql
create table if not exists silver.order_items_enriched(
	order_item_id int primary key,
	order_id int not null,
	product_id int not null,
	product_name varchar(100) not null,
	category varchar(50) not null,
	quantity int not null,
	unit_price numeric(10,2) not null,
	discount_percent numeric(5,2) not null,
	gross_value numeric(10,2) not null,
	discount_value numeric(10,2) not null,
	net_value numeric(10,2) not null,
		constraint fk_order_items_orders
		foreign key (order_id)
		references silver.orders_enriched(order_id),
	constraint fk_order_items_products
		foreign key (product_id)
		references silver.products_clean(product_id)
	);
```
```sql
begin;
```
```sql
insert into silver.customers_clean
(customer_id,
customer_name,
email,
country,
signup_date,
acquisition_channel)
select
	customer_id,
	initcap(trim(customer_name)) as customer_name,
	trim(lower(email)) as email,
	upper(country) as country,
	signup_date,
	lower(acquisition_channel) as acquisition_channel
from bronze.customers
```
```sql
begin;
```
```sql
insert into silver.products_clean(
product_id,
product_name,
category,
base_price)
select
product_id,
trim(product_name) as product_name,
lower(trim(category)) as category,
base_price
from bronze.products;
```
```sql
commit;
```
```sql
begin
```
```sql
insert into silver.orders_enriched(
order_id,
customer_id,
order_date,
order_timestamp,
order_month,
order_status,
currency,
sales_channel,
payment_status,
payment_method,
is_paid,
first_order_date)
select
o.order_id,
o.customer_id,
date_trunc('day',o.order_timestamp) as order_date,
o.order_timestamp,
date_trunc('month',o.order_timestamp) as order_month,
lower(o.status) as order_status,
upper(o.currency) as currency,
lower(o.sales_channel) as sales_channel,
trim(lower(p.payment_status)) as payment_status,
trim(lower(p.payment_method)) as payment_method,
case
	when p.payment_status = 'paid' then true 
	else false
end as is_paid,
first_value(o.order_timestamp) over(
	partition by o.customer_id
	order by o.order_timestamp, o.order_id
) as first_order_date
from bronze.orders o
left join bronze.payments p
on o.order_id = p.order_id
```
```sql
commit
```
```sql
begin
```
```sql
insert into silver.order_items_enriched(
order_item_id,
order_id,
product_id,
product_name,
category,
quantity,
unit_price,
discount_percent,
gross_value,
discount_value,
net_value
)
select
oi.order_item_id,
oi.order_id,
p.product_id,
trim(p.product_name),
lower(p.category),
oi.quantity,
oi.unit_price,
oi.discount_percent,
(oi.quantity * oi.unit_price) as gross_value,
(oi.quantity * oi.unit_price * oi.discount_percent / 100) as discount_value,
oi.quantity * oi.unit_price * (1- oi.discount_percent /100 ) as net_value
from bronze.order_items oi
left join bronze.products p
on oi.product_id = p.product_id;
```
```sql
commit
```
