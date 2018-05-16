# sql中updte inner join
  update count_table c inner join
　　  　(select count(*) cout,cust_id 
       from alarm_table
        where 
       to_days(alarm_date) = to_days(now())
       group by cust_id
       ) z
  on c.cust_id = z.cust_id
  set
  c.alarm_count=z.cout,c.date=current_date
  where
  c.cust_id = z.cust_id;
