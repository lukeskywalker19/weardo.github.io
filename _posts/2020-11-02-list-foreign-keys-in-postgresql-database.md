---
layout: post
title: List foreign keys in PostgreSQL Database
date: 2020-11-02 13:03 +0530
---

Query below returns foreign key constraints defined in a PostgreSQL database

### Query

```sql
select kcu.table_schema || '.' ||kcu.table_name as foreign_table,
       '>-' as rel,
       rel_tco.table_schema || '.' || rel_tco.table_name as primary_table,
       string_agg(kcu.column_name, ', ') as fk_columns,
       kcu.constraint_name
from information_schema.table_constraints tco
join information_schema.key_column_usage kcu
          on tco.constraint_schema = kcu.constraint_schema
          and tco.constraint_name = kcu.constraint_name
join information_schema.referential_constraints rco
          on tco.constraint_schema = rco.constraint_schema
          and tco.constraint_name = rco.constraint_name
join information_schema.table_constraints rel_tco
          on rco.unique_constraint_schema = rel_tco.constraint_schema
          and rco.unique_constraint_name = rel_tco.constraint_name
where tco.constraint_type = 'FOREIGN KEY'
group by kcu.table_schema,
         kcu.table_name,
         rel_tco.table_name,
         rel_tco.table_schema,
         kcu.constraint_name
order by kcu.table_schema,
         kcu.table_name;
```

### Columns
- **foreign_table** - foreign table schema and name
- **rel** - relationship symbol implicating direction
- **primary_table** - primary (rerefenced) table schema and name
- **fk_columns** - list of FK colum names, separated with ","
- **constraint_name** - foreign key constraint name

### Rows
- **One row** represents one foreign key. If foreign key consists of multiple columns (composite key) it is still represented as one row.
- **Scope of rows:** all foregin keys in a database
- **Ordered by** foreign table schema name and table name