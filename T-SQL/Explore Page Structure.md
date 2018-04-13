## Explore Page Structure

1. Start Microsoft SQL Server Management Studio and connect to your database instance.
2. Click New Query, select the AdventureWorksLT database, type the following Transact-SQL into the query window, then click Execute:
``` sql
SELECT db_name(database_id) AS Database_Name, 
	object_name([object_id]) AS Table_Name, 
	allocation_unit_type, 
	allocation_unit_type_desc,
	allocated_page_file_id, 
	allocated_page_page_id, 
	page_type, 
	page_type_desc 
FROM  sys.dm_db_database_page_allocations(db_id('AdventureWorks2017'),NULL,NULL,NUll,'DETAILED')
WHERE object_name([object_id])='Product'
AND   page_type IS NOT NULL;
```
3. Examine the query results, nothing that there are three page types: IAM pages, index pages, and data pages.

## Explore the Index Statistics

1. In the Microsoft SQL Server Management Studio query window, type the following Transact-SQL to examine the indexes for the Product table, then highlight the code and click Execute:
``` sql
SELECT db_name(database_id) AS Database_Name, 
	object_name([object_id]) AS Table_Name, 
	index_id,
	partition_number,
	index_type_desc,
	alloc_unit_type_desc,
	index_depth,
	index_level,
	avg_fragmentation_in_percent,
	fragment_count,
	avg_fragment_size_in_pages,
	page_count
FROM  sys.dm_db_index_physical_stats (db_id('AdventureWorks2017'),NULL,NULL,NULL,NULL)
WHERE object_name([object_id])='Product'
```
2. Examine the query results, noting the index types and fragmentation.
