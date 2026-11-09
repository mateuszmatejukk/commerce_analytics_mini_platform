### First of all, we need to create schemas following medallion architecture pattern.
```sql
create schema if not exists raw;
```
- `raw` - raw data
```sql
create schema if not exists bronze;
```
- `bronze` - technical copy of raw data
```sql
create schema if not exists silver;
```
- `silver` - data cleaned, standardized and enriched
```sql
create schema if not exists gold;
```
- `gold` - final tables for analysis
