## PhysicalStandbyWithDGBroker
Create Physical Standby Database and configure Data Guard Broker - 19c

> Note: Please modify all necessary configuration files based on your own environment.

> This article describes the Oracle Database 19c -Create a Physical Standby Database and Data Guard Broker Setup on Oracle Linux 7 (OL7) 64-bit.

**Setup:**
 * OS: OEL 7.5 
 * Ansible: ansible 2.7.6
 * Database Version: Oracle 19.3 Linux64

## Oracle DBA - Automation with Ansible (Create a Physical Standby Database and Data Guard Broker Setup)

**Summary Steps:** 
### 1: Create a primary Database using ansible [- - completed before] 
 * Please check below link - 
 * [Create Oracle container database and pluggable database](https://https://github.com/asiandevs/Oracle_CDBnPDB_19c)  
 
### 2: Create a physical standby database
 * Enable force logging
 * Make sure Primary database is in archibvelog mode
 * Enable log archiving
 * Create standby redo log groups at least more than one of the number of logfile groups
 * Add TNS entry for Primary and Physical Standby databases on both sides
 * Create an static listenr entry for broker switchover purpose
       
### 3: Setup Broker Configuration
* Setup initialization parameters(dg_broker_config_file1 and dg_broker_config_file2)
* dg_broker_start to TRUE
* Create configuration for primary database
* Create configuration for Standby database
* enable configuration

### Validation - Check with SQLPLUS and DGMGRL command line utility     

## Ansible commands Summary: 
1. Clone this repository:
 * git clone https://github.com/asiandevs/OracleDBAwithAnsible

2. Define variables as per your own setup or requirements:
 * [ Modify main.yml file under vars directory ]
 
3. Configure an Ansible inventory file (example as below) 
 * git clone https://github.com/asiandevs/OracleDBAwithAnsible
- Configure an Ansible inventory file (example as below) 
```javascript
cat ansible.cfg | grep inventory
          inventory = ./inventory
          [root@oel75 ansible]# cat inventory
          [ora-x1]
          192.168.56.102
          [ora-x2]
          192.168.56.103
          [dbservers]
          192.168.56.102
          192.168.56.103
```
4. Run the playbook role "create_physical_sb19c.yml" 
 * ansible-playbook create_physical_sb19c.yml  [ with options for testing, use --check / --diff / --step / -vvv ]

     ## *Example Output* 

```javascript
[root@oel75 ansible]# ansible-playbook PhysicalSB_Broker19c.yml

PLAY [ora-x1,ora-x2] ************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [192.168.56.102]
ok: [192.168.56.103]

TASK [sbdb19c_create : check if any Data Guard Setup already exist] *************************************************************
changed: [192.168.56.102]

TASK [sbdb19c_create : failed message details] **********************************************************************************

TASK [sbdb19c_create : Get DG details] ******************************************************************************************
changed: [192.168.56.102]

TASK [sbdb19c_create : Create LogFile  Directory] *******************************************************************************
ok: [192.168.56.102]
ok: [192.168.56.103]

TASK [sbdb19c_create : Create_standbyDB | create required directories on standby database server] *******************************
ok: [192.168.56.103] => (item=/u01/stage)
ok: [192.168.56.103] => (item=/u01/stage/logs)
ok: [192.168.56.103] => (item=/u01/app/oracle/oradata/pmjks)
ok: [192.168.56.103] => (item=/u01/app/oracle/redo/pmjks)
ok: [192.168.56.103] => (item=/u01/app/oracle/redo/pmjks/pdbseed)

TASK [sbdb19c_create : Create_standbyDB | create required directories on Primary database server] *******************************
ok: [192.168.56.102] => (item=/u01/stage)
ok: [192.168.56.102] => (item=/u01/stage/logs)
ok: [192.168.56.102] => (item=/u01/app/oracle/redo/eaymp)

TASK [sbdb19c_create : Create_standbyDB | Copy required script to Primary database server] **************************************
ok: [192.168.56.102] => (item=chkexistdg.sql)
ok: [192.168.56.102] => (item=gensbredologs.sql)
changed: [192.168.56.102] => (item=postcrpr.sql)
ok: [192.168.56.102] => (item=dg_status.sql)
changed: [192.168.56.102] => (item=tns_upd.sh)
ok: [192.168.56.102] => (item=add_staticlsnr.sh)

TASK [sbdb19c_create : Create_standbyDB | Copy database SQL script to standby database server] **********************************
ok: [192.168.56.103] => (item=postcrsb.sql)
ok: [192.168.56.103] => (item=tns_upd.sh)
ok: [192.168.56.103] => (item=dg_status.sql)
ok: [192.168.56.103] => (item=add_staticlsnr.sh)

TASK [sbdb19c_create : Create_standbyDB | template for create dg_role modify script] ********************************************
ok: [192.168.56.102]
ok: [192.168.56.103]

TASK [sbdb19c_create : Create_standbyDB | template for postsql on standby database] *********************************************
changed: [192.168.56.103]

TASK [sbdb19c_create : Create_standbyDB | template for postsql on primary database] *********************************************
changed: [192.168.56.102]

TASK [sbdb19c_create : Create_standbyDB | template for physical standby using dbca] *********************************************
ok: [192.168.56.103]

TASK [sbdb19c_create : Create_standbyDB | create silent listener config file on SB Host] ****************************************
ok: [192.168.56.103]

TASK [sbdb19c_create : Create_standbyDB | Configuring listener on SB Host] ******************************************************
changed: [192.168.56.103]

TASK [sbdb19c_create : Create_standbyDB | execute tns update for primary database] **********************************************
changed: [192.168.56.102]
changed: [192.168.56.103]

TASK [sbdb19c_create : Create_standbyDB | execute tns update for standby database] **********************************************
changed: [192.168.56.102]
changed: [192.168.56.103]

TASK [sbdb19c_create : Create_standbyDB | add static listener] ******************************************************************
changed: [192.168.56.102]

TASK [sbdb19c_create : Create_standbyDB | add static listener] ******************************************************************
changed: [192.168.56.103]

TASK [sbdb19c_create : Create_standbyDB | reload listener on primary host] ******************************************************
changed: [192.168.56.102]
changed: [192.168.56.103]

TASK [sbdb19c_create : Create_standbyDB | Setup force logging Parameter] ********************************************************
changed: [192.168.56.102] => (item=alter database force logging)

TASK [sbdb19c_create : Create_standbyDB | Ensure archiving is set to Fast Recovery Area on the primary] *************************
changed: [192.168.56.102] => (item=alter system set log_archive_dest_1='location=USE_DB_RECOVERY_FILE_DEST valid_for=(ALL_LOGFILES,ALL_ROLES) db_unique_name=eaymp')

TASK [sbdb19c_create : Create_standbyDB | execute standby redologs script] ******************************************************
changed: [192.168.56.102]

TASK [sbdb19c_create : display prep configuration message] **********************************************************************
ok: [192.168.56.102] => {
    "msg": [
        "This Steps completed below task for Single Instance at 2019-05-01T12:33:21Z:",
        "- Create a Physical Standby Database pre configuration task completed",
        "-  Next task is to create Physical Standby database"
    ]
}

TASK [sbdb19c_create : Create_standbyDB | Create a Physical Standby Database] ***************************************************
changed: [192.168.56.103]

TASK [sbdb19c_create : Get Primary Database Information] ************************************************************************
changed: [192.168.56.102] => (item=SELECT DATABASE_ROLE, OPEN_MODE, DATAGUARD_BROKER FROM v\$database)

TASK [sbdb19c_create : display details of primary databse] **********************************************************************
ok: [192.168.56.102] => (item=
SQL*Plus: Release 19.0.0.0.0 - Production on Wed May 1 22:37:32 2019
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0

SQL>
DATABASE_ROLE    OPEN_MODE            DATAGUAR
---------------- -------------------- --------
PRIMARY          READ WRITE           DISABLED

SQL> Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0) => {
    "msg": "\nSQL*Plus: Release 19.0.0.0.0 - Production on Wed May 1 22:37:32 2019\nVersion 19.3.0.0.0\n\nCopyright (c) 1982, 2019, Oracle.  All rights reserved.\n\n\nConnected to:\nOracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production\nVersion 19.3.0.0.0\n\nSQL> \nDATABASE_ROLE\t OPEN_MODE\t      DATAGUAR\n---------------- -------------------- --------\nPRIMARY \t READ WRITE\t      DISABLED\n\nSQL> Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production\nVersion 19.3.0.0.0"
}
ok: [192.168.56.103] => (item=[Undefined]) => {
    "msg": "[Undefined]"
}

TASK [sbdb19c_create : Get Standby Database Information] ************************************************************************
changed: [192.168.56.103] => (item=SELECT DATABASE_ROLE, OPEN_MODE, DATAGUARD_BROKER FROM v\$database)

TASK [sbdb19c_create : display details of standby database] *********************************************************************
ok: [192.168.56.102] => (item=[Undefined]) => {
    "msg": "[Undefined]"
}
ok: [192.168.56.103] => (item=
SQL*Plus: Release 19.0.0.0.0 - Production on Wed May 1 22:37:35 2019
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0

SQL>
DATABASE_ROLE    OPEN_MODE            DATAGUAR
---------------- -------------------- --------
PHYSICAL STANDBY READ ONLY            DISABLED

SQL> Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0) => {
    "msg": "\nSQL*Plus: Release 19.0.0.0.0 - Production on Wed May 1 22:37:35 2019\nVersion 19.3.0.0.0\n\nCopyright (c) 1982, 2019, Oracle.  All rights reserved.\n\n\nConnected to:\nOracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production\nVersion 19.3.0.0.0\n\nSQL> \nDATABASE_ROLE\t OPEN_MODE\t      DATAGUAR\n---------------- -------------------- --------\nPHYSICAL STANDBY READ ONLY\t      DISABLED\n\nSQL> Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production\nVersion 19.3.0.0.0"
}

TASK [sbdb19c_create : Create_standbyDB | execute postsql from the standby database] ********************************************
changed: [192.168.56.103]

TASK [sbdb19c_create : Create_standbyDB | execute broker Setup from the Primary Database] ***************************************
changed: [192.168.56.102]

TASK [sbdb19c_create : display post install message] ****************************************************************************
ok: [192.168.56.102] => {
    "msg": [
        "This Steps completed below task for Single Instance at 2019-05-01T12:33:21Z:",
        "- Data Guard Broker setup is completed",
        "- Next we will validate Data Guard broker setup"
    ]
}

TASK [sbdb19c_create : Execute dgmgrl show configuration command] ***************************************************************
changed: [192.168.56.102]

TASK [sbdb19c_create : set_fact] ************************************************************************************************
ok: [192.168.56.102]

TASK [sbdb19c_create : set_fact] ************************************************************************************************
ok: [192.168.56.102]

TASK [sbdb19c_create : Execute dgmgrl show configuration command] ***************************************************************
changed: [192.168.56.102]

TASK [sbdb19c_create : Execute dgmgrl show configuration command] ***************************************************************
changed: [192.168.56.102]

TASK [sbdb19c_create : debug] ***************************************************************************************************
ok: [192.168.56.102] => {
    "msg": [
        "Primary database – eaymp, Standby database – pmjks ",
        [
            "ApplyLag:0seconds",
            "TransportLag:0seconds"
        ]
    ]
}

TASK [sbdb19c_create : display post install message after creating of Physical Standby] *****************************************
ok: [192.168.56.102] => {
    "msg": [
        "Following tasks are completed for Single Instance at 2019-05-01T12:33:21Z:",
        "- Create a Physical Standby Database is completed on server ora-x2",
        "- Setup Data Guard Broker configuration",
        "- END OF ALL: git repository \"PhysicalStandbyWithDGBroker\" will be shared"
    ]
}

PLAY RECAP **********************************************************************************************************************
192.168.56.102             : ok=28   changed=16   unreachable=0    failed=0
192.168.56.103             : ok=18   changed=9    unreachable=0    failed=0


```
       ## *For an existing Data Guard setup*      


```javascript

[root@oel75 ansible]# ansible-playbook PhysicalSB_Broker19c.yml

PLAY [ora-x1,ora-x2] ************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [192.168.56.103]
ok: [192.168.56.102]

TASK [sbdb19c_create : check if any Data Guard Setup already exist] *************************************************************
changed: [192.168.56.102]

TASK [sbdb19c_create : failed message details] **********************************************************************************
fatal: [192.168.56.102]: FAILED! => {"changed": false, "msg": "Dataguarad Setup already exist for this database - 'pmjks'. Define a different target database"}

NO MORE HOSTS LEFT **************************************************************************************************************
        to retry, use: --limit @/etc/ansible/PhysicalSB_Broker19c.retry

PLAY RECAP **********************************************************************************************************************
192.168.56.102             : ok=2    changed=1    unreachable=0    failed=1
192.168.56.103             : ok=1    changed=0    unreachable=0    failed=0

```

> I hope you can play with Ansible for doing different daily activities.
> Thanks for checking this out. Any advise welcome !! :smile: :heart:
