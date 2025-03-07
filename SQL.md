# Find customer id having more than 1 orders and item name as well
```sql
Orders
order_id	item	      amount	customer_id
1	        Keyboard	  400	    4
2	        Mouse	      300	    4
3	        Monitor	    12000	  3
4	        Keyboard	  400	    1
5	        Mousepad	  250	    2

//oracle
select customer_id, listtag(item, ', ') within group (order by item) as items, count(order_id) from orders group by customer_id having count(order_id) > 1;
//SQLlite
select customer_id, group_concat(item, ', ') as items, count(order_id) from orders group by customer_id having count(order_id) > 1;
```
