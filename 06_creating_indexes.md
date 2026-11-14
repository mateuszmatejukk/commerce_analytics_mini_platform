## silver layer
```sql
create index if not exists idx_order_items_order_id
    on silver.order_items_enriched(order_id);

create index if not exists idx_order_items_product_id
    on silver.order_items_enriched(product_id);

create index if not exists idx_orders_customer_id
    on silver.orders_enriched(customer_id);
```
## gold layer
```sql
create index if not exists idx_daily_sales_date
    on gold.daily_sales_summary(sales_date);

create index if not exists idx_product_performance_month_category
    on gold.product_performance_monthly(sales_month, category);
```
