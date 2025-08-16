# SSD_assignment
ASSIGNMENT_2 Database Setup and Execution Guide
================================================

1. Open MySQL client
--------------------
Use MySQL Workbench, CLI, or any other MySQL client.

2. Create and select database
------------------------------
Run the following commands:

CREATE DATABASE IF NOT EXISTS ASSIGNMENT_2;
USE ASSIGNMENT_2;

3. Create tables
----------------
Run the script to create the following tables in this order:

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

4. Insert initial data
----------------------
Run the script to insert sample data into Shows, Subscribers, and WatchHistory:

-- Example Inserts
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

5. Create stored procedures
---------------------------
Drop existing procedures if any, then create them:

-- Drop procedures
DROP PROCEDURE IF EXISTS ListAllSubscribers
DROP PROCEDURE IF EXISTS GetWatchHistoryBySubscriber;
DROP PROCEDURE IF EXISTS AddSubscriberIfNotExists;
DROP PROCEDURE IF EXISTS SendWatchTimeReports;
-- This is so that the sql command run hazzle free

----------------------------------------------------------------------------------------
1>
delimiter // 
create procedure ListAllSubscribers()
	begin
    declare subs_name varchar(255);
    declare done int default 0;
    declare subs_cursor cursor for 
    select SubscriberName from Subscribers; 
	declare continue handler for not found set done = 1;
    open subs_cursor;
read_loop: LOOP
		Fetch subs_cursor into subs_name;
		if done =1 then leave read_loop;
        end if;
        select subs_name;
	end LOOP;
    close subs_cursor;
       
    end //
delimiter ;
--------------------------------------------------------------------------------
-- Create procedures
2>
DELIMITER //

create procedure GetWatchHistoryBySubscriber(IN sub_id INT)
	begin

	SELECT 
        s.SubscriberName,
        sh.Title AS ShowTitle,
        wh.WatchTime
    FROM 
        WatchHistory wh
    JOIN 
        Shows sh ON wh.ShowID = sh.ShowID
    JOIN 
        Subscribers s ON wh.SubscriberID = s.SubscriberID
    WHERE 
        wh.SubscriberID = sub_id; 
	        
    end //
    delimiter ;

---------------------------------------------------------------------------------

3> AddSubscriberIfNotExists(IN subName VARCHAR(100))
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
------------------------------------------------------------------------------------
4>
DELIMITER //
CREATE PROCEDURE SendWatchTimeReports()
BEGIN
    SELECT 
        s.SubscriberName,
        sh.Title AS ShowTitle,
        wh.WatchTime
    FROM 
        WatchHistory wh
    JOIN 
        Shows sh ON wh.ShowID = sh.ShowID
    JOIN 
        Subscribers s ON wh.SubscriberID = s.SubscriberID
    ORDER BY 
        s.SubscriberName, sh.Title;
END //

DELIMITER ;
-----------------------------------------------------------------------------------
5>
DELIMITER //

CREATE PROCEDURE SendReport()
BEGIN
    DECLARE done INT DEFAULT 0;
    DECLARE subId INT;
    DECLARE cur_subs CURSOR FOR
        SELECT DISTINCT SubscriberID
        FROM WatchHistory;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;
    OPEN cur_subs;

    read_loop: LOOP
        FETCH cur_subs INTO subId;
        IF done THEN
            LEAVE read_loop;
        END IF;
        CALL GetWatchHistoryBySubscriber(subId);
    END LOOP;

    CLOSE cur_subs;
END //

DELIMITER ;
---------------------------------------------------------------------------------------
6. Execute procedures
CALL ListAllSubscribers()
-- It uses cursor and thus goes row by row so the output might be
   just a single row which was read last
CALL GetWatchHistoryBySubscriber(IN SUB_ID)
-- It takes input as subscriber id and returns the shows watched by 
   that subscriber

CALL AddSubscriberIfNotExists('Peeyush');
-- It takes input as a varchar and checks if the subscriber by that
   name is present or not if not it adds.

-- Get full watch time report for all subscribers
CALL SendWatchTimeReports();

-- Get watch history for a specific subscriber
CALL GetWatchHistoryBySubscriber(1);



7. Notes
--------
- Ensure you are using the database ASSIGNMENT_2 (USE ASSIGNMENT_2;).  
- AUTOCOMMIT should be enabled, or commit manually if needed.  
- By default commit is only enabled for 3rd part.
- Run scripts in this order: tables → inserts → procedures → procedure calls.  
- This setup ensures all foreign key constraints and default values work correctly.
