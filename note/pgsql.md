## 查询当月当天之前的日期
```sql
select to_char(tdate.day, 'yyyy-mm-dd') as day 
from (
	select generate_series(cast(to_char(current_date, 'yyyy-mm') || '-01' as date),
		cast(cast(to_char(current_date, 'yyyy-mm') || '-01' as timestamp) + '1 d' + '-1 d' as date), '1 d'
		) as day
	) as tdate
order by day;
```
## 查询当月的日期
```sql
select to_char(tdate.day, 'yyyy-mm-dd') as day 
from (
	select generate_series(
		cast(to_char(current_date, 'yyyy-mm') || '-01' as date),
		cast(cast(to_char(current_date, 'yyyy-mm') || '-01' as timestamp) + '1 MONTH' + '-1 d' as date), '1 d'
		) as day
	) as tdate
order by day;
```

## 查询指定区间的所有日期
```sql
select to_char(ddata.day, 'yyyy-mm-dd') as orderDate
from (
	select generate_series(
		cast(to_char(to_date('2018-07-11', 'YYYY-MM'), 'yyyy-mm') || '-01' as date),
    	cast(cast(to_char(to_date('2018-08-12', 'YYYY-MM'), 'yyyy-mm') || '-01' as timestamp) + '-1 d' as date), '1 d'
    ) as day
    ) as ddata 
order by orderDate;
```

## 查询会话
```sql
select T.PID, T.STATE, T.QUERY, T.WAIT_EVENT_TYPE, T.WAIT_EVENT,T.QUERY_START 
from PG_STAT_ACTIVITY T
```

## kill 进程 
```sql
select PG_CANCEL_BACKEND(pid);  # 可杀死 Lock

select PG_TERMINATE_BACKEND(pid); # 可关闭 空闲的会话
```

## 为空时赋默认值
```sql
COALESCE(field, 0)
```