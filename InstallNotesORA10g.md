
##Installing Oracle 10g R2 on Debian 7.

Don't run create database step with dbca: Create database manually and run dbms script. 

Use 10201_database_linux_x86_64.cpio.gz archive.

In this installation, my environment is:
```
/app/oracle/product10 for oracle engine (ORACLE_HOME)
/app/oracle/oraInvetory 
/data/oracle/oradata for database data
DBORA for SID and Database.
/home/oracle for oracle user unix
```

###Tuning Kernel:
* Example with 4GB of RAM. Take half of physical memory for shmmax.
```
   kernel.shmall = 524288.  
   #shmmax/PAGE_SIZE.
   kernel.shmmax = 2147483648 (2GO ici).
   #half of machine memory.
   kernel.shmmni = 4096.
   kernel.sem = 250 32000 100 128.
   fs.file-max = 65536.
   net.ipv4.ip_local_port_range = 1024 65000.
   net.core.rmem_default = 262144.
   net.core.rmem_max = 262144.
```
###tuning shell:
/etc/security/limits.conf.
```
  oracle soft nproc 2047
  oracle hard nproc 16384
  oracle soft nofile 1024
  oracle hard nofile 65536
```

###Group and users:
```
Add group:
groupadd oinstall
groupadd dba
addgroup nobody

Oracle user:

adduser --ingroup oinstall --home /home/oracle --shell /bin/bash --uid 1020 oracle
usermod -a -G dba oracle
usermod -g nobody nobody
chown -R oracle:oinstall /home/oracle

mkdir -p /app/oracle/oraInventory
mkdir -p /app/oracle/install
mkdir -p /data/oracle/oradata
chown -R or```le.dba /data/oracle/oradata (répertoire des données)
chown -R oracle. /app/oracle

```


### I'm redhat:

* create file redhat-release
```
vi /etc/redhat-release
Red Hat Enterprise Linux Server release 4  (Nahant)
```

###Enable i386 packages installation with add 32bits architecture: for ia32-libs :
```
dpkg --add-architecture i386
apt-get update
```

### Packages installation:
* Run this for apt-get install.  don't worry, it's interactive step.
* See if there are remove packages.
```
for i in autoconf automake binutils bzip2 doxygen gcc less libc6-dev make perl-doc unzip zlibc gcc make binutils gawk x11-utils autotools-dev libltdl-dev libaio1 lesstif2 libmotif4 libaio-dev ksh libpthread-stubs0 libpthread-stubs0-dev libpth-dev libc6-i386  libc6-dev-i386 g++-multilib gcc-multilib xscreensaver ia32-libs ; do apt-get install $i; done

apt-get install gcc make binutils gawk x11-utils rpm alien
apt-get install libstdc++5
apt-get install rlwrap (for history and recall sql commands: great tool!)

Symbolic link:
cd /bin

ln -s /usr/bin/rpm
ln -s /usr/bin/awk
ln -s /usr/bin/basename

copy 10201_database_linux_x86_64.cpio.gz under /app/oracle/install/ and gunzip
cd /app/oracle/install/
cpio -idmv < 10201_database_linux_x86_64.cpio
```
* for graphical install, authorize X11 forwarding and start X server on machine. 
* ssh connect with oracle user and launch under /app/oracle/install/database/:
```
cd /app/oracle/install/database/
./runInstaller 
```

#### Graphiques step:

1:
Oracle Home Location: /app/oracle/product10 (where you want to install engine)
Don't check box who "Create starter database (additional 720MB)"
and clic on "next"
![First Step](https://github.com/lgattoni/Oracle10GSetup/blob/master/First-orasetp1.png?raw=true) 

2: Specify Inventory directory and credentials
/app/oracle/oraInventory (where you want Inventory)

"specify operating System group name" 
choose "dba"
and clic on "next"

3:
Summary
and clic on "install"

4:
Installation is running...

5:
clic continue on error with makefile ins_rdbms.mk

6:
run script in popup as root in another term.
/app/oracle/oraInventory/orainstRoot.sh (if directory premissions ok, there is no this script)
/app/oracle/oraInventory/Root.sh

7:
End of installation, exit and yes.


Start listener:

with user oracle/unix
```
export ORACLE_HOME=/app/oracle/product10
export PATH=/app/oracle/product10/bin:$PATH
export ORACLE_SID=DBORA
export DISABLE_HUGETLBFS=1    (deactivate hugepages: activate if use. In this example, startup faile if no deactivate)
PS1="\\[\\033[0;31m\\][\ENV]\\[\\033[00m\\]\${debian_chroot:+(\$debian_chroot)}\\[\\033[01;32m\\]\\u@\\h\\[\\033[00m\\]:\\[\\033[01;34m\\]\\w\\[\\033[00m\\][${ORACLE_SID} Instance]\\$ " (for fun)

