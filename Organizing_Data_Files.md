Our target variables are spread out over multiple data files, so we first have to get everything in the same place.

Let's rename the "return" table so we don't have to keep using backticks

```sql
rename table `return` to return1;

select ID, `Session number` from return1
limit 20;
```

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
