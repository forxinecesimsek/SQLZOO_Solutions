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
