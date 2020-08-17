*Tables:* *[link to tables of HelpDesk](https://sqlzoo.net/wiki/Helpdesk_Hard_Questions)*

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

### 11. Show the manager and number of calls received for each hour of the day on 2017-08-12.

```sql
WITH date_table AS 
       (SELECT DATE_FORMAT(call_date, '%Y-%m-%d %H') date_hour, 
        DATE_FORMAT(call_date, '%Y-%m-%d') date, DATE_FORMAT(call_date, '%H') 
        hour 
        FROM Issue
        WHERE DATE_FORMAT(call_date, '%Y-%m-%d')='2017-08-12'
        )

SELECT Manager, date_hour, COUNT(*) cc 
FROM date_table
INNER JOIN Shift 
       ON date_table.date=Shift.Shift_date
WHERE Shift.Shift_type = 'early' 
      AND date_table.hour <= 13 
      OR Shift.Shift_type = 'late' 
      AND date_table.hour > 13
      GROUP BY Manager,date_hour

ORDER BY date_hour
```
### 13. Annoying customers. Customers who call in the last five minutes of a shift are annoying. Find the most active customer who has never been annoying.


```sql
WITH not_annoyings AS 
(SELECT
	Caller.Company_ref,
	COUNT(*) count
FROM Caller
    JOIN Issue ON (Caller.Caller_id = Issue.Caller_id)
WHERE Caller.Caller_id NOT IN (
    SELECT DISTINCT i.caller_id
	FROM Issue i
		WHERE HOUR(i.call_date) in (13, 19)
			AND MINUTE(i.call_date) >= 55)
GROUP BY Caller.Company_ref)

SELECT
    Customer.Company_Name, 
    not_annoyings.count
FROM not_annoyings
    JOIN Customer ON Customer.Company_ref = not_annoyings.Company_ref   
ORDER BY not_annoyings.count DESC LIMIT 1;
```

### 14. Maximal usage. If every caller registered with a customer makes a call in one day then that customer has "maximal usage" of the service. List the maximal customers for 2017-08-13.

```sql
SELECT
	a.Company_name,
	a.caller_count,
	b.issue_count
FROM
	(
		SELECT
			Customer.Company_name,
			COUNT(Caller.Company_ref) AS caller_count
		FROM
			Customer
			JOIN
				Caller
				ON (Customer.Company_ref = Caller.Company_ref)
		GROUP BY
			Customer.Company_name
	)
	AS a
	JOIN
		(
			SELECT
				Customer.Company_name,
				COUNT(DISTINCT Issue.Caller_id) AS issue_count
			FROM
				Customer
				JOIN
					Caller
					ON (Customer.Company_ref = Caller.Company_ref)
				JOIN
					Issue
					ON (Caller.Caller_id = Issue.Caller_id)
			WHERE
				YEAR(Issue.call_date) = '2017'
				AND MONTH(Issue.call_date) = '08'
				AND DAY(Issue.call_date) = '13'
			GROUP BY
				Customer.Company_name
		)
		AS b
		ON a.Company_name = b.Company_name
WHERE
	a.caller_count = b.issue_count;
```

### 15. Consecutive calls occur when an operator deals with two callers within 10 minutes. Find the longest sequence of consecutive calls – give the name of the operator and the first and last call date in the sequence.

```sql
SELECT
	a.taken_by,
	a.first_call,
	a.last_call,
	a.call_count AS calls
FROM
	(
		SELECT
			taken_by,
			call_date AS last_call,
			@row_number1:= CASE
				WHEN
					TIMESTAMPDIFF(MINUTE, @call_date, call_date) <= 10
				THEN
					@row_number1 + 1
				ELSE
					1
			END AS call_count,
			@first_call_date:= CASE
				WHEN
					@row_number1 = 1
				THEN
					call_date
				ELSE
					@first_call_date
			END AS first_call,
			@call_date:= Issue.call_date AS call_date
		FROM
			Issue,
			(
				SELECT
					@row_number1 := 0,
					@call_date := 0,
					@first_call_date := 0
			)
			AS row_number_init
		ORDER BY
			taken_by,
			call_date
	)
	AS a
ORDER BY
	a.call_count DESC LIMIT 1;
```

