*Tables:* *[link to tables of Guest House](https://sqlzoo.net/wiki/Guest_House)*

# Easy Questions

### 1. Guest 1183. Give the booking_date and the number of nights for guest 1183.

```sql
SELECT DATE_FORMAT(booking_date, '%Y-%m-%d') booking_date, nights
FROM booking
WHERE guest_id = 1183
```

### 2. When do they get here? List the arrival time and the first and last names for all guests due to arrive on 2016-11-05, order the output by time of arrival.

```sql
SELECT arrival_time, first_name, last_name FROM booking
INNER JOIN guest ON booking.guest_id=guest.id
WHERE booking_date='2016-11-05'
ORDER BY arrival_time ASC
```

### 3. Look up daily rates. Give the daily rate that should be paid for bookings with ids 5152, 5165, 5154 and 5295. Include booking id, room type, number of occupants and the amount.

```sql
SELECT booking.booking_id, room_type_requested, occupants, amount FROM booking 
INNER JOIN rate ON booking.room_type_requested=rate.room_type AND booking.occupants=rate.occupancy
WHERE booking.booking_id IN (5152,5165,5154,5295)
```

### 4. Who’s in 101? Find who is staying in room 101 on 2016-12-03, include first name, last name and address.

```sql
SELECT first_name, last_name, address FROM guest
INNER JOIN booking ON guest.id=booking.guest_id
WHERE room_no=101 AND booking_date='2016-12-03'
```

### 5. How many bookings, how many nights? For guests 1185 and 1270 show the number of bookings made and the total number of nights. Your output should include the guest id and the total number of bookings and the total number of nights.

```sql
SELECT guest_id, COUNT(nights), SUM(nights) FROM booking
WHERE booking.guest_id IN (1185,1270)
GROUP BY guest_id
```

# Medium Questions

### 6. Ruth Cadbury. Show the total amount payable by guest Ruth Cadbury for her room bookings. You should JOIN to the rate table using room_type_requested and occupants.

```sql
WITH a AS 

(SELECT guest.first_name, guest.last_name, b.nights, rate.amount
 
 FROM booking b
   INNER JOIN rate ON b.room_type_requested=rate.room_type 
   AND b.occupants=rate.occupancy
     INNER JOIN guest ON b.guest_id=guest.id)

SELECT cast(truncate(SUM(nights*amount) + 0.00001, 2) as varchar(255)) 
FROM a
WHERE a.first_name='Ruth' AND a.last_name='Cadbury'
```

### 7. Including Extras. Calculate the total bill for booking 5346 including extras.

```sql
WITH join_table AS (SELECT * FROM booking b
   INNER JOIN rate ON b.room_type_requested=rate.room_type 
   AND b.occupants=rate.occupancy
WHERE b.booking_id='5346')

SELECT SUM(extra.amount) + join_table.amount total_amount FROM join_table
INNER JOIN extra ON join_table.booking_id=extra.booking_id
```

### 8. Edinburgh Residents. For every guest who has the word “Edinburgh” in their address show the total number of nights booked. Be sure to include 0 for those guests who have never had a booking. Show last name, first name, address and number of nights. Order by last name then first name.

```sql
WITH not_sum_table AS 
   (SELECT last_name, first_name, address, nights 
    FROM guest g
    LEFT JOIN booking ON g.id=booking.guest_id
    WHERE address LIKE 'Edinburgh%')

SELECT last_name, first_name, address, IFNULL(SUM(nights),0) 
FROM not_sum_table
GROUP BY last_name, first_name
ORDER BY last_name, first_name
```


### 9. How busy are we? For each day of the week beginning 2016-11-25 show the number of bookings starting that day. Be sure to show all the days of the week in the correct order.

```sql
SELECT DATE_FORMAT(b.booking_date, '%Y-%m-%d') i, COUNT(*) arrivals FROM booking b
WHERE b.booking_date BETWEEN '2016-11-25' AND '2016-12-01'
GROUP BY 1
ORDER BY 1 ASC
```

### 10. How many guests? Show the number of guests in the hotel on the night of 2016-11-21. Include all occupants who checked in that day but not those who checked out.

```sql
SELECT SUM(occupants)
FROM booking b
WHERE b.booking_date + INTERVAL nights DAY > '2016-11-21' AND b.booking_date <= '2016-11-21'
```

# Hard Questions

### 11. Coincidence. Have two guests with the same surname ever stayed in the hotel on the evening? Show the last name and both first names. Do not include duplicates.

```sql
SELECT last.last_name, substring_index(group_concat(last.first1 ORDER BY last.last_name),',',1),
substring_index(group_concat(last.first1 ORDER BY last.last_name),',',-1)
FROM (
 SELECT DISTINCT a.last_name,a.first_name first1,b.first_name first2
 FROM (SELECT * FROM booking bo JOIN guest g ON bo.guest_id = g.id) a, 
      (SELECT * FROM booking bo JOIN guest g ON bo.guest_id = g.id) b
 WHERE a.last_name = b.last_name AND a.first_name != b.first_name
 AND (
   (a.booking_date + INTERVAL a.nights DAY > b.booking_date) AND (a.booking_date<=b.booking_date)
 OR (b.booking_date + INTERVAL b.nights DAY > a.booking_date) AND (b.booking_date<=a.booking_date))
  ) last
GROUP BY last_name
```
### 12. Check out per floor. The first digit of the room number indicates the floor – e.g. room 201 is on the 2nd floor. For each day of the week beginning 2016-11-14 show how many rooms are being vacated that day by floor number. Show all days in the correct order.

```sql
SELECT DATE_FORMAT(booking_date, '%Y-%m-%d') i,
       COUNT(CASE WHEN room_no LIKE '1%' then 1 end) as 1st,
       COUNT(CASE WHEN room_no LIKE '2%' then 1 end) as 2nd,
       COUNT(CASE WHEN room_no LIKE '3%' then 1 end) as 3rd
FROM booking
GROUP BY booking_date
HAVING booking_date BETWEEN '2016-11-14' AND '2016-11-20'
ORDER BY 1
```
### 13. Free rooms? List the rooms that are free on the day 25th Nov 2016.

```sql
SELECT r.id
FROM room r
LEFT JOIN
(
  SELECT DISTINCT room_no
  FROM booking
  WHERE booking_date <= '2016-11-25' and DATE_ADD(booking_date,INTERVAL nights DAY) > '2016-11-25'
) a
ON r.id = a.room_no

WHERE a.room_no is null
```

### 14. Single room for three nights required. A customer wants a single room for three consecutive nights. Find the first available date in December 2016.

```sql
SELECT id, date_format(next_date, '%Y-%m-%d'),
CASE WHEN avail_days IS NULL THEN EXTRACT(DAY from next_date) - EXTRACT(DAY from bkl) 
  ELSE avail_days end as avail
FROM
(
SELECT l.id, l.bkl, 
DATE_ADD(l.bkl, INTERVAL l.nl DAY) AS next_date,
MIN(EXTRACT(DAY from r.bkr) - EXTRACT(DAY from l.bkl) - l.nl) AS avail_days
FROM
(
SELECT r.id as id
, b.booking_date as bkl
, b.nights as nl
FROM booking b
LEFT JOIN room r on r.id=b.room_no
WHERE r.room_type='single'
AND b.booking_date between '2016-12-01' AND '2016-12-31'
) l
LEFT JOIN
(
SELECT r.id AS id, b.booking_date AS bkr
FROM booking b
LEFT JOIN room r ON r.id=b.room_no
WHERE r.room_type='single'
AND b.booking_date BETWEEN '2016-12-01' and '2016-12-31'
) r
ON l.id = r.id
AND EXTRACT(DAY from r.bkr) - EXTRACT(DAY from l.bkl) >= l.nl
GROUP BY l.id, l.bkl, 
DATE_ADD(l.bkl, INTERVAL l.nl DAY)
) f

WHERE avail_days >= 3 or avail_days is null
ORDER BY avail DESC
LIMIT 1

```

### 15. Gross income by week. Money is collected from guests when they leave. For each Thursday in November and December 2016, show the total amount of money collected from the previous Friday to that day, inclusive.

```sql
SELECT
    DATE_ADD(MAKEDATE(2016, 7), INTERVAL WEEK(DATE_ADD(booking.booking_date, INTERVAL booking.nights - 5 DAY), 0) WEEK) AS i,
    SUM(booking.nights * rate.amount) + SUM(e.amount)AS Total
FROM booking
   JOIN rate 
   ON (booking.occupants = rate.occupancy AND booking.room_type_requested = rate.room_type)
LEFT JOIN
   (SELECT booking_id, SUM(amount) as amount
    FROM extra GROUP BY booking_id) AS e
    ON (e.booking_id = booking.booking_id)
    GROUP BY i
 ```
