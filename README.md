How-to-install-Oracle-SOA-Suite-12C-PS3-12.2.1.3.0-Docker-Containers

First Install Docker following the steps in "Install Docker, Docker-Compose and Helm on Oracle Linux 8" repository:  

After that; Let’s start to install

Login to Oracle Container Registry

# docker login container-registry.oracle.com 

Username: c***.p******@oracle.com
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

Download & Pull Oracle 19c DB Image

# docker pull container-registry.oracle.com/database/enterprise:19.3.0.0
19.3.0.0: Pulling from database/enterprise
66fb34780033: Pull complete
09fdde9a2705: Pull complete
8eef51bdc524: Pull complete
ef20be24fdb7: Pull complete
3eec5dd39cb8: Pull complete
f54e46803ca1: Pull complete
bbf9b5dc6e4f: Pull complete
acce8c5c4fc6: Pull complete
4eed3ae7ef08: Pull complete
ea799524d345: Pull complete
dc921b3204ee: Pull complete
ff21863176fa: Pull complete
a3f0ecf1aafc: Pull complete
58fb15665386: Pull complete
630de49b8518: Pull complete
94e9e3c3db5e: Pull complete
c62e2b0d3050: Pull complete
Digest: sha256:111e8b3c943bee7017cfd3465bd604aaa2966782b60dd2bff9abe7201a5808cc
Status: Downloaded newer image for container-registry.oracle.com/database/enterprise:19.3.0.0
container-registry.oracle.com/database/enterprise:19.3.0.0

You can tag the name you prefer;

# docker tag container-registry.oracle.com/database/enterprise:19.3.0.0 localhost/oracle/database:19.3.0.0-ee


Download & Pull SOA Image

# docker pull container-registry.oracle.com/middleware/soasuite:12.2.1.3

12.2.1.3: Pulling from middleware/soasuite
35defbf6c365: Pull complete
e406c26b7097: Pull complete
51d87468764e: Pull complete
8bcb1774cc5f: Pull complete
42439840a3ce: Pull complete
ebb1c8ed0ed5: Pull complete
472d1f30da2f: Pull complete
01959fe14815: Pull complete
09c7abad440a: Pull complete
c3d45d18a5a1: Pull complete
Digest: sha256:ac56e1d77d1c972f5713a3810101fe5c500ee42a2b7b6187ecac65ee250feef6
Status: Downloaded newer image for container-registry.oracle.com/middleware/soasuite:12.2.1.3
container-registry.oracle.com/middleware/soasuite:12.2.1.3

You can tag the name you prefer;

# docker tag container-registry.oracle.com/middleware/soasuite:12.2.1.3 localhost/oracle/soasuite:12.2.1.3

Download & Edit SOA Suite Docker files from the FMW Repository

# mkdir /u01
# cd /u01
# git clone https://github.com/oracle/docker-images
Cloning into 'docker-images'...
remote: Enumerating objects: 16220, done.
remote: Counting objects: 100% (137/137), done.
remote: Compressing objects: 100% (106/106), done.
remote: Total 16220 (delta 39), reused 83 (delta 26), pack-reused 16083
Receiving objects: 100% (16220/16220), 10.53 MiB | 7.68 MiB/s, done.
Resolving deltas: 100% (9549/9549), done.


# cd docker-images/OracleSOASuite/
# ll
total 12
-rw-r--r--. 1 root root 1095 Nov 28 13:44 COPYRIGHT
-rw-r--r--. 1 root root 1325 Nov 28 13:44 README.md
drwxr-xr-x. 5 root root   98 Nov 28 13:44 dockerfiles
drwxr-xr-x. 2 root root   49 Nov 28 13:44 samples
-rw-r--r--. 1 root root 1909 Nov 28 13:44 setenv.sh

Configure setenv.sh as follows;

# vi setenv.sh 

#!/bin/sh
#
#===============================================
# MUST: Customize this to your local env
#===============================================
#
# Directory where all domains/db data etc are
# kept. Directories will be created here
export DC_USERHOME=/scratch/${USER}/docker

