# CREATE TABLE<a name="r_CREATE_TABLE_NEW"></a>

**Topics**
+ [Syntax](#r_CREATE_TABLE_NEW-synopsis)
+ [Parameters](#r_CREATE_TABLE_NEW-parameters)
+ [Usage notes](r_CREATE_TABLE_usage.md)
+ [Examples](r_CREATE_TABLE_examples.md)

Creates a new table in the current database\. The owner of this table is the issuer of the CREATE TABLE command\.

## Syntax<a name="r_CREATE_TABLE_NEW-synopsis"></a>

```
CREATE [ [LOCAL ] { TEMPORARY | TEMP } ] TABLE 
[ IF NOT EXISTS ] table_name
( { column_name data_type [column_attributes] [ column_constraints ] 
  | table_constraints
  | LIKE parent_table [ { INCLUDING | EXCLUDING } DEFAULTS ] } 
  [, ... ]  )
[ BACKUP { YES | NO } ]
[table_attribute]

where column_attributes are:
  [ DEFAULT default_expr ]
  [ IDENTITY ( seed, step ) ] 
  [ GENERATED BY DEFAULT AS IDENTITY ( seed, step ) ]             
  [ ENCODE encoding ] 
  [ DISTKEY ]
  [ SORTKEY ]
  [ COLLATE CASE_SENSITIVE | COLLATE CASE_INSENSITIVE  ]

and column_constraints are:
  [ { NOT NULL | NULL } ]
  [ { UNIQUE  |  PRIMARY KEY } ]
  [ REFERENCES reftable [ ( refcolumn ) ] ] 

and table_constraints  are:
  [ UNIQUE ( column_name [, ... ] ) ]
  [ PRIMARY KEY ( column_name [, ... ] )  ]
  [ FOREIGN KEY (column_name [, ... ] ) REFERENCES reftable [ ( refcolumn ) ] 


and table_attributes are:
  [ DISTSTYLE { AUTO | EVEN | KEY | ALL } ] 
  [ DISTKEY ( column_name ) ]
  [ [COMPOUND | INTERLEAVED ] SORTKEY ( column_name [,...]) |  [ SORTKEY AUTO ] ]
  [ ENCODE AUTO ]
```

## Parameters<a name="r_CREATE_TABLE_NEW-parameters"></a>

LOCAL   
Optional\. Although this keyword is accepted in the statement, it has no effect in Amazon Redshift\.

TEMPORARY \| TEMP   
Keyword that creates a temporary table that is visible only within the current session\. The table is automatically dropped at the end of the session in which it is created\. The temporary table can have the same name as a permanent table\. The temporary table is created in a separate, session\-specific schema\. \(You can't specify a name for this schema\.\) This temporary schema becomes the first schema in the search path, so the temporary table will take precedence over the permanent table unless you qualify the table name with the schema name to access the permanent table\. For more information about schemas and precedence, see [search\_path](r_search_path.md)\.  
By default, users have permission to create temporary tables by their automatic membership in the PUBLIC group\. To deny this privilege to a user, revoke the TEMP privilege from the PUBLIC group, and then explicitly grant the TEMP privilege only to specific users or groups of users\.

IF NOT EXISTS  
Clause that indicates that if the specified table already exists, the command should make no changes and return a message that the table exists, rather than terminating with an error\. Note that the existing table might be nothing like the one that would have been created; only the table name is used for comparison\.  
This clause is useful when scripting, so the script doesn’t fail if CREATE TABLE tries to create a table that already exists\.

 *table\_name*   
Name of the table to be created\.  
If you specify a table name that begins with '\# ', the table is created as a temporary table\. The following is an example:  

```
create table #newtable (id int);
```
The maximum length for the table name is 127 bytes; longer names are truncated to 127 bytes\. You can use UTF\-8 multibyte characters up to a maximum of four bytes\. Amazon Redshift enforces a quota of the number of tables per cluster by node type, including user\-defined temporary tables and temporary tables created by Amazon Redshift during query processing or system maintenance\. Optionally, the table name can be qualified with the database and schema name\. In the following example, the database name is `tickit`, the schema name is `public`, and the table name is `test`\.   

```
create table tickit.public.test (c1 int);
```
If the database or schema doesn't exist, the table isn't created, and the statement returns an error\. You can't create tables or views in the system databases `template0`, `template1`, and `padb_harvest`\.  
If a schema name is given, the new table is created in that schema \(assuming the creator has access to the schema\)\. The table name must be a unique name for that schema\. If no schema is specified, the table is created by using the current database schema\. If you are creating a temporary table, you can't specify a schema name, because temporary tables exist in a special schema\.  
Multiple temporary tables with the same name can exist at the same time in the same database if they are created in separate sessions because the tables are assigned to different schemas\. For more information about valid names, see [Names and identifiers](r_names.md)\.

 *column\_name*   
Name of a column to be created in the new table\. The maximum length for the column name is 127 bytes; longer names are truncated to 127 bytes\. You can use UTF\-8 multibyte characters up to a maximum of four bytes\. The maximum number of columns you can define in a single table is 1,600\. For more information about valid names, see [Names and identifiers](r_names.md)\.  
If you are creating a "wide table," take care that your list of columns doesn't exceed row\-width boundaries for intermediate results during loads and query processing\. For more information, see [Usage notes](r_CREATE_TABLE_usage.md)\.

 *data\_type*   
Data type of the column being created\. For CHAR and VARCHAR columns, you can use the MAX keyword instead of declaring a maximum length\. MAX sets the maximum length to 4,096 bytes for CHAR or 65535 bytes for VARCHAR\. The maximum size of a GEOMETRY object is 1,048,447 bytes\.  
For information about the data types that Amazon Redshift supports, see [Data types](c_Supported_data_types.md)\.

DEFAULT *default\_expr*   <a name="create-table-default"></a>
Clause that assigns a default data value for the column\. The data type of *default\_expr* must match the data type of the column\. The DEFAULT value must be a variable\-free expression\. Subqueries, cross\-references to other columns in the current table, and user\-defined functions aren't allowed\.  
The *default\_expr* expression is used in any INSERT operation that doesn't specify a value for the column\. If no default value is specified, the default value for the column is null\.  
If a COPY operation with a defined column list omits a column that has a DEFAULT value, the COPY command inserts the value of *default\_expr*\.

IDENTITY\(*seed*, *step*\)   <a name="identity-clause"></a>
Clause that specifies that the column is an IDENTITY column\. An IDENTITY column contains unique autogenerated values\. The data type for an IDENTITY column must be either INT or BIGINT\.   
When you add rows using an `INSERT` or `INSERT INTO [tablename] VALUES()` statement, these values start with the value specified as *seed* and increment by the number specified as *step*\.   
When you load the table using an `INSERT INTO [tablename] SELECT * FROM` or `COPY` statement, the data is loaded in parallel and distributed to the node slices\. To be sure that the identity values are unique, Amazon Redshift skips a number of values when creating the identity values\. Identity values are unique, but the order might not match the order in the source files\. 

GENERATED BY DEFAULT AS IDENTITY\(*seed*, *step*\)   <a name="identity-generated-bydefault-clause"></a>
Clause that specifies that the column is a default IDENTITY column and enables you to automatically assign a unique value to the column\. The data type for an IDENTITY column must be either INT or BIGINT\. When you add rows without values, these values start with the value specified as *seed* and increment by the number specified as *step*\. For information about how values are generated, see [IDENTITY](#identity-clause) \.  
Also, during INSERT, UPDATE, or COPY you can provide a value without EXPLICIT\_IDS\. Amazon Redshift uses that value to insert into the identity column instead of using the system\-generated value\. The value can be a duplicate, a value less than the seed, or a value between step values\. Amazon Redshift doesn't check the uniqueness of values in the column\. Providing a value doesn't affect the next system\-generated value\.  
If you require uniqueness in the column, don't add a duplicate value\. Instead, add a unique value that is less than the seed or between step values\.
Keep in mind the following about default identity columns:   
+ Default identity columns are NOT NULL\. NULL can't be inserted\.
+ To insert a generated value into a default identity column, use the keyword `DEFAULT`\. 

  ```
  INSERT INTO tablename (identity-column-name) VALUES (DEFAULT);
  ```
+ Overriding values of a default identity column doesn't affect the next generated value\. 
+ You can't add a default identity column with the ALTER TABLE ADD COLUMN statement\. 
+ You can append a default identity column with the ALTER TABLE APPEND statement\. 

ENCODE *encoding*   
The compression encoding for a column\. ENCODE AUTO is the default for tables\. Amazon Redshift automatically manages compression encoding for all columns in the table\. If you specify compression encoding for any column in the table, the table is no longer set to ENCODE AUTO\. Amazon Redshift no longer automatically manages compression encoding for all columns in the table\. You can specify the ENCODE AUTO option for the table to enable Amazon Redshift to automatically manage compression encoding for all columns in the table\.  
  
Amazon Redshift automatically assigns an initial compression encoding to columns for which you don't specify compression encoding as follows:  
+ All columns in temporary tables are assigned RAW compression by default\.
+ Columns that are defined as sort keys are assigned RAW compression\.
+ Columns that are defined as BOOLEAN, REAL, DOUBLE PRECISION, GEOMETRY, or GEOGRAPHY data type are assigned RAW compression\.
+ Columns that are defined as SMALLINT, INTEGER, BIGINT, DECIMAL, DATE, TIME, TIMETZ, TIMESTAMP, or TIMESTAMPTZ are assigned AZ64 compression\.
+ Columns that are defined as CHAR, VARCHAR, or VARBYTE are assigned LZO compression\.
If you don't want a column to be compressed, explicitly specify RAW encoding\.
 The following [compression encodings](c_Compression_encodings.md#compression-encoding-list) are supported:  
+ AZ64
+ BYTEDICT
+ DELTA
+ DELTA32K
+ LZO
+ MOSTLY8
+ MOSTLY16
+ MOSTLY32
+ RAW \(no compression\)
+ RUNLENGTH
+ TEXT255
+ TEXT32K
+ ZSTD

DISTKEY  
Keyword that specifies that the column is the distribution key for the table\. Only one column in a table can be the distribution key\. You can use the DISTKEY keyword after a column name or as part of the table definition by using the DISTKEY \(*column\_name*\) syntax\. Either method has the same effect\. For more information, see the DISTSTYLE parameter later in this topic\.  
The data type of a distribution key column can be: BOOLEAN, REAL, DOUBLE PRECISION, SMALLINT, INTEGER, BIGINT, DECIMAL, DATE, TIME, TIMETZ, TIMESTAMP, or TIMESTAMPTZ, CHAR, or VARCHAR\.

SORTKEY  
Keyword that specifies that the column is the sort key for the table\. When data is loaded into the table, the data is sorted by one or more columns that are designated as sort keys\. You can use the SORTKEY keyword after a column name to specify a single\-column sort key, or you can specify one or more columns as sort key columns for the table by using the SORTKEY \(*column\_name* \[, \.\.\.\]\) syntax\. Only compound sort keys are created with this syntax\.  
You can define a maximum of 400 SORTKEY columns per table\.  
The data type of a sort key column can be: BOOLEAN, REAL, DOUBLE PRECISION, SMALLINT, INTEGER, BIGINT, DECIMAL, DATE, TIME, TIMETZ, TIMESTAMP, or TIMESTAMPTZ, CHAR, or VARCHAR\.

COLLATE CASE\_SENSITIVE \| COLLATE CASE\_INSENSITIVE  
A clause that specifies whether string search or comparison on the column is CASE\_SENSITIVE or CASE\_INSENSITIVE\. The default value is the same as the current case sensitivity configuration of the database\.  
To find the database collation information, use the following command:  

```
select db_collation();
  db_collation
----------------
 case_sensitive
(1 row)
```

NOT NULL \| NULL   
NOT NULL specifies that the column isn't allowed to contain null values\. NULL, the default, specifies that the column accepts null values\. IDENTITY columns are declared NOT NULL by default\.

UNIQUE  
Keyword that specifies that the column can contain only unique values\. The behavior of the unique table constraint is the same as that for column constraints, with the additional capability to span multiple columns\. To define a unique table constraint, use the UNIQUE \( *column\_name* \[, \.\.\. \] \) syntax\.  
Unique constraints are informational and aren't enforced by the system\.

PRIMARY KEY  
Keyword that specifies that the column is the primary key for the table\. Only one column can be defined as the primary key by using a column definition\. To define a table constraint with a multiple\-column primary key, use the PRIMARY KEY \( *column\_name* \[, \.\.\. \] \) syntax\.  
Identifying a column as the primary key provides metadata about the design of the schema\. A primary key implies that other tables can rely on this set of columns as a unique identifier for rows\. One primary key can be specified for a table, whether as a column constraint or a table constraint\. The primary key constraint should name a set of columns that is different from other sets of columns named by any unique constraint defined for the same table\.  
Primary key constraints are informational only\. They aren't enforced by the system, but they are used by the planner\.

References *reftable* \[ \( *refcolumn* \) \]  
Clause that specifies a foreign key constraint, which implies that the column must contain only values that match values in the referenced column of some row of the referenced table\. The referenced columns should be the columns of a unique or primary key constraint in the referenced table\.   
 Foreign key constraints are informational only\. They aren't enforced by the system, but they are used by the planner\. 

LIKE *parent\_table* \[ \{ INCLUDING \| EXCLUDING \} DEFAULTS \]   <a name="create-table-like"></a>
A clause that specifies an existing table from which the new table automatically copies column names, data types, and NOT NULL constraints\. The new table and the parent table are decoupled, and any changes made to the parent table aren't applied to the new table\. Default expressions for the copied column definitions are copied only if INCLUDING DEFAULTS is specified\. The default behavior is to exclude default expressions, so that all columns of the new table have null defaults\.   
Tables created with the LIKE option don't inherit primary and foreign key constraints\. Distribution style, sort keys, BACKUP, and NULL properties are inherited by LIKE tables, but you can't explicitly set them in the CREATE TABLE \.\.\. LIKE statement\.

BACKUP \{ YES \| NO \}   <a name="create-table-backup"></a>
A clause that specifies whether the table should be included in automated and manual cluster snapshots\. For tables, such as staging tables, that don't contain critical data, specify BACKUP NO to save processing time when creating snapshots and restoring from snapshots and to reduce storage space on Amazon Simple Storage Service\. The BACKUP NO setting has no affect on automatic replication of data to other nodes within the cluster, so tables with BACKUP NO specified are restored in a node failure\. The default is BACKUP YES\.

DISTSTYLE \{ AUTO \| EVEN \| KEY \| ALL \}  
Keyword that defines the data distribution style for the whole table\. Amazon Redshift distributes the rows of a table to the compute nodes according to the distribution style specified for the table\. The default is AUTO\.  
The distribution style that you select for tables affects the overall performance of your database\. For more information, see [Working with data distribution styles](t_Distributing_data.md)\. Possible distribution styles are as follows:  
+ AUTO: Amazon Redshift assigns an optimal distribution style based on the table data\. For example, if AUTO distribution style is specified, Amazon Redshift initially assigns ALL distribution style to a small table, then changes the table to EVEN distribution when the table grows larger\. If Amazon Redshift determines that a distribution key will improve the performance of queries, then Amazon Redshift might change the DISTSTYLE to KEY and assign a distribution key to your table\. The change in distribution style occurs in the background with minimal impact to user queries\. 

  To view the distribution style applied to a table, query the PG\_CLASS system catalog table\. For more information, see [Viewing distribution styles](viewing-distribution-styles.md)\. 
+ EVEN: The data in the table is spread evenly across the nodes in a cluster in a round\-robin distribution\. Row IDs are used to determine the distribution, and roughly the same number of rows are distributed to each node\. 
+ KEY: The data is distributed by the values in the DISTKEY column\. When you set the joining columns of joining tables as distribution keys, the joining rows from both tables are collocated on the compute nodes\. When data is collocated, the optimizer can perform joins more efficiently\. If you specify DISTSTYLE KEY, you must name a DISTKEY column, either for the table or as part of the column definition\. For more information, see the DISTKEY parameter earlier in this topic\.
+  ALL: A copy of the entire table is distributed to every node\. This distribution style ensures that all the rows required for any join are available on every node, but it multiplies storage requirements and increases the load and maintenance times for the table\. ALL distribution can improve execution time when used with certain dimension tables where KEY distribution isn't appropriate, but performance improvements must be weighed against maintenance costs\. 

DISTKEY \( *column\_name* \)  
Constraint that specifies the column to be used as the distribution key for the table\. You can use the DISTKEY keyword after a column name or as part of the table definition, by using the DISTKEY \(*column\_name*\) syntax\. Either method has the same effect\. For more information, see the DISTSTYLE parameter earlier in this topic\.

\[COMPOUND \| INTERLEAVED \] SORTKEY \(* column\_name* \[,\.\.\.\]\) \| \[ SORTKEY AUTO \]  
Specifies one or more sort keys for the table\. When data is loaded into the table, the data is sorted by the columns that are designated as sort keys\. You can use the SORTKEY keyword after a column name to specify a single\-column sort key, or you can specify one or more columns as sort key columns for the table by using the `SORTKEY (column_name [ , ... ] )` syntax\.   
You can optionally specify COMPOUND or INTERLEAVED sort style\. If you specify SORTKEY with columns the default is COMPOUND\. For more information, see [Working with sort keys](t_Sorting_data.md)\.  
If you don't specify any sort keys options, the default is AUTO\.  
You can define a maximum of 400 COMPOUND SORTKEY columns or 8 INTERLEAVED SORTKEY columns per table\.     
AUTO  
Specifies that Amazon Redshift assigns an optimal sort key based on the table data\. For example, if AUTO sort key is specified, Amazon Redshift initially assigns no sort key to a table\. If Amazon Redshift determines that a sort key will improve the performance of queries, then Amazon Redshift might change the sort key of your table\. The actual sorting of the table is done by automatic table sort\. For more information, see [Automatic table sort](t_Reclaiming_storage_space202.md#automatic-table-sort)\.   
Amazon Redshift doesn't modify tables that have existing sort or distribution keys\. With one exception, if a table has a distribution key that has never been used in a JOIN, then the key might be changed if Amazon Redshift determines there is a better key\.   
To view the sort key of a table, query the SVV\_TABLE\_INFO system catalog view\. For more information, see [SVV\_TABLE\_INFO](r_SVV_TABLE_INFO.md)\. To view the Amazon Redshift Advisor recommendations for tables, query the SVV\_ALTER\_TABLE\_RECOMMENDATIONS system catalog view\. For more information, see [SVV\_ALTER\_TABLE\_RECOMMENDATIONS](r_SVV_ALTER_TABLE_RECOMMENDATIONS.md)\. To view the actions taken by Amazon Redshift, query the SVL\_AUTO\_WORKER\_ACTION system catalog view\. For more information, see [SVL\_AUTO\_WORKER\_ACTION](r_SVL_AUTO_WORKER_ACTION.md)\.   
COMPOUND  
Specifies that the data is sorted using a compound key made up of all of the listed columns, in the order they are listed\. A compound sort key is most useful when a query scans rows according to the order of the sort columns\. The performance benefits of sorting with a compound key decrease when queries rely on secondary sort columns\. You can define a maximum of 400 COMPOUND SORTKEY columns per table\.   
INTERLEAVED  
Specifies that the data is sorted using an interleaved sort key\. A maximum of eight columns can be specified for an interleaved sort key\.   
An interleaved sort gives equal weight to each column, or subset of columns, in the sort key, so queries don't depend on the order of the columns in the sort key\. When a query uses one or more secondary sort columns, interleaved sorting significantly improves query performance\. Interleaved sorting carries a small overhead cost for data loading and vacuuming operations\.   
Don’t use an interleaved sort key on columns with monotonically increasing attributes, such as identity columns, dates, or timestamps\.

ENCODE AUTO   
Enables Amazon Redshift to automatically adjust the encoding type for all columns in the table to optimize query performance\. ENCODE AUTO preserves the initial encode types that you specify in creating the table\. Then, if Amazon Redshift determines that a new encoding type can improve query performance, Amazon Redshift can change the encoding type of the table columns\. ENCODE AUTO is the default if you don't specify an encoding type on any column in the table\.

UNIQUE \( *column\_name* \[,\.\.\.\] \)  
Constraint that specifies that a group of one or more columns of a table can contain only unique values\. The behavior of the unique table constraint is the same as that for column constraints, with the additional capability to span multiple columns\. In the context of unique constraints, null values aren't considered equal\. Each unique table constraint must name a set of columns that is different from the set of columns named by any other unique or primary key constraint defined for the table\.   
 Unique constraints are informational and aren't enforced by the system\. 

PRIMARY KEY \( *column\_name* \[,\.\.\.\] \)  
Constraint that specifies that a column or a number of columns of a table can contain only unique \(nonduplicate\) non\-null values\. Identifying a set of columns as the primary key also provides metadata about the design of the schema\. A primary key implies that other tables can rely on this set of columns as a unique identifier for rows\. One primary key can be specified for a table, whether as a single column constraint or a table constraint\. The primary key constraint should name a set of columns that is different from other sets of columns named by any unique constraint defined for the same table\.   
 Primary key constraints are informational only\. They aren't enforced by the system, but they are used by the planner\. 

FOREIGN KEY \( *column\_name* \[, \.\.\. \] \) REFERENCES *reftable* \[ \( *refcolumn* \) \]   
Constraint that specifies a foreign key constraint, which requires that a group of one or more columns of the new table must only contain values that match values in the referenced column or columns of some row of the referenced table\. If *refcolumn* is omitted, the primary key of *reftable* is used\. The referenced columns must be the columns of a unique or primary key constraint in the referenced table\.  
Foreign key constraints are informational only\. They aren't enforced by the system, but they are used by the planner\.