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
	payment_stauts varchar(30) not null,
	payment_method varchar(30),
	is_paid boolean not null,
	first_order_date date not null,
	constraint fk_orders_customers
	foreign key(customer_id)
	references silver.customers_clean(customer_id)
	);
```
^ to do rozkminienia
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
	discount_value numeric(5,2) not null,
	net_value numeric(10,2) not null
	);
```
