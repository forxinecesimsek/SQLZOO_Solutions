*Tables:* *[link to tables of Musicians](https://sqlzoo.net/wiki/Musicians_easy_questions)*

# Easy Questions

### 1. Give the organiser's name of the concert in the Assembly Rooms after the first of Feb, 1997.

```sql

SELECT m.m_name as organiser
FROM   concert c
JOIN   musician m ON m.m_no = c.concert_orgniser
WHERE  concert_venue = 'Assembly Rooms'
AND    con_date > date'1997-02-01'
```


### 2. Find all the performers who played guitar or violin and were born in England.

```sql

WITH performers AS(SELECT * from performer p
INNER JOIN musician m ON p.perf_no=m.m_no)

SELECT m_name FROM performers
INNER JOIN place pl ON performers.born_in=pl.place_no
WHERE instrument IN ('guitar', 'violin') AND place_country ='England'
```


### 3. List the names of musicians who have conducted concerts in USA together with the towns and dates of these concerts.

```sql

WITH prfmnc AS (SELECT * FROM performance
JOIN place pl ON performance.performed_in=pl.place_no
WHERE place_country='USA')

SELECT m_name, place_town,con_date FROM prfmnc 
INNER JOIN concert c ON prfmnc.conducted_by=c.concert_orgniser
INNER JOIN musician m ON m.m_no = c.concert_orgniser
GROUP BY m_name, place_town,con_date
```

### 4. How many concerts have featured at least one composition by Andy Jones? List concert date, venue and the composition's title.

```sql

SELECT c.c_title,cn.concert_venue, DATE_FORMAT(cn.con_date, '%Y-%m-%d') con_date
FROM performance p
JOIN composition c ON c.c_no = p.performed
JOIN concert cn ON cn.concert_no = p.performed_in
WHERE p.performed = (
SELECT hc.cmpn_no
FROM musician m
    JOIN composer c ON m.m_no = c.comp_is
    JOIN has_composed hc ON c.comp_no = hc.cmpr_no
WHERE m.m_name = 'Andy Jones'
)
```

### 5. list the different instruments played by the musicians and avg number of musicians who play the instrument.

```sql
WITH a AS (SELECT * FROM performer p
JOIN musician m ON p.perf_is=m.m_no),

diff_instruments AS (SELECT DISTINCT m_no, m_name, COUNT(instrument) different_ins FROM a
GROUP BY m_name),

_null AS (SELECT COUNT(*) cannot_play FROM musician m
LEFT JOIN diff_instruments ON m.m_no=diff_instruments.m_no
WHERE diff_instruments.m_name IS NULL),

not_null AS (SELECT COUNT(*) can_play FROM musician m
LEFT JOIN diff_instruments ON m.m_no=diff_instruments.m_no
WHERE diff_instruments.m_name IS NOT NULL),

av AS (SELECT can_play, cannot_play, cannot_play/can_play AS average FROM _null,not_null)

SELECT m_no, m_name, different_ins, average FROM diff_instruments, av
```