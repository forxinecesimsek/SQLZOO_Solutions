
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

### 6. List the Company name and the number of calls for those companies with more than 18 calls.

```sql
SELECT Company_name, Count(*) as cc FROM Issue
NATURAL JOIN Caller
NATURAL JOIN Customer
GROUP BY Company_name
HAVING Count(Company_name)>18
```

### 7. Find the callers who have never made a call. Show first name and last name.

```sql
SELECT first_name, last_name FROM Caller
LEFT JOIN Issue ON Caller.Caller_id=Issue.Caller_id
WHERE Issue.Caller_id IS NULL
```

### 8. For each customer show: Company name, contact name, number of calls where the number of calls is fewer than 5.

```sql
SELECT a.company_name, b.first_name, b.last_name, a.nc
FROM
(SELECT Customer.company_name, COUNT(*) AS nc, Customer.contact_id
FROM Customer JOIN Caller ON Customer.company_ref = Caller.company_ref
  JOIN Issue ON Caller.caller_id = Issue.caller_id
GROUP BY Customer.company_name, Customer.contact_id
HAVING COUNT(*) < 5) AS a
JOIN
(SELECT *
FROM Caller) AS b
ON (a.contact_id = b.caller_id);
```

### 9. For each shift show the number of staff assigned. Beware that some roles may be NULL and that the same person might have been assigned to multiple roles (The roles are 'Manager', 'Operator', 'Engineer1', 'Engineer2').

```sql
SELECT Shift_date,Shift_type, COUNT(*) as cw FROM (
  SELECT Shift_date,Shift_type,Manager,Operator,Engineer1,Engineer2,staff_code 
  FROM Shift 
  INNER JOIN Staff ON staff_code = Manager
UNION
  SELECT Shift_date,Shift_type,Manager,Operator,Engineer1,Engineer2,staff_code 
  FROM Shift 
  INNER JOIN Staff ON staff_code = Operator
UNION
  SELECT Shift_date,Shift_type,Manager,Operator,Engineer1,Engineer2,staff_code 
  FROM Shift 
  INNER JOIN Staff ON staff_code = Engineer1
UNION
  SELECT Shift_date,Shift_type,Manager,Operator,Engineer1,Engineer2,staff_code 
  FROM Shift INNER JOIN Staff ON staff_code = Engineer2) s

GROUP BY Shift_date,Shift_type
```

### 10. Caller 'Harry' claims that the operator who took his most recent call was abusive and insulting. Find out who took the call (full name) and when.
```sql
WITH a AS (SELECT Taken_by, caller_id, call_date FROM Caller
NATURAL JOIN Issue
WHERE first_name='Harry')

SELECT first_name, last_name, call_date FROM a
INNER JOIN Staff ON Taken_by=Staff_Code
ORDER BY call_date DESC
LIMIT 1
```
