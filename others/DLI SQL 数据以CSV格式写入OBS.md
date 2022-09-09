```sql
INSERT OVERWRITE DIRECTORY 'obs://yishou-data/export/dashboard'
  USING csv

select * from  yishou_data.dws_finebi_dw_table_access_count_dt 
where dt = 20220619
```

