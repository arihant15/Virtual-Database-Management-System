
Project Modules
--------------------------------------------------------------------------------------
C source files:  buffer_mgr.c,buffer_mgr_stat.c,dberror.c,expr.c,record_mgr.c,rm_serializer.c,storage_mgr.c,test_assign3_1.c,test_expr.c, test_assign3_2.c
Header files:    buffer_mgr.h,buffer_mgr_stat.h,dberror.h,dt.h,expr.h,mgmt.h,record_mgr.h,storage_mgr.h,tables.h,test_helper.h


Aim:
----------------------------------------------------------------------------------------
The goal of this assignment is to implement a record manager - a module that 
handles tables with fixed schema.Each table is stored as a separate page file. Record manager access the table using the buffer manager.Various functions written in record_mgr.c will provide the facility insert records, delete records,update records and scan through all records in the table. Scanning of record depends on the search condition and records that matched with this condition will be returned.

Instructions to run the code
----------------------------
1) Run the below command:
   make
2) Run the below command for testing test_assign3_1.c:
   ./test_assign3_1
3) Run the below command for testing test_expr.c:
   ./test_expr
4) Run the below command for testing extra test cases for extra functionality mentioned in test_assign3_2.c:
   ./test_assign3_2
5) To remove object files and executables, run following command:
   make clean  

Extensions: 
--------------
Our implementation consists of the following extensions:

1) TIDs and Tombstone :
This functionality is used in insert , update and delete record.
We are having a flag set  as "deleted" for soft delete.
When user deletes a record, flag is set.

2) Check Primary key constraint: 
We are keeping track of primary keys into a hashmap using structure implementation.
Before inserting a record, primary key will be compared to the saved keys.
If there exists key already in the map, error will be thrown.

3) Extra test case: 
Added extra test cases in file test_assign3_2.c to test tombstone implementation and Primary key constraint implementation.

Additional Error codes
--------------------
We have defined some new error codes and corresponding error 
messages in  dberror.h file which are mentioned below:

RC_NULLTABLE 17 
RC_NULLCONDITION 18 
RC_RM_CREATE_TABLE_ERROR 19 
RC_RM_RECORD_NOT_FOUND_ERROR 20
RC_RM_TABLE_DATA_NOT_INIT 21 

Modification of mgmt.h
---------------------------------------------------
Added following structures for implementation of record manager, tombstone strategy and primary key constraints.
1. Record_Key : Hash Map implementation for Primary key checking functionality
2. RM_RecordMgmt : Structure for all the records to be created
3. RM_ScanMgmt : Structure for scan management

Functions:
---------------------------------------------------
There are five types of functions in the record manager: 
1)functions for table and record manager management
2)functions for handling the records in a table, 
3)functions related to scans, 
4)functions for dealing with schemas 
5)function for dealing with attribute values and creating records.

All these functions are described below:

initRecordManager()
------------------------------
Initializes the program. Returns RC_OK.

recordMemoryRequired()
---------------------------
This functions returns bytes of memory required for a particular schema ,based upon number of attributes and data type present in schema.

createTable()
--------------------------
Creates a table. Each table is a page file. Therefore this function will create and open the page file. If page file is successfully created and opened ,serialize  the schema, write the data to file handler. Finally free 

openTable()
--------------------------
This function opens the table. Initializes the record management structure and the buffer pool . On successful initialization, deserializes the schema and returns error/response code accordingly.

closeTable()
-------------------------
This function shuts down the buffer pool, marks the allocated memory to null, frees the memory. It closes the table and returns RC_OK.

deleteTable()
--------------------------
This function deletes an open table/ page file by using destroyPageFile function.

serSchema()
---------------
This functions takes the object and  converts the object data type to the string type . This string type data is then written to the file. 
Based on the memory required for schema ,allocates the memory.Then sends the formatted data to a structure variable. Thus this function serializes the schema using attributes, data type and typeLength present in schema.

deserSchema()
---------------
This function de-serializes the schema from the page.
Reads the contents from the file. According to the data type it de-serializes into schema objects.

getNumTuples()
-------------------
This function counts the number of tuples in the given table. Initializes the count to 0,starts from 1st page and slot 0 of the table, moves thorough all pages of table. If getRecord() returns RC_OK, increment number of tuples counter. Once the table's end is reached,free the memory and returns the final count of tuples.


insertRecord()
----------------
This function inserts the record in table if user enters primary key which is not present in the given table. Scans the table, obtains a record and checks for soft delete. If the tombstone is found, frees the memory. Then sets the page number for insertion of record.Pins the page and starts putting record in the given page.Marks the page dirty, since there will be new data in table, flushes the data into page.
Primary key constraints are always fulfilled.

deleteRecord()
--------------------
This function (soft)deletes the record and creates a tombstone by marking space as "deleted:".It scans through the page file,pins the required page, sets a flag of tombstone, marks the page dirty , and then unpins the page. Finally using 'forcePage' flushes the memory page. Else if record is not found in the table returns error code accordingly.
Primary key constraints are  always fulfilled.

updateRecord()
----------------------
This function takes the given data, pull the existing record and update it with the given data based on primary key constraint fulfilment. 

getRecord ()
------------------------
This functions scans through the table. If required record is found pins the page, stores the data and id of record and then unpins the page. On successful completion of operation return RC_OK. Else if record is not found, 'Record not found error' code.

startScan ()
----------------------
This function initializes all the scan data , puts it in table so that tuples that satisfy certain condition can be retrived later.

next()
--------------------
This function returns the tuple that satisfy certain condition.

closeScan()
-------------------
This function closes the scan operation, makes scan data null and frees the corresponding memory.

getRecordSize()
-------------
This returns the size required by a record.

createSchema()
------------------------
This function creates schema and initilaize the schema attributes to given value.

freeSchema()
----------------------
This function will free all the schema attributes. Returns RC_OK on successful completion.

createRecord()
--------------------
This function creates records and adds it to the record list. First it gets the memory required for the record and allocates that memory.Then it creates record and adds the data to the record list and returns RC accordingly
  
freeRecord()
--------------------
This function frees the memory allocated to records.

getAttr()
--------------------
This function gets attributes of the record.

setAttr()
--------------------
This function sets attribute of values as integer, float, boolean or as a string.

shutdownRecordManager()
------------------------
Returns RC_OK.


