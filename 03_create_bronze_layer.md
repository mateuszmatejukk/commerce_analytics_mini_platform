##  Bronze layer
### Table `bronze.customers`
```sql
create table if not exists bronze.customers(
customer_id INT primary key,
customer_name varchar(100) not null,
email varchar(150),
country varchar(2),
signup_date date not null,
acquisition_channel varchar(30)
);
```
### Table `bronze.products`
```sql
create table if not exists bronze.products(
product_id int primary key,
product_name varchar(100) not null,
category varchar(50) not null,
base_price numeric(10,2) not null
);
```
### Table `bronze.orders`
```sql
create table if not exists bronze.orders(
	order_id int primary key,
	customer_id int not null,
	order_timestamp timestamp not null,
	status varchar(30) not null,
	currency varchar(3) not null,
	sales_channel varchar(30) not null,
	constraint fk_orders_customers
		foreign key(customer_id)
		references bronze.customers(customer_id)
);
```
### Table `bronze.order_items`
```sql
create table if not exists bronze.order_items(
order_item_id int primary key,
order_id int not null,
product_id int not null,
quantity int not null,
unit_price numeric(10,2) not null,
discount_percent numeric(5,2) not null,
	constraint fk_order_items_orders
		foreign key (order_id)
		references bronze.orders(order_id),
	constraint fk_order_items_products
		foreign key (product_id)
		references bronze.products(product_id)
);
```
### Table `bronze.payments`
```sql
create table if not exists bronze.payments(
payment_id int primary key,
order_id int not null,
payment_status varchar(30) not null,
payment_method varchar(30) not null,
paid_at timestamp,
	constraint fk_payments_orders
	foreign key (order_id)
	references bronze.orders(order_id)
);
```
```sql
begin
```
```sql
insert into bronze.customers(
	customer_id,
	customer_name,
	email,
	country,
	signup_date,
	acquisition_channel)
select
	customer_id,
	customer_name,
	email,
	country,
	signup_date,
	acquisition_channel
from raw.customers;
```
```sql
insert into bronze.products(
	product_id,
	product_name,
	category,
	base_price)
select
	product_id,
	product_name,
	category,
	base_price
from raw.products;
```
```sql
insert into bronze.orders(
	order_id,
	customer_id,
	order_timestamp,
	status,
	currency,
	sales_channel)
select
	order_id,
	customer_id,
	order_timestamp,
	status,
	currency,
	sales_channel
from raw.orders;
```
```sql
insert into bronze.order_items(
	order_item_id,
	order_id,
	product_id,
	quantity,	
	unit_price,
	discount_percent)
select 
	order_item_id,
	order_id,
	product_id, 
	quantity, 
	unit_price,
	discount_percent
from raw.order_items
```
```sql
insert into bronze.payments(
	payment_id,
	order_id,
	payment_status,
	payment_method,
	paid_at)
select
	payment_id,
	order_id,
	payment_status,
	payment_method,
	paid_at
from raw.payments;
```
```sql
commit;
```
