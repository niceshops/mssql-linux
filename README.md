# Implementation of MSSQL on Linux

We want to share an Ansible Role to provision [MSSQL-Server on Linux](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-setup) and some Know-How related to running it.

The Ansible Role focuses on the setup for [Ubuntu Server](https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-ubuntu).

It only supports one instance.

----

## Limitations

Make sure to read into the limitations of MSSQL on Linux beforehand:

* [Release notes](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-release-notes-2022)
* [Editions and Features](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-editions-and-components-2022)
* [Limitations of SSIS](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-ssis-known-issues?view=sql-server-ver16#supported-and-unsupported-maintenance-plan-tasks)

----

## Prerequisites

### OS

Install the [Ubuntu-Server version](https://ubuntu.com/download/alternative-downloads) as specified as supported in the MSSQL documentation linked above.

### Filesystem

Make sure you prepare the target server filesystem as described in the [best-practice guide](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-performance-best-practices) (_RAID, partition format, mounts_).

The role will also check your existing mssql-related mounts for those best-practices.

### Network

You might also want to optimize the [NIC configuration](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-performance-best-practices?#network-setting-recommendations).

### Language

Pull the ['LCID' of for your preferred language](https://learn.microsoft.com/sql/relational-databases/system-catalog-views/sys-fulltext-languages-transact-sql?#values-returned-for-default-languages) to configure it.

----

## SSIS

The Role will install the 'SQL Server Integration Services' as shown in [the documentation](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-setup-ssis?tabs=ubuntu)

SSIS on linux needs to execute its [xml-based .dtsx packages](https://learn.microsoft.com/en-us/sql/integration-services/integration-services-ssis-packages) using the [dtexec binary](https://learn.microsoft.com/en-us/sql/integration-services/packages/dtexec-utility).

### Maintenance plans

We did not (_yet_) manage to successfully get maintenance-plans running using dtexec locally..

As a workaround we use a minimal SQL-Server installation on a Windows-Server to manage and run those scheduled tasks.
Adding maintenance-plan and changing 'server connection' to remote MSSQL host.
See: [connect to MSSQL on Linux using SSMS](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-manage-ssms?view=sql-server-ver16)

----

## Post-Install

### Agent

If you are using the 'SQL-Server Agent' you might want to configure [database mail](https://learn.microsoft.com/en-us/sql/relational-databases/database-mail/configure-sql-server-agent-mail-to-use-database-mail) for it using the SSMS-GUI and add set/provision your 'agent_mail' and 'operator_mail' so it is used by the MSSQL instance! 

### SSIS

If you want to run SSIS jobs/packages you need to:

* Copy the dtsx-packages to the target server
* Test the execution manually using the [dtexec utility](https://learn.microsoft.com/en-us/sql/integration-services/packages/dtexec-utility) (_make sure login is working and mssql-processes are spawned_)
* Schedule the execution via [systemd timer](https://wiki.archlinux.org/title/systemd/Timers) or [cron](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-schedule-ssis-packages)

----

## Config

```yaml
---

# see defaults/main.yml 'mssql_defaults' for all available settings
mssql:
  agent: true  # enable sql-server agent
  fts: true  # install full-text-search
  ssis: true  # install sql-server integration services
  admin_script: true

  # either version or license need to be set!
  version: 'standard'
  # license: 'YYYY-AAAAA-KKK-UUUU-OOOOO'

  path:
    config: '/var/opt/mssql'  # is default
    data: '/var/opt/mssql/data'  # is default
    index: '/var/opt/mssql/index'  # is default
    binlog: '/var/opt/mssql/binlog'  # is default; for transaction-logs
    backup: '/var/backups/mssql'  # is default

  open_file_limit:
    soft: 16000
    hard: 32727

  settings:
    network:
      ip: '192.168.0.1'
      port:
        tcp: 1433
        rpc: 13500
        dtc: 51999

    memory:
      limit_mb: 10240

    language:
      lcid: 1031

    agent_mail: 'local_postfix'  # created via Studio-GUI
    operator_mail: 'Admin'  # created via Studio-GUI
```

----

## Usage

### Service

```bash
# status
systemctl status mssql-server.service

# logs
journalctl -u mssql-server.service -n 100
tail -n 100 /var/log/mssql/sql.error.log
tail -n 100 /var/log/mssql/agent.error.log

# stop
systemctl stop mssql-server.service
# restart
systemctl restart mssql-server.service
```

### Upgrade

```bash
systemctl stop mssql-server.service
apt update
apt upgrade --only-install mssql-*
# reboot or start
systemctl start mssql-server.service
```

### Script

If 'admin_script' is set to true => the role provisions a useful script to simplify administration!

You can enter the interactive MSSQL-shell using this command:

```bash
root@mssql:~# mssql.sh 
>
> ####################
> Connecting to MSSQL-Server..
> ####################
>
> 1> 
```

Or run a single command:

```bash
root@mssql:~# mssql.sh "select login_time,hostname,program_name,cpu,physical_io,status,cmd from master..sysprocesses"
>
> ####################
> Connecting to MSSQL-Server to run query: 'select login_time,hostname,program_name,cpu,physical_io,status,cmd from master..sysprocesses'
> ####################
>
> login_time              hostname                  program_name              cpu         physical_io          status                    cmd                      
> ----------------------- ------------------------- ------------------------- ----------- -------------------- ------------------------- -------------------------
> 2023-04-22 07:14:20.537                                                            5760                    0 background                XIO_RETRY_WORKER
> ......
```