==> input this lines in .bash_profile for oracle user


/app/oracle/product10/bin/lsnrctl start LISTENER
============================>  
LSNRCTL for Linux: Version 10.2.0.1.0 - Production on 25-FEB-2015 16:21:40

Copyright (c) 1991, 2005, Oracle.  All rights reserved.

Starting /app/oracle/product10/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 10.2.0.1.0 - Production
Log messages written to /app/oracle/product10/network/log/listener.log
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=myhost.mydomain.com)(PORT=1521)))

Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 10.2.0.1.0 - Production
Start Date                25-FEB-2015 16:21:40
Uptime                    0 days 0 hr. 0 min. 0 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Log File         /app/oracle/product10/network/log/listener.log
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=myhost.mydomaine.com)(PORT=1521)))
The listener supports no services
The command completed successfully
====================================================================================
```
* Create password file:
```
cd /app/oracle/product10/dbs

orapwd file=orapwDBORA password=password entries=10
```
* Create init(SID).ora
```
cd /app/oracle/product10/dbs
cp init.ora initDBORA.ora


Parameters in file: 
DB_CREATE_FILE_DEST = /data/oracle/oradata     (add)
shared_pool_size = 268435456                      (modify)
UNDO_MANAGEMENT = AUTO                            (add)
streams_pool_size = 41943040                  (40M add)



mkdir -p  /data/oracle/oradata/DBORA
```
* Create spfile:
* Run sqlplus with rlwrap:
```
rlwrap sqlplus / as sysdba ==> (rlwrap for history and recall commands)

SQL> CREATE SPFILE='/app/oracle/product10/dbs/spfileDBORA.ora' FROM PFILE='/app/oracle/product10/dbs/initDBORA.ora';

SQL> startup nomount
ORACLE instance started.

Total System Global Area  343932928 bytes
Fixed Size                  2020640 bytes
Variable Size             335547104 bytes
Database Buffers            4194304 bytes
Redo Buffers                2170880 bytes
```
#### Database creation:
* build file createDBORA.sql with this informations, or other you want:

```
=================================== DBORA

CREATE DATABASE DBORA
   USER SYS IDENTIFIED BY password
   USER SYSTEM IDENTIFIED BY password
   LOGFILE GROUP 1 ('/data/oracle/oradata/DBORA/redo01.log') SIZE 100M,
           GROUP 2 ('/data/oracle/oradata/DBORA/redo02.log') SIZE 100M,
           GROUP 3 ('/data/oracle/oradata/DBORA/redo03.log') SIZE 100M
   MAXLOGFILES 5
   MAXLOGMEMBERS 5
   MAXLOGHISTORY 1
   MAXDATAFILES 100
   MAXINSTANCES 1
   CHARACTER SET AL32UTF8
   NATIONAL CHARACTER SET AL16UTF16
   DATAFILE '/data/oracle/oradata/DBORA/system01.dbf' SIZE 325M REUSE
   EXTENT MANAGEMENT LOCAL
   SYSAUX DATAFILE '/data/oracle/oradata/DBORA/sysaux01.dbf' SIZE 325M REUSE
   DEFAULT TABLESPACE tbs_1
   DEFAULT TEMPORARY TABLESPACE tempts1
      TEMPFILE '/data/oracle/oradata/DBORA/temp01.dbf' 
      SIZE 20M REUSE
   UNDO TABLESPACE undotbs 
   DATAFILE '/data/oracle/oradata/DBORA/undotbs01.dbf'
      SIZE 200M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED;



