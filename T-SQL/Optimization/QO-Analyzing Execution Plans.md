# Analyzing Execution Plans
## Prepare the Lab Environment
```sql
USE AdventureWorks2017;
GO

IF EXISTS(select * from sys.objects
where name='up_Campaign_Replace')
BEGIN
	DROP PROC Proseware.up_Campaign_Replace
END
GO

IF EXISTS(select * from sys.objects
where name='up_Campaign_Report')
BEGIN
	DROP PROC Proseware.up_Campaign_Report
END
GO

IF EXISTS(select * from sys.objects
where name='up_CampaignResponse_Add')
BEGIN
	DROP PROC Proseware.up_CampaignResponse_Add
END
GO

IF EXISTS(select * from sys.objects
where name='CampaignResponse')
BEGIN
	DROP TABLE Proseware.CampaignResponse
END
GO

IF EXISTS(select * from sys.objects
where name='Campaign')
BEGIN
	DROP TABLE Proseware.Campaign
END
GO

IF EXISTS(select * from sys.schemas
where name='Proseware')
BEGIN
	DROP schema Proseware
END
GO

-------------  Build Tables --------------
CREATE SCHEMA Proseware;
GO
CREATE TABLE Proseware.Campaign
( 	CampaignID int PRIMARY KEY,
	CampaignName varchar(20) NOT NULL,
	CampaignTerritoryID int NOT NULL,
	CampaignStartDate date NOT NULL,
	CampaignEndDate date NOT NULL
)
GO

INSERT Proseware.Campaign
(CampaignID, CampaignName, CampaignTerritoryID, CampaignStartDate, CampaignEndDate)
SELECT TOP (10000)
	ROW_NUMBER() OVER (ORDER BY a.name, b.name),
	CAST(1000000 +ROW_NUMBER() OVER (ORDER BY a.name, b.name) AS nvarchar(20)),
	(ROW_NUMBER() OVER (ORDER BY a.name, b.name) % 10) + 1,
	DATEADD(dd, ROW_NUMBER() OVER (ORDER BY a.name, b.name) % 3650, '2006-01-01'),
	DATEADD(dd, (ROW_NUMBER() OVER (ORDER BY a.name, b.name) % 3650) + 30, '2006-01-01')
FROM  Production.Product AS a
CROSS JOIN Production.Product AS b
GO

CREATE TABLE Proseware.CampaignResponse
(	CampaignResponseID int IDENTITY(1,1) PRIMARY KEY,
	CampaignID int NOT NULL,
	ResponseDate date NOT NULL,
	ConvertedToSale bit NOT NULL,
	ConvertedSaleValueUSD decimal(20,2) NULL
)

GO

ALTER TABLE Proseware.CampaignResponse WITH 
	CHECK ADD  CONSTRAINT FK_CampaignResponse_Campaign FOREIGN KEY (CampaignID)
	REFERENCES Proseware.Campaign(CampaignID)
GO
ALTER TABLE Proseware.CampaignResponse 
	CHECK CONSTRAINT FK_CampaignResponse_Campaign
GO
;

WITH myCTE
AS
(
	SELECT c.*,
	ROW_NUMBER() OVER (PARTITION BY c.CampaignID ORDER BY b.name ) as rn1,
	CASE WHEN c.CampaignID % 10 < 4 THEN 1 ELSE 0 END AS rnd1
	FROM Proseware.Campaign AS c
	CROSS JOIN Production.Product AS b
)
INSERT Proseware.CampaignResponse
(CampaignID, ResponseDate, ConvertedToSale, ConvertedSaleValueUSD)
	SELECT c.CampaignID,
	DATEADD(dd, rn1 % 40, c.CampaignStartDate),
	c.rnd1,
	CASE WHEN rnd1 = 1 THEN rn1 * 1.99 ELSE NULL END
	FROM myCTE AS c
	WHERE c.rn1 <= c.CampaignID % 1000
GO

UPDATE STATISTICS Proseware.Campaign 	
	WITH ROWCOUNT = 10, PAGECOUNT=  10;
UPDATE STATISTICS Proseware.CampaignResponse 
	WITH ROWCOUNT = 10, PAGECOUNT=  10;
GO

CREATE PROCEDURE Proseware.up_CampaignResponse_Add
(
	@CampaignName int,
	@ResponseDate date,
	@ConvertedToSale bit,
	@ConvertedSaleValueUSD decimal(20,2)
)
WITH RECOMPILE AS
	SET NOCOUNT ON	
	DECLARE @CampaignId int;

	-- lookup CampaignId
	SELECT @CampaignId = CampaignID
	FROM Proseware.Campaign
	WHERE CampaignName = @CampaignName;

	--insert values
	INSERT Proseware.CampaignResponse
	(CampaignID, ResponseDate, ConvertedToSale, ConvertedSaleValueUSD)
	VALUES
	(@CampaignId,@ResponseDate, @ConvertedToSale, @ConvertedSaleValueUSD);
GO
```
## Collect an Actual Execution Plan

