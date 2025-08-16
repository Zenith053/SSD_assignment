# ASSIGNMENT\_2: Database Setup and Execution Guide

## 1. Open MySQL Client

Use **MySQL Workbench**, CLI, or any other MySQL client.

## 2. Create and Select Database

```sql
CREATE DATABASE IF NOT EXISTS ASSIGNMENT_2;
USE ASSIGNMENT_2;
```

## 3. Create Tables

```sql
-- Shows table
CREATE TABLE Shows (
    ShowID INT PRIMARY KEY,
    Title VARCHAR(100),
    Genre VARCHAR(50),
    ReleaseYear INT
);

-- Subscribers table
CREATE TABLE Subscribers (
    SubscriberID INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    SubscriberName VARCHAR(100),
    SubscriptionDate DATE DEFAULT CURRENT_DATE
);

-- WatchHistory table
CREATE TABLE WatchHistory (
    HistoryID INT PRIMARY KEY,
    ShowID INT,
    SubscriberID INT,
    WatchTime INT,
    FOREIGN KEY (ShowID) REFERENCES Shows(ShowID),
    FOREIGN KEY (SubscriberID) REFERENCES Subscribers(SubscriberID)
);
```

## 4. Insert Initial Data

```sql
INSERT INTO Shows (ShowID, Title, Genre, ReleaseYear) VALUES
(1, 'Stranger Things', 'Sci-Fi', 2016),
(2, 'The Crown', 'Drama', 2016),
(3, 'The Witcher', 'Fantasy', 2019);

INSERT INTO Subscribers (SubscriberID, SubscriberName, SubscriptionDate) VALUES
(1, 'Emily Clark', '2023-01-10'),
(2, 'Chris Adams', '2023-02-15'),
(3, 'Jordan Smith', '2023-03-05');

INSERT INTO WatchHistory (HistoryID, SubscriberID, ShowID, WatchTime) VALUES
(1, 1, 1, 100),
(2, 1, 2, 10),
(3, 2, 1, 20),
(4, 2, 2, 40),
(5, 2, 3, 10),
(6, 3, 2, 10),
(7, 3, 1, 10);
```

## 5. Create Stored Procedures

```sql
DROP PROCEDURE IF EXISTS ListAllSubscribers;
DROP PROCEDURE IF EXISTS GetWatchHistoryBySubscriber;
DROP PROCEDURE IF EXISTS AddSubscriberIfNotExists;
DROP PROCEDURE IF EXISTS SendWatchTimeReports;
DROP PROCEDURE IF EXISTS SendReport;
```

### 5.1 ListAllSubscribers

```sql
DELIMITER //
CREATE PROCEDURE ListAllSubscribers()
BEGIN
    DECLARE subs_name VARCHAR(255);
    DECLARE done INT DEFAULT 0;
    DECLARE subs_cursor CURSOR FOR
        SELECT SubscriberName FROM Subscribers;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

    OPEN subs_cursor;

    read_loop: LOOP
        FETCH subs_cursor INTO subs_name;
        IF done = 1 THEN LEAVE read_loop; END IF;
        SELECT subs_name;
    END LOOP;

    CLOSE subs_cursor;
END //
DELIMITER ;
```

### 5.2 GetWatchHistoryBySubscriber

```sql
DELIMITER //
CREATE PROCEDURE GetWatchHistoryBySubscriber(IN sub_id INT)
BEGIN
    SELECT
        s.SubscriberName,
        sh.Title AS ShowTitle,
        wh.WatchTime
    FROM WatchHistory wh
    JOIN Shows sh ON wh.ShowID = sh.ShowID
    JOIN Subscribers s ON wh.SubscriberID = s.SubscriberID
    WHERE wh.SubscriberID = sub_id;
END //
DELIMITER ;
```

### 5.3 AddSubscriberIfNotExists

```sql
DELIMITER //
CREATE PROCEDURE AddSubscriberIfNotExists(IN subName VARCHAR(100))
BEGIN
    IF NOT EXISTS (
        SELECT SubscriberID
        FROM Subscribers
        WHERE SubscriberName = subName
    ) THEN
        INSERT INTO Subscribers (SubscriberName, SubscriptionDate)
        VALUES (subName, CURRENT_DATE);
    END IF;
END //
DELIMITER ;
```

### 5.4 SendWatchTimeReports

```sql
DELIMITER //
CREATE PROCEDURE SendWatchTimeReports()
BEGIN
    SELECT
        s.SubscriberName,
        sh.Title AS ShowTitle,
        wh.WatchTime
    FROM WatchHistory wh
    JOIN Shows sh ON wh.ShowID = sh.ShowID
    JOIN Subscribers s ON wh.SubscriberID = s.SubscriberID
    ORDER BY s.SubscriberName, sh.Title;
END //
DELIMITER ;
```

### 5.5 SendReport

```sql
DELIMITER //
CREATE PROCEDURE SendReport()
BEGIN
    DECLARE done INT DEFAULT 0;
    DECLARE subId INT;
    DECLARE cur_subs CURSOR FOR
        SELECT DISTINCT SubscriberID FROM WatchHistory;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

    OPEN cur_subs;

    read_loop: LOOP
        FETCH cur_subs INTO subId;
        IF done THEN LEAVE read_loop; END IF;
        CALL GetWatchHistoryBySubscriber(subId);
    END LOOP;

    CLOSE cur_subs;
END //
DELIMITER ;
```

## 6. Execute Procedures

```sql
CALL ListAllSubscribers();
CALL GetWatchHistoryBySubscriber(1);
CALL AddSubscriberIfNotExists('Peeyush');
CALL SendWatchTimeReports();
```

## 7. Notes

- Ensure you are using the database: `USE ASSIGNMENT_2;`
- AUTOCOMMIT should be enabled, or commit manually if needed.
- Run scripts in this order: Tables → Inserts → Procedures → Procedure Calls.
- This ensures foreign key constraints and default values work correctly.

