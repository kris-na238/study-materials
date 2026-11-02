## Notes
    - When a table schema is given, and you need to fetch records, look for active column as well.
    - Query to create indexing in mysql
        create index index_name on table_name(column1, column2 desc/asc,.. )
    - LeftMost-prefix rule in mysql says mysql can efficiently use a composite index starting from leftmost colum and it can continue using subsequent columns only when the preceding indexed columns are constrained appropriately.
    - downside of Indexing is write heavy, and consume storages, so everytime table is upadated indexed data also will be updated.
    - Offset is used to skip rows in mysql ex:
        select salary from employess where status = 'active' order by salary desc limit 1 offset 1.
    -