# Registry names where requisite standard images
# can be found
export DC_REGISTRY_SOA="localhost"
export DC_REGISTRY_DB="localhost"

# Proxy Environment
#export http_proxy=""
#export https_proxy=""
#export no_proxy=""

#===============================================
exportComposeEnv() {
#
export DC_HOSTNAME=`hostname -f`
#
# Used by Docker Compose from the env
# Oracle DB Parameters
#
export DC_ORCL_PORT=1521
export DC_ORCL_OEM_PORT=5500
export DC_ORCL_SID=soadb
export DC_ORCL_PDB=soapdb
export DC_ORCL_SYSPWD="password"
export DC_ORCL_HOST=${DC_HOSTNAME}
#
export DC_ORCL_DBDATA=${DC_USERHOME}/dbdata
#
# AdminServer Password
#
export DC_ADMIN_PWD="password"
#
# RCU Common password for all schemas + Prefix Names
#
export DC_RCU_SCHPWD="password"
export DC_RCU_SOAPFX=SOA01
export DC_RCU_BPMPFX=BPM01
export DC_RCU_OSBPFX=OSB01
#
# Domain directories for the various domain types
#
export DC_DDIR_SOA=${DC_USERHOME}/soadomain
export DC_DDIR_BPM=${DC_USERHOME}/bpmdomain
export DC_DDIR_OSB=${DC_USERHOME}/osbdomain
#
# Default version to use for compose images
#
export DC_SOA_VERSION=12.2.1.3
}

#===============================================
createDirs() {
mkdir -p ${DC_ORCL_DBDATA} ${DC_DDIR_SOA} ${DC_DDIR_BPM} ${DC_DDIR_OSB}
chmod 777 ${DC_ORCL_DBDATA} ${DC_DDIR_SOA} ${DC_DDIR_BPM} ${DC_DDIR_OSB}
}
#===============================================
#== MAIN starts here
#===============================================
#
echo "INFO: Setting up SOA Docker Environment..."
exportComposeEnv
createDirs
#echo "INFO: Environment variables"
#env | grep -e "DC_" | sort

Now Run setenv.sh;

# . ./setenv.sh
INFO: Setting up SOA Docker Environment...
INFO: Environment variables
DC_ADMIN_PWD=
DC_DDIR_BPM=/scratch/root/docker/bpmdomain
DC_DDIR_OSB=/scratch/root/docker/osbdomain
DC_DDIR_SOA=/scratch/root/docker/soadomain
DC_HOSTNAME=soa-k8s-001.sub09080508470.aeralpvcn01.oraclevcn.com
DC_ORCL_DBDATA=/scratch/root/docker/dbdata
DC_ORCL_HOST=soadb
DC_ORCL_OEM_PORT=5500
DC_ORCL_PDB=soapdb
DC_ORCL_PORT=1521
DC_ORCL_SID=soadb
DC_ORCL_SYSPWD=
DC_RCU_BPMPFX=BPM01
DC_RCU_OSBPFX=OSB01
DC_RCU_SCHPWD=
DC_RCU_SOAPFX=SOA01
DC_REGISTRY_DB=localhost
DC_REGISTRY_SOA=localhost
DC_SOA_VERSION=12.2.1.3
DC_USERHOME=/scratch/root/docker

Now Lets Run DB on Docker;

# cd samples

# docker-compose up -d soadb
Creating soadb … done

# docker logs -f soadb

[2022:11:28 15:50:34]: Acquiring lock .SOADB.create_lck with heartbeat 30 secs
[2022:11:28 15:50:34]: Lock acquired
[2022:11:28 15:50:34]: Starting heartbeat
[2022:11:28 15:50:34]: Lock held .SOADB.create_lck
ORACLE EDITION: ENTERPRISE
 
LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 28-NOV-2022 15:50:34
 
