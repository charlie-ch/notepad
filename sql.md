# sql中updte inner join
  update count_table c inner join<br>
　　  　(select count(*) cout,cust_id <br>
       from alarm_table<br>
        where <br>
       to_days(alarm_date) = to_days(now()) <br>
       group by cust_id<br>
       ) z<br>
  on c.cust_id = z.cust_id<br>
  set<br>
  c.alarm_count=z.cout,c.date=current_date<br>
  where<br>
  c.cust_id = z.cust_id;<br>
