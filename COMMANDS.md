# Useful TSQL commands

## Show databases

```
SELECT name, database_id, create_date FROM sys.databases
```


## Show processlist

```
select * from master..sysprocesses

# interesting infos
select login_time,hostname,program_name,loginame,cpu,physical_io,status,cmd from master..sysprocesses

# filter by program
select login_time,hostname,program_name,cpu,physical_io,status,cmd from master..sysprocesses WHERE program_name LIKE '%PROG%'
```

## Restore database

```
# show files that need to be restored
RESTORE FILELISTONLY FROM DISK = '/var/backups/mssql/app1.bak'
GO

# drop db before restoring it
drop database APP1
GO

RESTORE DATABASE APP1 FROM DISK = '/var/backups/mssql/app1.bak'

# if paths should be changed:
WITH MOVE 'APP1' TO '/var/opt/mssql/data/app1.mdf', MOVE 'APP1_IND' TO '/var/opt/mssql/index/app1.idx',
MOVE 'APP1_LOG' TO '/var/opt/mssql/binlog/app1_1.ldf', MOVE 'APP1_LOG2' TO '/var/opt/mssql/binlog/app1_2.ldf'
# endif

GO
```

## Show agent jobs

```
select subsystem, step_name, database_name, last_run_outcome, last_run_duration, last_run_date, last_run_time from msdb.dbo.sysjobsteps
```

## Show existing logins

```
SELECT accdate, loginname, sysadmin FROM sys.syslogins
```
