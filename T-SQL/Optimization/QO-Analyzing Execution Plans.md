# Analyzing Execution Plans
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

-- Task 2 
-- Execute the following query to rebuild the statistics held for the 
-- Proseware.Campaign and Proseware.CampaignResponse tables.
ALTER TABLE Proseware.Campaign REBUILD
GO
ALTER TABLE Proseware.CampaignResponse REBUILD;
GO

-- Task 3
-- Re-run the query under the heading for task 1 and collect a new actual execution plan. 
-- Compare the new execution plan to the plan you saved in task 1 (as plan1.sqlplan)