Copyright (c) 1991, 2019, Oracle.  All rights reserved.
 
Starting /opt/oracle/product/19c/dbhome_1/bin/tnslsnr: please wait...
 
TNSLSNR for Linux: Version 19.0.0.0.0 - Production
System parameter file is /opt/oracle/product/19c/dbhome_1/network/admin/listener.ora
Log messages written to /opt/oracle/diag/tnslsnr/17edb0e1fb36/listener/alert/log.xml
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1)))
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=0.0.0.0)(PORT=1521)))
 
Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 19.0.0.0.0 - Production
Start Date                28-NOV-2022 15:50:34
Uptime                    0 days 0 hr. 0 min. 0 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /opt/oracle/product/19c/dbhome_1/network/admin/listener.ora
Listener Log File         /opt/oracle/diag/tnslsnr/17edb0e1fb36/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=0.0.0.0)(PORT=1521)))
The listener supports no services
The command completed successfully
Prepare for db operation
8% complete
Copying database files
31% complete
Creating and starting Oracle instance
32% complete
36% complete
40% complete
43% complete
46% complete
Completing Database Creation
 
51% complete
54% complete
Creating Pluggable Databases
58% complete
77% complete
Executing Post Configuration Actions
100% complete
Database creation complete. For details check the logfiles at:
 /opt/oracle/cfgtoollogs/dbca/SOADB.
Database Information:
Global Database Name:SOADB
System Identifier(SID):SOADB
Look at the log file "/opt/oracle/cfgtoollogs/dbca/SOADB/SOADB.log" for further details.
 
SQL*Plus: Release 19.0.0.0.0 - Production on Mon Nov 28 16:05:43 2022
Version 19.3.0.0.0
 
Copyright (c) 1982, 2019, Oracle.  All rights reserved.
 
 
Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0
 
SQL> 
System altered.
 
SQL> 
System altered.
 
SQL> 
Pluggable database altered.
 
SQL> 
PL/SQL procedure successfully completed.
 
SQL> SQL> 
Session altered.
 
SQL> 
User created.
 
SQL> 
Grant succeeded.
 
SQL> 
Grant succeeded.
 
SQL> 
Grant succeeded.
 
SQL> 
User altered.
 
SQL> SQL> Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0
The Oracle base remains unchanged with value /opt/oracle
 
Executing user defined scripts
/opt/oracle/runUserScripts.sh: running /opt/oracle/scripts/extensions/setup/swapLocks.sh
[2022:11:28 16:05:46]: Releasing lock .SOADB.create_lck
[2022:11:28 16:05:46]: Lock released .SOADB.create_lck
[2022:11:28 16:05:46]: Acquiring lock .SOADB.exist_lck with heartbeat 30 secs
[2022:11:28 16:05:46]: Lock acquired
[2022:11:28 16:05:46]: Starting heartbeat
[2022:11:28 16:05:46]: Lock held .SOADB.exist_lck
 
DONE: Executing user defined scripts
 
The Oracle base remains unchanged with value /opt/oracle
#########################
DATABASE IS READY TO USE!
#########################
The following output is now a tail of the alert.log:
SOAPDB(3):ALTER DATABASE DEFAULT TABLESPACE "USERS"
SOAPDB(3):Completed: ALTER DATABASE DEFAULT TABLESPACE "USERS"
2022-11-28T16:05:43.147231+00:00
ALTER SYSTEM SET control_files='/opt/oracle/oradata/SOADB/control01.ctl' SCOPE=SPFILE;
2022-11-28T16:05:43.153771+00:00
ALTER SYSTEM SET local_listener='' SCOPE=BOTH;
   ALTER PLUGGABLE DATABASE SOAPDB SAVE STATE
Completed:    ALTER PLUGGABLE DATABASE SOAPDB SAVE STATE
 
XDB initialized.





Now Lets Start SOA AdminServer;

# docker-compose up -d soaas
Creating soaas ... done

# docker logs -f soaas

