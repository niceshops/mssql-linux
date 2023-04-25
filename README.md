# Implementation of MSSQL on Linux

We want to share an Ansible Role to provision [MSSQL-Server on Linux](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-setup) and some Know-How related to running it.

The Ansible Role focuses on the setup for [Ubuntu Server](https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-ubuntu).


## Limitations

Make sure to read into the limitations of MSSQL on Linux beforehand:

* [Release notes](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-release-notes-2022)
* [Editions and Features](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-editions-and-components-2022)
* [Limitations of SSIS](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-ssis-known-issues?view=sql-server-ver16#supported-and-unsupported-maintenance-plan-tasks)

## SSIS

The Role will install the 'SQL Server Integration Services' as shown in [the documentation](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-setup-ssis?tabs=ubuntu)

SSIS on linux needs to execute its [xml-based .dtsx packages](https://learn.microsoft.com/en-us/sql/integration-services/integration-services-ssis-packages) using the [dtexec binary](https://learn.microsoft.com/en-us/sql/integration-services/packages/dtexec-utility).

### Maintenance plans

We did not (_yet_) manage to successfully get maintenance-plans running using dtexec locally..

As a workaround we use a minimal SQL-Server installation on a Windows-Server to manage and run those scheduled tasks. See: [connect to MSSQL on Linux using SSMS](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-manage-ssms?view=sql-server-ver16)
