## Shrinking a file -- Best Practices

#### Consider the following information when you plan to shrink a file:

  * A shrink operation is most effective after an operation that creates lots of unused space, such as a truncate table or a drop table operation.
  * Most databases require some free space to be available for regular day-to-day operations. If you shrink a database repeatedly and notice that the database size grows again, this indicates that the space that was shrunk is required for regular operations. In these cases, repeatedly shrinking the database is a wasted operation.
  * A shrink operation does not preserve the fragmentation state of indexes in the database, and generally increases fragmentation to a degree. This is another reason not to repeatedly shrink the database.
  * Shrink multiple files in the same database sequentially instead of concurrently. Contention on system tables can cause delays due to blocking.
