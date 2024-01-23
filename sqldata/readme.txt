Placeholder file to keep folder in source control.
For development purposes, the /sqldata file is mapped to the container so you can easily copy files to/from the container for ease of restore.

Here's the script to make life even easier:


USE [master]
GO

-- Step 1: Run this code block to determine the location of the data file and the log file
SELECT 
  DB_NAME([database_id]) [database_name]
, [file_id]
, [type_desc] [file_type]
, [name] [logical_name]
, [physical_name]
FROM sys.[master_files]
WHERE [database_id] IN (DB_ID('MyDb'))
ORDER BY [type], DB_NAME([database_id]);

-- Step 2: Check the filelist of the .bak file (optional)

RESTORE FILELISTONLY FROM DISK = '/var/opt/mssql/data/MyDb.bak' WITH FILE = 1

-- Step 3: Perform the restore to the locations determined in Step 1.
ALTER DATABASE [MyDb]
SET SINGLE_USER
WITH ROLLBACK AFTER 30

RESTORE DATABASE [MyDb] FROM DISK='/var/opt/mssql/MyDb.bak'
WITH REPLACE,
MOVE 'MyDb' TO '/var/opt/mssql/data/MyDb.mdf',
MOVE 'MyDb_Log' TO '/var/opt/mssql/log/MyDb_log.ldf'

ALTER DATABASE [MyDb] SET MULTI_USER
GO
