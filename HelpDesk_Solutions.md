
# Easy Questions

### 1. There are three issues that include the words "index" and "Oracle". Find the call_date for each of them.

```sql
SELECT  call_date, call_ref FROM Issue```
WHERE Detail LIKE '%Oracle%' AND Detail LIKE '%index%'
```

### 2.Samantha Hall made three calls on 2017-08-14. Show the date and time for each.

```sql
SELECT call_date, first_name, last_name FROM Issue
NATURAL JOIN Caller
WHERE First_name = 'Samantha' AND Last_name = 'Hall' AND Date(call_date) = '2017-08-14'
```

### 3. There are 500 calls in the system (roughly). Write a query that shows the number that have each status.


```sql
SELECT Status status, COUNT(Detail) Volume FROM Issue
GROUP BY Status
```


### 4. Calls are not normally assigned to a manager but it does happen. How many calls have been assigned to staff who are at Manager Level?


```sql
SELECT COUNT(Assigned_to) mlcc FROM Issue
WHERE Assigned_to='LB1' OR Assigned_to='AE1'
GROUP BY Assigned_to
```

### 5. Show the manager for each shift. Your output should include the shift date and type; also the first and last name of the manager.

```sql
SELECT Shift_date, Shift_type, first_name, last_name FROM Shift
NATURAL JOIN Shift_type, Staff
WHERE Manager=Staff_code
ORDER BY Shift_date ASC
```

# Medium Questions