INFO: CONNECTION_STRING = soadb:1521/soapdb
INFO: RCUPREFIX         = SOA01
INFO: Copying RCU backup file
INFO: Setting RCU for ORCL
INFO: Dropping Schema SOA01...
 
              RCU Logfile: /tmp/RCU2022-11-28_16-07_1143364844/logs/rcu.log
 
 
 
Processing command line ....
Repository Creation Utility - Checking Prerequisites
Checking Global Prerequisites
 
ERROR - RCU-6013 Prefix validation failed.
CAUSE - RCU-6013 Specified prefix could not be found. Existing prefixes are: None
ACTION - RCU-6013 Specify a valid prefix from the existing prefix list.
 
ERROR - RCU-6091 Component validation failed.
CAUSE - RCU-6091 One or more component specific validation failed.
ACTION - RCU-6091 Check log for more details and fix the validation error.
INFO: Creating Schema SOA01...
 
              RCU Logfile: /tmp/RCU2022-11-28_16-07_1995331943/logs/rcu.log
 
 
 
Processing command line ....
Repository Creation Utility - Checking Prerequisites
Checking Global Prerequisites
The selected database is more recent than the supported list of certified databases for this version of Oracle Fusion Middleware. For the most recent list of certified databases, refer to the Supported System Configurations information on the Oracle Technology Network.
 
 
Repository Creation Utility - Checking Prerequisites
Checking Component Prerequisites
Repository Creation Utility - Creating Tablespaces
Validating and Creating Tablespaces
Repository Creation Utility - Create
Repository Create in progress.
Percent Complete: 11
Percent Complete: 27
Percent Complete: 27
Percent Complete: 28
Percent Complete: 29
Percent Complete: 30
Percent Complete: 30
Percent Complete: 30
Percent Complete: 38
Percent Complete: 38
Percent Complete: 47
Percent Complete: 47
Percent Complete: 47
Percent Complete: 54
Percent Complete: 54
Percent Complete: 63
Percent Complete: 63
Percent Complete: 63
Percent Complete: 70
Percent Complete: 70
Percent Complete: 71
Percent Complete: 71
Percent Complete: 72
Percent Complete: 72
Percent Complete: 80
Percent Complete: 80
Percent Complete: 80
Percent Complete: 81
Percent Complete: 82
Percent Complete: 83
Percent Complete: 83
Percent Complete: 88
Percent Complete: 88
Percent Complete: 89
Percent Complete: 90
Percent Complete: 92
Percent Complete: 94
Percent Complete: 94
Percent Complete: 94
Percent Complete: 98
Percent Complete: 98
Percent Complete: 100
 
Repository Creation Utility: Create - Completion Summary
 
Database details:
-----------------------------
Host Name                                    : soadb
Port                                         : 1521
Service Name                                 : SOAPDB
Connected As                                 : sys
Prefix for (prefixable) Schema Owners        : SOA01
RCU Logfile                                  : /tmp/RCU2022-11-28_16-07_1995331943/logs/rcu.log
 
Component schemas created:
-----------------------------
Component                                    Status         Logfile                     
 
Common Infrastructure Services               Success        /tmp/RCU2022-11-28_16-07_1995331943/logs/stb.log 
Oracle Platform Security Services            Success        /tmp/RCU2022-11-28_16-07_1995331943/logs/opss.log 
SOA Infrastructure                           Success        /tmp/RCU2022-11-28_16-07_1995331943/logs/soainfra.log 
User Messaging Service                       Success        /tmp/RCU2022-11-28_16-07_1995331943/logs/ucsums.log 
Audit Services                               Success        /tmp/RCU2022-11-28_16-07_1995331943/logs/iau.log 
Audit Services Append                        Success        /tmp/RCU2022-11-28_16-07_1995331943/logs/iau_append.log 
Audit Services Viewer                        Success        /tmp/RCU2022-11-28_16-07_1995331943/logs/iau_viewer.log 
Metadata Services                            Success        /tmp/RCU2022-11-28_16-07_1995331943/logs/mds.log 
WebLogic Services                            Success        /tmp/RCU2022-11-28_16-07_1995331943/logs/wls.log 
 