1. Select the query under the comment which begins Task 1. On the Query menu, click Include Actual Execution Plan.
2. Click Execute. Note the execution time.
3. In the Results pane, on the Execution plan tab, right-click the graphical execution plan, and then click Save Execution Plan As.
4. In the Save As dialog box, save the file as plan1.sqlplan on your desktop.
5. On the Execution plan tab, scroll to the far right-hand side of the actual query plan. Position the cursor over the query plan operator in the top right-hand corner, the name of which starts Clustered Index Scan (Clustered). In the pop-up that appears, notice the difference between Estimated Number of Rows and Actual Number of Rows.

This type of issue is caused by a poor estimate of cardinality. The table statistics indicate that the tables involved have very few rows, but in reality, the row count is much higher. Updating statistics for the tables involved will improve performance.

```sql
-- Task 1 
-- Collect an actual execution plan for the query shown under the heading for task 1. 
-- Save the actual query plan on your desktop as plan1.sqlplan. Note the execution time.
USE AdventureWorks2017;
GO
SELECT c.CampaignName, c.CampaignStartDate, c.CampaignEndDate,
SUM(cr.ConvertedSaleValueUSD) AS SalesValue
FROM Proseware.Campaign AS c
JOIN Proseware.CampaignResponse AS cr
ON cr.CampaignID = c.CampaignID
WHERE cr.ConvertedToSale = 1
GROUP BY c.CampaignName, c.CampaignStartDate, c.CampaignEndDate
OPTION (RECOMPILE); --RECOMPILE option is used to ensure the plan is not drawn from the cache
```
## Rebuild Table Statistics
1. Select the query under the comment which begins Task 2 and click Execute.
```sql
-- Task 2 
-- Execute the following query to rebuild the statistics held for the 
-- Proseware.Campaign and Proseware.CampaignResponse tables.
ALTER TABLE Proseware.Campaign REBUILD
GO
ALTER TABLE Proseware.CampaignResponse REBUILD;
GO
```
## Compare the Execution Plan

1. Select the query under the comment which begins Task 1. On the Query menu, ensure Include Actual Execution Plan is selected.
2. Click Execute. Note the execution time.
3. On the Execution plan tab, scroll to the far right-hand side of the actual query plan. Position the cursor over the query plan operator in the top right-hand corner, the name of which starts Clustered Index Scan (Clustered). In the pop-up that appears, notice the similarity between Estimated Number of Rows and Actual Number of Rows. Note that the estimated and actual row counts now match almost exactly. The query will execute faster than it did previously. The execution plan includes a suggestion for an index that would further improve query performance.
4. On the Execution plan tab, right-click the graphical execution plan and click Compare Showplan.
5. In the Open dialog box, browse to your desktop, select plan1.sqlplan and click Open. The new and old query execution plans will open side-by-side.
6. When you have finished your comparison, close the Showplan Comparison tab.

```sql
-- Task 3
-- Re-run the query under the heading for task 1 and collect a new actual execution plan. 
-- Compare the new execution plan to the plan you saved in task 1 (as plan1.sqlplan)
```

