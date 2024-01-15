Our target variables are spread out over multiple data files, so we first have to get everything in the same place.

Let's rename the "return" table so we don't have to keep using backticks

```sql
rename table `return` to return1;

select ID, `Session number` from return1
limit 20;
```
Output:
```
+----------+-----------------+
| ID         | Session number|
+----------+-----------------+
|  5	       |       1       |
|  5	       |       2       | 
|  3	       |       2       |
|  6	       |       1       |
|  4	       |       2       |
|  5	       |       3       |
|  6	       |       2       |
|  5	       |       4       |
+----------+----------------+
```

ID 5, session 3 was incorrectly marked as session 4
We can tell this was supposed to be session 3 by looking at the date
(This session is dated before the other row with ID 5, session 4)
Let's fix this before moving forward.

```sql
update return1
set `Session number` = 3
where `Recorded Date` = "3/28/2022 15:19";
```

Now, let's make a new table of return sessions that contains ONLY 
participants who completed all 4 sessions. 

```sql
create table complete_returns as
select *
from return1
where ID in(
	select ID
    from return1
	where `Session number` in (1, 2, 3, 4)
	group by ID
	having count(distinct `Session number`) = 4
);

select ID, `Session number` from complete_returns
order by ID, `Session number`
limit 10;
```
Output:

```
+--------------+-----------------+
| ID           | Session number  |
+--------------+-----------------+
|  1	       |       1         |
|  1	       |       2         | 
|  1	       |       3         |
|  1	       |       4         |
|  103	       |       1         |
|  103         |       2         |
|  103         |       3         |
|  103         |       4         |
|  15          |       1         |
|  15          |       2         |
+--------------+-----------------+
```
Great! But something is wrong with our variable types
because ID is not being displayed in ascending order.
Let's fix that.

```sql
describe complete_returns;
```

Nearly all of our columns are type "text" but
they need to be "int"

```sql
alter table complete_returns
modify column FoC8 int;
```

And it looks like we have some blank cells in a few columns,
so we'll need to put numeric values in there before we 
can change those columns to type "int"

```sql
update complete_returns
set LoV7 = 0
where LoV7 = "";

update complete_returns
set FoC8 = 0
where FoC8 = "";
```

Okay, let's take a look at the table again

```sql
select ID, `Session number` from complete_returns
order by ID, `Session number`
limit 10;
```

