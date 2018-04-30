## Query Plan Samples with analyzing Query Optimizer
### Join tables
  1. Run Task1
  2. Have a look at the results of the execution plan, and we'll see we've got a "Clustered Index Scan" here, and this one is going to have a look at the "Product" table, and another "Index Scan" here, this one is having a sales with a data table.
  
```sql
-- Task1 (actually joining multiple tables)
SELECT  pp.Name
FROM    Production.Product pp
JOIN    Sales.SalesOrderDetail ss
  ON    pp.ProductID=ss.ProductID
JOIN    Sales.SalesOrderHeader oh
  ON    ss.SalesOrderID=oh.SalesOrderID;
```
  Have a look at the results of the execution plan, and we'll see we've got a "Clustered Index Scan" here, and this one is going to have a look at the "Product" table, and another "Index Scan" here, his one is having a "Sales" with a data table.
  So, there is no reference whatsoever to the ''Sales Order Header" table. The reason for that is the optimizer is efficient enough to realize that actually, it's impossible for there to be any information from "Sales Order Header". So, if is not possible that there is information from that table,it won't actually add that table to the plan.

![SQL-Join-Query Plan](../Pictures/SQL-Join-Query_Plan.png)

**Index Scan:**

Since a scan touches every row in the table, whether or not it qualifies, the cost is proportional to the total number of rows in the table. Thus, a scan is an efficient strategy if the table is small or if most of the rows qualify for the predicate.

**Index Seek:**
  1. Since a seek only touches rows that qualify and pages that contain these qualifying rows, the cost is proportional to the number of qualifying rows and pages rather than to the total number of rows in the table.

  2. Index Scan is nothing but scanning on the data pages from the first page to the last page. If there is an index on a table, and if the query is touching a larger amount of data, which means the query is retrieving more than 50 percent or 90 percent of the data, and then the optimizer would just scan all the data pages to retrieve the data rows. If there is no index, then you might see a Table Scan (Index Scan) in the execution plan.

  3. Index seeks are generally preferred for the highly selective queries. What that means is that the query is just requesting a fewer number of rows or just retrieving the other 10 (some documents says 15 percent) of the rows of the table.

  4. In general query optimizer tries to use an Index Seek which means that the optimizer has found a useful index to retrieve recordset. But if it is not able to do so either because there is no index or no useful indexes on the table, then SQL Server has to scan all the records that satisfy the query condition.
  
```sql
SELECT * 
FROM    HumanResources.Employee 
WHERE   SickLeaveHours=50;

SELECT  CHECK_CLAUSE
FROM    INFORMATION_SCHEMA.CHECK_CONSTRAINTS
WHERE   CONSTRAINT_NAME = 'CK_Employee_SickLeaveHours';
____________________________________________________________
```

```sql
SELECT *
FROM sys.dm_exec_query_transformation_stats
ORDER BY name;

SELECT TOP 5 total_worker_time/execution_count AS [Avg CPU Time],  
Plan_handle, query_plan   
FROM sys.dm_exec_query_stats AS qs  
CROSS APPLY sys.dm_exec_query_plan(qs.plan_handle)  
ORDER BY total_worker_time/execution_count DESC;  
```

## Links
[Clustered and Nonclustered Indexes Described](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described?view=sql-server-2017)

[System Dynamic Management Views-Microsoft](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views?view=sql-server-2017)

[Execution Related Dynamic Management Views and Functions (Transact-SQL)](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/execution-related-dynamic-management-views-and-functions-transact-sql?view=sql-server-2017)

[sys.dm_exec_query_plan (Transact-SQL)](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-plan-transact-sql?view=sql-server-2017#examples)
