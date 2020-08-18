*Tables:* *[link to tables of Musicians](https://db.grussell.org/ermusician.html)*

# Easy Questions

### 1. Give the organiser's name of the concert in the Assembly Rooms after the first of Feb, 1997.

```sql

SELECT m.m_name
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

av AS (SELECT can_play, cannot_play, can_play/(cannot_play+can_play) AS average FROM _null,not_null)

SELECT m_no, m_name, different_ins, average FROM diff_instruments, av
```
# Medium Questions

### 6. List the names, dates of birth and the instrument played of living musicians who play a instrument which Theo also plays.

```sql
WITH living_musicians AS (SELECT * FROM musician
WHERE died IS NULL),

list_Theo AS (SELECT perf_no, m_name, born, instrument FROM performer p
JOIN musician m ON p.perf_is=m.m_no
WHERE m_name LIKE 'Theo%'),

a AS (SELECT * FROM performer p
WHERE p.instrument IN (SELECT instrument FROM list_Theo) 
AND p.perf_is IN (SELECT m_no FROM living_musicians))

SELECT m_name, DATE_FORMAT(born, '%Y-%m-%d') born_date, instrument FROM a
JOIN musician m ON a.perf_is=m.m_no
WHERE m_name NOT LIKE 'Theo%'
```

### 7. List the name and the number of players for the band whose number of players is greater than the average number of players in each band.

```sql
WITH a AS (SELECT band_name, COUNT(*) count FROM band b
JOIN plays_in p ON b.band_no=p.band_id
GROUP BY band_name),

av AS (SELECT SUM(count)/COUNT(band_name) AS avg FROM a)

SELECT * FROM a, av
WHERE count> avg
```

### 8. List the names of musicians who both conduct and compose and live in Britain.

```sql
WITH a AS (SELECT * FROM performance p
JOIN musician m ON p.conducted_by=m.m_no
JOIN composer c ON m.m_no=c.comp_is
WHERE comp_is=conducted_by)

SELECT DISTINCT m_name FROM a
JOIN place pl ON a.living_in=pl.place_no
WHERE place_country IN ('England', 'Scotland')
```

### 9. Show the least commonly played instrument and the number of musicians who play it.

```sql
WITH least_played AS (SELECT instrument, COUNT(*) count FROM performer
GROUP BY instrument
HAVING count <= 1)

SELECT m_name, lp.instrument FROM performer p
JOIN least_played lp ON p.instrument=lp.instrument
JOIN musician m ON perf_is = m.m_no
WHERE p.instrument=lp.instrument
```

### 10. List the bands that have played music composed by Sue Little; Give the titles of the composition in each case.

```sql
SELECT band_name, c_title FROM band b
JOIN performance p ON b.band_no=p.gave
JOIN composition c ON p.performed=c.c_no
JOIN has_composed hc ON c.c_no=hc.cmpn_no
JOIN composer cp ON hc.cmpr_no=cp.comp_no
JOIN musician m ON cp.comp_is=m.m_no
WHERE m.m_name='Sue Little'
ORDER BY band_name
```

# Hard Questions

### 11. List the name and town of birth of any performer born in the same city as James First.

```sql
WITH born_James AS (SELECT * FROM musician m 
JOIN place pl ON m.born_in=pl.place_no
WHERE m.m_name='James First'),

perf AS (SELECT * FROM performer
JOIN musician m ON performer.perf_is=m.m_no)

SELECT DISTINCT perf.m_name, place_town FROM perf, born_James
WHERE perf.born_in = born_James.born_in 
AND perf.m_name != 'James First'
```

### 12. Create a list showing for EVERY musician born in Britain the number of compositions and the number of instruments played.

```sql
WITH born_in_Britain AS (SELECT * FROM musician m
JOIN place pl ON m.born_in=pl.place_no
WHERE pl.place_country IN ('England', 'Scotland')),

comp AS (SELECT br.m_no, count(distinct hc.cmpn_no) count FROM born_in_Britain br
LEFT JOIN composer cp ON br.m_no=cp.comp_is
LEFT JOIN has_composed hc ON cp.comp_no=hc.cmpr_no
GROUP BY 1),

inst_count AS (
SELECT br.m_no, count(distinct p.instrument) count
FROM born_in_Britain br
LEFT JOIN performer p on br.m_no = p.perf_is
GROUP BY 1
)
SELECT m.m_name, comp.count comp_count, coalesce(ic.count, 0) inst_count
FROM comp
LEFT JOIN inst_count ic on ic.m_no = comp.m_no
LEFT JOIN musician m on m.m_no = comp.m_no
ORDER BY 1
```

### 13. Give the band name, conductor and contact of the bands performing at the most recent concert in the Royal Albert Hall.

```sql
WITH royal_concert AS (SELECT * FROM concert c
WHERE c.concert_venue = 'Royal Albert Hall'),

a AS (SELECT band_name, m_name, band_contact FROM royal_concert rc
JOIN performance p ON rc.concert_no=p.performed_in
JOIN musician m ON p.conducted_by=m.m_no
JOIN band b ON p.gave=b.band_no)

SELECT band_name, a.m_name conductor, m.m_name contact FROM a
JOIN musician m ON a.band_contact=m.m_no 
```


### 14. Give a list of musicians associated with Glasgow. Include the name of the musician and the nature of the association - one or more of 'LIVES_IN', 'BORN_IN', 'PERFORMED_IN' AND 'IN_BAND_IN'.

```sql
WITH born_in AS (
SELECT
    m_no
FROM musician m
JOIN place pl ON m.born_in=pl.place_no
WHERE pl.place_town='Glasgow'),

lives_in AS (
SELECT
    m_no
FROM musician m
JOIN place pl ON m.living_in=pl.place_no
where pl.place_town='Glasgow'),

performed_in AS (
SELECT
    DISTINCT m_no
FROM performance p
    JOIN concert c ON p.performed_in=c.concert_no
    JOIN place pl ON c.concert_in=pl.place_no
    JOIN musician m ON p.conducted_by=m.m_no
WHERE pl.place_town='Glasgow'),

band_in AS (
SELECT
    DISTINCT p.perf_is as m_no
FROM band b 
JOIN plays_in pi on pi.band_id = b.band_no
JOIN performer p on pi.player = p.perf_no
JOIN place pl on pl.place_no = b.band_home
WHERE pl.place_town = 'Glasgow')

SELECT
    m.m_name,
    IF(ISNULL(born_in.m_no), 'No', 'Yes') born_in,
    IF(ISNULL(lives_in.m_no), 'No', 'Yes') lives_in,
    IF(ISNULL(performed_in.m_no), 'No', 'Yes') performed_in,
    IF(ISNULL(band_in.m_no), 'No', 'Yes') band_in
FROM musician m
LEFT JOIN born_in ON m.m_no=born_in.m_no
LEFT JOIN lives_in ON m.m_no=lives_in.m_no
LEFT JOIN performed_in ON m.m_no=performed_in.m_no
LEFT JOIN band_in ON m.m_no=band_in.m_no
```

### 15. Jeff Dawn plays in a band with someone who plays in a band with Sue Little. Who is it and what are the bands?

*There is no a band that Jeff Dawn plays with Sue Little. You may find the names that Jeff Dawn plays in a band with via query below.

```sql
WITH jeff_plays AS (SELECT * FROM band b
JOIN plays_in pi ON b.band_no=pi.band_id
JOIN musician m ON pi.player=m.m_no
WHERE m_name='Jeff Dawn')

SELECT m_name FROM band b
JOIN plays_in pi ON b.band_no=pi.band_id
JOIN musician m ON pi.player=m.m_no
WHERE b.band_name IN (SELECT jeff_plays.band_name FROM jeff_plays) AND m_name != 'Jeff Dawn'
```


