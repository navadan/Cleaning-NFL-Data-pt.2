# Cleaning-NFL-Data-pt.2
-- Let's take a quick look at the Basis Stats table and  Career Stats Defensive table

SELECT *
FROM [dbo].[Basic_Stats];

-- A few are empty for Birth Place and most rows are missing the Current Team section

SELECT *
FROM [dbo].[Career_Stats_Defensive];

-- Most rows are empty for the position column and there appears to be incomplete data identified as '--'

--Are there any players in the Basics table that are not included in the Career Stats Defensive table? 
SELECT DISTINCT [Player ID]
FROM Career_Stats_Defensive 
WHERE [Player ID] NOT IN 
	(SELECT DISTINCT [Player ID] FROM Basic_Stats);

-- All players in the Basic Stats tables are also in the Career Stats Defensive table 

--Let's take a closer look at the [Ints fot TDs] column from the Career Stats Defensive table

SELECT [Player Name],[Games Played],[Ints],[Ints for TDs]
FROM [dbo].[Career_Stats_Defensive];

SELECT [Player Name],[Games Played],[Ints],[Ints for TDs]
FROM [dbo].[Career_Stats_Defensive]
WHERE [Ints for TDs] > [Ints];

-- There are 4 records that have a higher # of Ints for TDs than regular Ints. That's odd

-- Produce a table that groups the [Ints for TDs] by value

SELECT [Ints for TDs], count(*)AS [# of records]
FROM [dbo].[Career_Stats_Defensive]
GROUP BY [Ints for TDs]
ORDER BY [# of records];

-- '--' is assigned to 19387 records (~ 81% of the total records)
-- The ins for TDs range from 0 to 4

-- Let's start cleaning our data. First create a copy of our table so that we can make adjustments

SELECT *
INTO Career_Stats_Defensive_Analysis
FROM[dbo].[Career_Stats_Defensive];

SELECT *
FROM [dbo].[Career_Stats_Defensive_Analysis];

-- Now we can add a new column and place the '--' values as NULL

ALTER TABLE [dbo].[Career_Stats_Defensive_Analysis]
ADD [Clean Ints for TDs] int NULL;

--Nice job! you created a new column. Now let's transfer the original column data over

UPDATE [dbo].[Career_Stats_Defensive_Analysis]
SET [Clean Ints for TDs] = CAST([Ints for TDs] AS int)
WHERE [Ints for TDs] != '--';

-- Now run two queries to show you the amount of records for each Int value
-- Compare the old column data with the new column data to make sure they match

SELECT [Ints for TDs], count(*) AS [# of records]
FROM [dbo].[Career_Stats_Defensive_Analysis]
GROUP BY [Ints for TDs]
ORDER BY [Ints for TDs];

SELECT [Clean Ints for TDs], count(*) AS [# of records]
FROM [dbo].[Career_Stats_Defensive_Analysis]
GROUP BY [Clean Ints for TDs]
ORDER BY [Clean Ints for TDs];

-- Both queries resulted in the same values
-- Now lets explore the Longest Int Return values 

SELECT COUNT([Longest Int Return])
FROM [dbo].[Career_Stats_Defensive_Analysis];
-- There are 23998 values total

SELECT [Longest Int Return], COUNT(*) AS [# of records]
FROM [dbo].[Career_Stats_Defensive_Analysis]
GROUP BY [Longest Int Return]
ORDER BY [Longest Int Return];

-- 19448 records are missing (~ 81% of all values) 
-- Note that we got the same percentage for missing values from the Ints for TD column
-- ALSO, we see many values with a 'T' attached tp them. We could assume that this means TD
-- We can run a query to verify that

SELECT [Player Name], [Games Played], [Clean Ints for TDs], [Longest Int Return]
FROM [dbo].[Career_Stats_Defensive_Analysis]
WHERE [Longest Int Return] LIKE '%T' AND [Clean Ints for TDs] < 1;

-- No players have a T in the Longest Int Return and a value below 1 in the Clean Int for TDs column
-- We can confirm that our previous assumption is valid
-- Let's create a new column for our our Longest Int Return

ALTER TABLE [dbo].[Career_Stats_Defensive_Analysis]
ADD [Clean Longest Int Return] int NULL;

ALTER TABLE [dbo].[Career_Stats_Defensive_Analysis]
ADD [Clean Longest Int Return was TD] bit NULL;

-- Input the data from the old column into the new column without the 'T'
UPDATE [dbo].[Career_Stats_Defensive_Analysis]
SET [Clean Longest Int Return] = CAST([Longest Int Return] AS int)
WHERE [Longest Int Return] != '--' AND [Longest Int Return] NOT LIKE '%T';

-- Use parsing to manipulate the string data. You should be able to view the data in seperate columns 
-- without the 'T'
SELECT	[Longest Int Return], LEN([Longest Int Return]) AS 'String Length', 
		LEFT([Longest Int Return], (LEN([Longest Int Return])-1)) AS 'Clean Longest Reception'
FROM [dbo].[Career_Stats_Defensive_Analysis]
WHERE [Longest Int Return] LIKE '%T'
ORDER BY 'String Length' DESC;

--	Parse the longest reception yards and store in clean longest reception

UPDATE [dbo].[Career_Stats_Defensive_Analysis]
SET [Clean Longest Int Return] = 
		CAST(LEFT([Longest Int Return], (LEN([Longest Int Return])-1)) AS int)
WHERE [Longest Int Return] LIKE '%T';

--	Set [Clean Longest Reception as TD] = 1 when longest reception is a TD
-- (Use an update statement)
UPDATE [dbo].[Career_Stats_Defensive_Analysis]
SET [Clean Longest Int Return was TD] = 1
WHERE [Longest Int Return] LIKE '%T';

--	Set [Clean Longest Reception as TD] = 0 when longest reception is NOT a TD
UPDATE [dbo].[Career_Stats_Defensive_Analysis]
SET [Clean Longest Int Return was TD] = 0
WHERE [Longest Int Return] <> '--' AND [Longest Int Return] NOT LIKE '%T';

-- Write a SELECT statement to view the values in the [Clean Longest Int Return was a TD] 
-- Include the Longest Int Return (Original data) 
SELECT [Longest Int Return], [Clean Longest Int Return], [Clean Longest Int Return was TD]
FROM [dbo].[Career_Stats_Defensive_Analysis]
WHERE [Longest Int Return] >= '0'
ORDER BY [Clean Longest Int Return was TD];

DROP TABLE [dbo].[Career_Stats_Defensive_Analysis];