Repository Creation Utility - Create : Operation Completed
 
Initializing WebLogic Scripting Tool (WLST) ...
 
Welcome to WebLogic Server Administration Scripting Shell
 
Type help() for help on available commands
 
createDomain.py called with the following inputs:
INFO: sys.argv[0] = /u01/oracle/dockertools/createDomain.py
INFO: sys.argv[1] = -oh
INFO: sys.argv[2] = /u01/oracle
INFO: sys.argv[3] = -jh
INFO: sys.argv[4] = /usr/java/default
INFO: sys.argv[5] = -parent
INFO: sys.argv[6] = /u01/oracle/user_projects/domains
INFO: sys.argv[7] = -name
INFO: sys.argv[8] = infra_domain
INFO: sys.argv[9] = -password
INFO: sys.argv[10] = Welcome1
INFO: sys.argv[11] = -rcuDb
INFO: sys.argv[12] = soadb:1521/soapdb
INFO: sys.argv[13] = -rcuPrefix
INFO: sys.argv[14] = SOA01
INFO: sys.argv[15] = -rcuSchemaPwd
INFO: sys.argv[16] = Welcome1
INFO: sys.argv[17] = -domainType
INFO: sys.argv[18] = soa
INFO: Creating Node Managers...
INFO: Creating Admin server...
INFO: SOA Servers created.....
INFO: Writing base domain...
INFO: Base domain created at /u01/oracle/user_projects/domains/infra_domain
INFO: Extending domain at /u01/oracle/user_projects/domains/infra_domain
INFO: Applying JRF templates...
INFO: Applying SOA templates...
INFO: Extension Templates added
INFO: Configuring the Service Table DataSource...
INFO: Getting Database Defaults...
INFO: Targeting Server Groups...
INFO: Set CoherenceClusterSystemResource to defaultCoherenceCluster for server:soa_server1
INFO: Set CoherenceClusterSystemResource to defaultCoherenceCluster for cluster:soa_cluster
INFO: Set WLS clusters as target of defaultCoherenceCluster:[soa_cluster]
INFO: Preparing to update domain...
Nov 28, 2022 4:09:31 PM oracle.mds
WARNING: MDS-11019: The default CharSet US-ASCII is not a unicode character set. File names with non-ASCII characters may not operate as expected. Check locale settings.
INFO: Domain updated successfully
INFO: Updating the listen address - 172.18.0.3  soa-k8s-001.sub09080508470.aeralpvcn01.oraclevcn.com
/u01/oracle/oracle_common/common/bin/wlst.sh -skipWLSModuleScanning /u01/oracle/dockertools/updListenAddress.py u01 172.18.0.3 AdminServer soa-k8s-001.sub09080508470.aeralpvcn01.oraclevcn.com
INFO: Starting the Admin Server...
INFO: Logs = /u01/oracle/user_projects/domains/infra_domain/logs/as.log
     
INFO: Admin server is running
INFO: Admin server running, ready to start managed server



Start SOA Managed Server;

# docker-compose up -d soams
soaas is up-to-date
Creating soams ... done

# docker logs -f soams
INFO: Updating the listen address - 172.18.0.4  soa-k8s-001.sub09080508470.aeralpvcn01.oraclevcn.com
INFO: Starting the managed server soa_server1
INFO: Waiting for the Managed Server to accept requests...
SOA Platform is running and accepting requests. Start up took 11230 ms, partition=DOMAIN
INFO: Managed Server is running
INFO: Managed server has been started




Stop SOA Servers;
# docker ps -a

# docker stop container soams
soams
# docker container stop soaas
soaas


Start BPM Admin Server;

