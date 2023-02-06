```SQL
USE sailors;
CREATE table sailors(
    sid INT PRIMARY KEY,
    sname VARCHAR(25) NOT NULL,
    rating INT NOT NULL,
    age INT NOT NULL
);
CREATE table boat(
    bid INT PRIMARY KEY,
    bname VARCHAR(25) NOT NULL,
    color VARCHAR(25) NOT NULL
);
CREATE table rservers(
    sid INT NOT NULL,
    bid INT NOT NULL,
    sdate date NOT NULL,
    FOREIGN KEY (sid) REFERENCES sailors(sid) ON DELETE CASCADE,
    FOREIGN KEY (bid) REFERENCES boat(bid) ON DELETE CASCADE
);
INSERT INTO sailors
VALUES (1, "Sailor_1", 4, 23),
    (2, "Sailor_2", 2, 25),
    (3, "Sailor_3", 5, 26),
    (4, "Sailor_4", 4, 30),
    (5, "Sailor_5", 5, 25);
INSERT INTO boat
VALUES (1, "Billy", "Black"),
    (2, "Pilly", "Red"),
    (3, "Trilly", "Green"),
    (4, "Willy", "White"),
    (5, "Lilly", "Blue");
INSERT INTO rservers
VALUES (1, 4, "2020-04-05"),
    (2, 2, "2019-12-16"),
    (3, 1, "2020-05-14"),
    (4, 3, "2019-08-30"),
    (5, 5, "2021-01-01");
```
  
1. Find the colours of boats reserved by Albert
```SQL
SELECT B.color
FROM sailors S,
    boat B,
    rservers R
WHERE S.sid = R.sid
    AND B.bid = R.bid
    AND S.sname = "Albert";
-- or 
SELECT color
FROM boat
WHERE bid IN (
        SELECT bid
        FROM rservers
        WHERE sid IN (
                SELECT sid
                FROM sailors
                WHERE sname = 'Albert'
            )
    );
```
2. Find all sailor id’s of sailors who have a rating of at least 8 or reserved boat 103
```SQL

SELECT sid
FROM sailors
WHERE rating >= 8
UNION
SELECT sid
FROM rservers
WHERE bid = 103;
-- or
SELECT sid
FROM sailors
WHERE rating >= 8
    OR sid IN (
        SELECT sid
        FROM rservers
        WHERE bid = 103
    );
```
3. Find the names of sailors who have not reserved a boat whose name contains the string “storm”. 
Order the names in ascending order
```SQL
SELECT sname
FROM sailors
WHERE sid not in (
        SELECT rservers.sid
        FROM boat,
            rservers
        WHERE boat.bid = rservers.bid
            AND bname like '%storm%'
    )
ORDER BY sname;
```
4. Find the names of sailors who have reserved all boats
```SQL
SELECT sname
FROM sailors
WHERE NOT EXISTS (
        SELECT bid
        FROM boat
        WHERE bid NOT IN (
                SELECT bid
                FROM rservers
                WHERE sid = sailors.sid
            )
    );
```
5. Find the name and age of the oldest sailor.
```SQL
SELECT sname,
    age
FROM sailors
WHERE age = (
        SELECT max(age)
        FROM sailors
    );
```
6. For each boat which was reserved by at least 5 sailors with age >= 40, 
-- find the boat id and the average age of such sailors
```SQL
SELECT B.bid,
    avg(S.age)
FROM sailors S,
    boat b,
    rservers R
WHERE S.sid = R.sid
    AND B.bid = R.bid
    AND S.age >= 40
GROUP BY(R.bid)
having count(B.bid) >= 5;
```
7. A view that shows names and ratings of all sailors sorted by rating in descending order.
```SQL
CREATE view namesandratings AS(
    SELECT sname,
        rating
    FROM sailors
    ORDER BY rating DESC
);
```
8. CREATE a view that shows the names of the sailors who have reserved a boat on a given date.
   
- enter any date insted of 2021-01-01
```SQL
CREATE view new_year_rservers AS (
    SELECT sname
    FROM sailors S,
        rservers R
    WHERE S.sid = R.sid
        AND sdate = '2021-01-01'
);
```
9. CREATE a view that shows the names and colours of all the boats that have been reserved by a sailor with a specific rating.
- enter any rating insted 2
```SQL
CREATE view NameAndColor AS (
    SELECT sname,
        color
    FROM sailors S,
        rservers R,
        boat B
    WHERE S.sid = R.sid
        AND B.bid = R.bid
        AND rating = 2
);
```
10. A trigger that prevents boats FROM being deleted If they have active reservations. 
- change delimiter to use this     
```SQL
CREATE trigger prevent_boat_delete BEFORE DELETE ON boat for each ROW if (
    SELECT count(*)
    FROM rservers
    WHERE bid = old.bid
) > 0 THEN SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = "Can't delete the reserved boat ";
END IF;
```
11.	A trigger that prevents sailors with rating less than 3 FROM reserving a boat.
- change delimiter to use this 
``` SQL
CREATE trigger prevent_boat_reserve BEFORE
INSERT ON rservers for each ROW if (
        SELECT rating
        FROM sailors
        WHERE sid = new.sid
    ) < 3 THEN SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = "Can't reserve boat by less rated sailor";
END IF;
```

12.	A trigger that deletes all expired reservations. 
- we can't add trigger on same table to modify that table rows that is trigger on rservers to delete rservers rows Therefore use any other table on any operation
 - Correct way to delete expired records is with event schedulers not triggers

```SQL 
CREATE trigger delete_expired
AFTER
INSERT ON boat for each row
DELETE FROM rservers
WHERE sdate < CURRENT_DATE;
```
