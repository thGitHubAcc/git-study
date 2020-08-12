#  查询表
```sql
select table_name,TABLE_COMMENT from information_schema.tables where table_schema = 'project'
```
# 查询表字段
```sql
SELECT
    TABLE_NAME,
    COLUMN_NAME,
    ORDINAL_POSITION,
    COLUMN_DEFAULT,
    DATA_TYPE AS 'data_type',
    COLUMN_KEY 'KEY',
    COLUMN_COMMENT AS 'des'
FROM
    information_schema.`COLUMNS`
WHERE
    TABLE_SCHEMA = 'project'
ORDER BY
    TABLE_NAME,
    ORDINAL_POSITION;
```
    