# cd /u01/docker-images/OracleSOASuite/samples/
# docker-compose up -d bpmas
Creating bpmas ... done
# docker logs -f bpmas
INFO: CONNECTION_STRING = soadb:1521/soapdb
INFO: RCUPREFIX         = BPM01
INFO: Copying RCU backup file
INFO: Setting RCU for ORCL
INFO: Dropping Schema BPM01...
 
              RCU Logfile: /tmp/RCU2022-11-28_16-27_1081384844/logs/rcu.log
 
 
 
Processing command line ....
Repository Creation Utility - Checking Prerequisites
Checking Global Prerequisites
 
ERROR - RCU-6013 Prefix validation failed.
CAUSE - RCU-6013 Specified prefix could not be found. Existing prefixes are: SOA01
ACTION - RCU-6013 Specify a valid prefix from the existing prefix list.
 
ERROR - RCU-6091 Component validation failed.
CAUSE - RCU-6091 One or more component specific validation failed.
ACTION - RCU-6091 Check log for more details and fix the validation error.
INFO: Creating Schema BPM01...
 
              RCU Logfile: /tmp/RCU2022-11-28_16-27_839787900/logs/rcu.log
 
 
 
Processing command line ....
Repository Creation Utility - Checking Prerequisites
Checking Global Prerequisites
The selected database is more recent than the supported list of certified databases for this version of Oracle Fusion Middleware. For the most recent list of certified databases, refer to the Supported System Configurations information on the Oracle Technology Network.
 
 
Repository Creation Utility - Checking Prerequisites
Checking Component Prerequisites
Repository Creation Utility - Creating Tablespaces
Validating and Creating Tablespaces
Repository Creation Utility - Create
Repository Create in progress.
Percent Complete: 18
Percent Complete: 18
Percent Complete: 19
Percent Complete: 20
Percent Complete: 21
Percent Complete: 21
Percent Complete: 21
Percent Complete: 29
Percent Complete: 29
Percent Complete: 38
Percent Complete: 38
Percent Complete: 38
Percent Complete: 45
Percent Complete: 45
Percent Complete: 54
Percent Complete: 54
Percent Complete: 54
Percent Complete: 61
Percent Complete: 61
Percent Complete: 62
Percent Complete: 62
Percent Complete: 63
Percent Complete: 63
Percent Complete: 71
Percent Complete: 71
Percent Complete: 71
Percent Complete: 72
Percent Complete: 73
Percent Complete: 74
Percent Complete: 74
Percent Complete: 79
Percent Complete: 79
Percent Complete: 80
Percent Complete: 81
Percent Complete: 83
Percent Complete: 85
Percent Complete: 85
Percent Complete: 85
Percent Complete: 89
Percent Complete: 89
Percent Complete: 92
Percent Complete: 92
Percent Complete: 100
 
Repository Creation Utility: Create - Completion Summary
 
Database details:
-----------------------------
Host Name                                    : soadb
Port                                         : 1521
Service Name                                 : SOAPDB
Connected As                                 : sys
Prefix for (prefixable) Schema Owners        : BPM01
RCU Logfile                                  : /tmp/RCU2022-11-28_16-27_839787900/logs/rcu.log
 
Component schemas created:
-----------------------------
Component                                    Status         Logfile                     
 
Common Infrastructure Services               Success        /tmp/RCU2022-11-28_16-27_839787900/logs/stb.log 
Oracle Platform Security Services            Success        /tmp/RCU2022-11-28_16-27_839787900/logs/opss.log 
SOA Infrastructure                           Success        /tmp/RCU2022-11-28_16-27_839787900/logs/soainfra.log 
User Messaging Service                       Success        /tmp/RCU2022-11-28_16-27_839787900/logs/ucsums.log 
Audit Services                               Success        /tmp/RCU2022-11-28_16-27_839787900/logs/iau.log 
Audit Services Append                        Success        /tmp/RCU2022-11-28_16-27_839787900/logs/iau_append.log 
Audit Services Viewer                        Success        /tmp/RCU2022-11-28_16-27_839787900/logs/iau_viewer.log 
Metadata Services                            Success        /tmp/RCU2022-11-28_16-27_839787900/logs/mds.log 
WebLogic Services                            Success        /tmp/RCU2022-11-28_16-27_839787900/logs/wls.log 
 
