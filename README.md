# PhysicalStandbyWithDGBroker
Create Physical Standby Database and configure Data Guard Broker - 19c

Note: Please modify all necessary configuration files based on your own environment.

This article describes the Oracle Database 19c -Create a Physical Standby Database and Data Guard Broker Setup on Oracle Linux 7 (OL7) 64-bit.

Setup: 
OS: OEL 7.5 
Ansible: ansible 2.7.6
Database Version: Oracle 19.2 Linux64

Oracle DBA - Automation with Ansible (Create a Physical Standby Database and Data Guard Broker Setup)

Summary Steps: 
       1: Create a primary Database using ansible [- - completed before] 
       Please check below link - 
       Create Oracle container database and pluggable database ( https://github.com/asiandevs/Oracle_CDBnPDB_19c ) 
       2: Create a physical standby database
           - Enable force logging
           - Make sure Primary database is in archibvelog mode
           - Enable log archiving
           - Create standby redo log groups at least more than one of the number of logfile groups
           - Add TNS entry for Primary and Physical Standby databases on both sides
           - Create an static listenr entry for broker switchover purpose
       3: Setup Broker Configuration
           - Setup initialization parameters(dg_broker_config_file1 and dg_broker_config_file2)
           - dg_broker_start to TRUE
           - Create configuration for primary database
           - Create configuration for Standby database
           - enable configuration
       4: Validation - Check with SQLPLUS and DGMGRL command line utility     

Ansible commands Summary: 
       1. Clone this repository:
          git clone https://github.com/asiandevs/OracleDBAwithAnsible
       2. Define variables as per your own setup or requirements [ Modify main.yml file under vars directory ]
       3. Configure an Ansible inventory file (example as below) 
          [root@oel75 ansible]# cat ansible.cfg | grep inventory
          inventory = ./inventory
          [root@oel75 ansible]# cat inventory
          [ora-x1]
          192.168.56.102
          [ora-x2]
          192.168.56.103
          [dbservers]
          192.168.56.102
          192.168.56.103
       4. Run the playbook role "create_physical_sb19c.yml"
          ansible-playbook create_physical_sb19c.yml  [ with options for testing, use --check / --diff / --step / -vvv ]
