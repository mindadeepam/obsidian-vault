
SQLAlchemy provides us very convenient utilities to interact with any sql-based database.

They offer 2 main things:
1. Core
2. ORM

Core can be used to interact with any db without setting up any schema classes and such, but if we want to make full use of 


- https://github.com/sqlalchemy/sqlalchemy/discussions/8272
- example of simple ops in core vs ORM: https://chatgpt.com/share/d0315806-2109-4e9c-a523-eb0a7835f141

> [!see also]
> [[library/notes/Transactions]]



## Albemic automate migrations

#### boiler plate code
  
```bash
alembic init migrations
alembic revision --autogenerate
alembic upgrade head 


alembic downgrade  # to revert
```


```python
target_metadata = MetaData()

def include_name(name, type_, parent_names):
	if type_ == "schema":
		return name in ["data_pipelines_raw"]
	else:
		return True

# return True
# inside run_migrations_online..
context.configure(
	connection=connection,
	target_metadata=target_metadata,    
	include_schemas=True,
	include_name = include_name,
	version_table_schema='data_pipelines_raw'
)
```

- `target_metadata` ko kuch bhi dedo (initialize new metadata)
- `include_schemas`=True will include all schemas, unless restricted by include_name
- `version_table_schema` : schema to place the alembic version table in





### make created at and updated at columns

1. create a function to trigger timestamp: do it at a schema level.
```sql
CREATE OR REPLACE FUNCTION data_pipelines_raw.trigger_set_timestamp()
RETURNS trigger
LANGUAGE plpgsql
AS $function$
BEGIN
NEW.updated_at = NOW();
RETURN NEW;
END;
$function$
;
```

2. create a table like so.
```sql
CREATE TABLE data_pipelines_raw.imports (
country text NOT NULL,
import_qty float8 NULL,
import_cost float8 NULL,
import_date timestamp NOT NULL,
commodity text NOT NULL,
created_at timestamp DEFAULT (now() AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Kolkata') NOT NULL,
updated_at timestamp DEFAULT  (now() AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Kolkata') NOT NULL,
relevant_commodity text NOT NULL,
CONSTRAINT imports_pk PRIMARY KEY (import_date, commodity, country, relevant_commodity)
);

-- Table Triggers
create trigger set_timestamp before
update
on
data_pipelines_raw.imports for each row execute function data_pipelines_raw.trigger_set_timestamp();
```


```sql
ALTER TABLE your_table_name 
ALTER COLUMN created_at SET DEFAULT (now() AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Kolkata'),
ALTER COLUMN modified_at SET DEFAULT (now() AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Kolkata');

```

you are all set


```

```