Repository Creation Utility - Create : Operation Completed
 
Initializing WebLogic Scripting Tool (WLST) ...
 
Welcome to WebLogic Server Administration Scripting Shell
 
Type help() for help on available commands
 
createDomain.py called with the following inputs:
INFO: sys.argv[0] = /u01/oracle/dockertools/createDomain.py
INFO: sys.argv[1] = -oh
INFO: sys.argv[2] = /u01/oracle
INFO: sys.argv[3] = -jh
INFO: sys.argv[4] = /usr/java/default
INFO: sys.argv[5] = -parent
INFO: sys.argv[6] = /u01/oracle/user_projects/domains
INFO: sys.argv[7] = -name
INFO: sys.argv[8] = infra_domain
INFO: sys.argv[9] = -password
INFO: sys.argv[10] = Welcome1
INFO: sys.argv[11] = -rcuDb
INFO: sys.argv[12] = soadb:1521/soapdb
INFO: sys.argv[13] = -rcuPrefix
INFO: sys.argv[14] = BPM01
INFO: sys.argv[15] = -rcuSchemaPwd
INFO: sys.argv[16] = Welcome1
INFO: sys.argv[17] = -domainType
INFO: sys.argv[18] = bpm
INFO: Creating Node Managers...
INFO: Creating Admin server...
INFO: SOA Servers created.....
INFO: Writing base domain...
INFO: Base domain created at /u01/oracle/user_projects/domains/infra_domain
INFO: Extending domain at /u01/oracle/user_projects/domains/infra_domain
INFO: Applying JRF templates...
INFO: Applying BPM templates...
INFO: Extension Templates added
INFO: Configuring the Service Table DataSource...
INFO: Getting Database Defaults...
INFO: Targeting Server Groups...
INFO: Set CoherenceClusterSystemResource to defaultCoherenceCluster for server:soa_server1
INFO: Set CoherenceClusterSystemResource to defaultCoherenceCluster for cluster:soa_cluster
INFO: Set WLS clusters as target of defaultCoherenceCluster:[soa_cluster]
INFO: Preparing to update domain...
Nov 28, 2022 4:29:36 PM oracle.mds
WARNING: MDS-11019: The default CharSet US-ASCII is not a unicode character set. File names with non-ASCII characters may not operate as expected. Check locale settings.
INFO: Domain updated successfully
INFO: Updating the listen address - 172.18.0.3  soa-k8s-001.sub09080508470.aeralpvcn01.oraclevcn.com
/u01/oracle/oracle_common/common/bin/wlst.sh -skipWLSModuleScanning /u01/oracle/dockertools/updListenAddress.py u01 172.18.0.3 AdminServer soa-k8s-001.sub09080508470.aeralpvcn01.oraclevcn.com
INFO: Starting the Admin Server...
INFO: Logs = /u01/oracle/user_projects/domains/infra_domain/logs/as.log
     
INFO: Admin server is running
INFO: Admin server running, ready to start managed server




Start BPM Managed Server;

# docker-compose up -d bpmms
bpmas is up-to-date
Creating bpmms ... done

# docker logs -f bpmms
INFO: Updating the listen address - 172.18.0.4  soa-k8s-001.sub09080508470.aeralpvcn01.oraclevcn.com
INFO: Starting the managed server soa_server1
INFO: Waiting for the Managed Server to accept requests...
SOA Platform is running and accepting requests. Start up took 36689 ms, partition=DOMAIN
INFO: Managed Server is running
INFO: Managed server has been started


Stop BPM Servers;

# docker container stop bpmms
bpmms
# docker container stop bpmas
bpmas


You can also Start OSB Admin and Managed Servers in the same way.
Have Fun!!