====================================
```

* Launch sql file:
```
SQL> @createDBORA.sql;

Database created.

```

####Install DBMS packages :
lrwrap sqlplus / as sysdba ==>

* Running this scripts:
```
SQL> @$ORACLE_HOME/rdbms/admin/catalog.sql;
SQL> @$ORACLE_HOME/rdbms/admin/catproc.sql;
SQL> @$ORACLE_HOME/rdbms/admin/utlrp.sql;
```

### Use  dbshut and dbstart

*  configure ORATAB : 

```
echo "DBORA:/app/oracle/product10:Y" >> /etc/oratab 

modify path in $ORACLE_HOME/bin/dbstart


good path on ORACLE_HOME_LISTNER 
ORACLE_HOME_LISTNER=$ORACLE_HOME
```

*  Stop oracle:
```
dbshut &&  /app/oracle/product10/bin/lsnrctl stop LISTENER
```
* start Oracle:
```
dbstart
```
* Stop/start on sqlplus:
```
SQL>shutdown
SQL>startup
```

### Init script for stop/start by root and after reboot:
```
Under /etc/init.d
create dbora file with this shell :
=====================================================================
#!/bin/sh
### BEGIN INIT INFO
# Provides:          dbora
# Required-Start:    $remote_fs $syslog open-iscsi iscsi
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Oracle 12
# Description:       Oracle 12 database
### END INIT INFO

ORACLE_HOME=/app/oracle/product10
ORACLE_OWNER=oracle

if [ ! -f $ORACLE_HOME/bin/dbstart ]
then
    echo "Oracle startup: cannot start"
    exit
fi

case "$1" in
    'start')
        # Start the Oracle databases:
        # The following command assumes that the oracle login
        # will not prompt the user for any values
        su - $ORACLE_OWNER -c "$ORACLE_HOME/bin/lsnrctl start"
        su - $ORACLE_OWNER -c $ORACLE_HOME/bin/dbstart
        ;;
    'stop')
        # Stop the Oracle databases:
        # The following command assumes that the oracle login
        # will not prompt the user for any values
        su - $ORACLE_OWNER -c $ORACLE_HOME/bin/dbshut
        su - $ORACLE_OWNER -c "$ORACLE_HOME/bin/lsnrctl stop"
        ;;
esac
========================================================
chmod +x /etc/init.d/dbora
Automation stop/start:  update-rc dbora defaults

Make a test with root user:
/etc/init.d/dbora [stop,start]

ps -ef |grep ora
oracle    9690     1  0 11:48 ?        00:00:00 /app/oracle/product10/bin/tnslsnr LISTENER -inherit
oracle    9767     1  0 11:48 ?        00:00:00 ora_pmon_DBORA
oracle    9769     1  0 11:48 ?        00:00:00 ora_psp0_DBORA
oracle    9771     1  0 11:48 ?        00:00:00 ora_mman_DBORA
oracle    9773     1  0 11:48 ?        00:00:00 ora_dbw0_DBORA
oracle    9775     1  0 11:48 ?        00:00:00 ora_lgwr_DBORA
oracle    9777     1  0 11:48 ?        00:00:00 ora_ckpt_DBORA
oracle    9779     1  0 11:48 ?        00:00:00 ora_smon_DBORA
oracle    9781     1  0 11:48 ?        00:00:00 ora_reco_DBORA
oracle    9783     1  1 11:48 ?        00:00:00 ora_mmon_DBORA
oracle    9785     1  0 11:48 ?        00:00:00 ora_mmnl_DBORA
oracle    9789     1  0 11:48 ?        00:00:00 ora_qmnc_DBORA
```

##Great, you have a Oracle 10g R2 installed on your debian 7 :-)  
