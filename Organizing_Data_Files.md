Our target variables are spread out over multiple data files, so we first have to get everything in the same place.

Let's rename the "return" table so we don't have to keep using backticks

```sql
rename table `return` to return1;

select ID, `Session number` from return1
limit 20;
```
Output:
```
+------------+---------------+
| ID         | Session number|
+------------+---------------+
|  5	     |       1       |
|  5	     |       2       | 
|  3	     |       2       |
|  6	     |       1       |
|  4	     |       2       |
|  5         |       3       |
|  6	     |       2       |
|  5	     |       4       |
+------------+---------------+
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

Output:

```
+--------------+-----------------+
| ID           | Session number  |
+--------------+-----------------+
|  1	       |       1         |
|  1	       |       2         | 
|  1	       |       3         |
|  1	       |       4         |
|  2	       |       1         |
|  2           |       2         |
|  2           |       3         |
|  2           |       4         |
|  5           |       1         |
|  5           |       2         |
+--------------+-----------------+
```

Now, let's filter the other tables.

```sql
describe intake;

describe tampon_data;
```

Make the ID column the correct data type

```sql
alter table tampon_data
modify column `Participant ID` int;
```

Filter both tables so they only contain participants who 
completed all four sessions (using the list of unique 
ID numbers from our complete_returns table).

```sql
create table complete_tamp as
select *
from tampon_data
where `Participant ID` in(1, 2, 5, 6, 15, 16, 17, 19, 20, 21, 25, 27, 30, 34, 35, 37, 39, 44, 47, 50, 51, 57, 58, 63, 64, 65, 70, 71, 72, 74, 76, 77, 82, 84, 85, 87, 88, 91, 92, 95, 96, 97, 98, 103);

create table complete_intake as
select *
from intake
where PID in(1, 2, 5, 6, 15, 16, 17, 19, 20, 21, 25, 27, 30, 34, 35, 37, 39, 44, 47, 50, 51, 57, 58, 63, 64, 65, 70, 71, 72, 74, 76, 77, 82, 84, 85, 87, 88, 91, 92, 95, 96, 97, 98, 103);
```

Now let's join the complete_tamp and complete_returns tables

Hang on, we can't have duplicate column names when we do a join.
Let's fix that first.

```sql
alter table complete_tamp
rename column `Session Number` to session;
```

Okay, now we can join them

```sql
create table complete_returns_tamp as
select *
from complete_returns
inner join complete_tamp
on complete_returns.ID = complete_tamp.`Participant ID`
and complete_returns.`Session number` = complete_tamp.session;
```

And let's put the ID and session number columns from each table next to each other
in the new table, just so we can double-check our work.

```sql
alter table complete_returns_tamp
modify `Participant ID` int
after ID;

alter table complete_returns_tamp
modify session text
after `Session number`;
```
Great! Everything looks good so far.
Now we should add the relevant information from the
fertility_bmi table, but first, we'll need to reorganize it.
Currently, the fertility_bmi table has separate columns for 
fertility status at each condition. We want to reorganize the data
so that we just have two columns: fertility status (high, low, no)
and condition (A, B, C, D).

```sql
create table fertility as
select PID, 'A' as cond, A_Fert as fertility
from fertility_bmi
union
select PID, 'B' as cond, B_Fert as fertility
from fertility_bmi
union
select PID, 'C' as cond, C_Fert as fertility
from fertility_bmi
union
select PID, 'D' as cond, D_Fert as fertility
from fertility_bmi;
```

Now we should be able to do an inner join using the 
experimental conditions (A, B, C, D) from both tables.

```sql
create table returns_tamp_fert as
select *
from complete_returns_tamp
inner join fertility
on complete_returns_tamp.ID = fertility.PID
and complete_returns_tamp.`Audio Condition` = fertility.cond;
```



















