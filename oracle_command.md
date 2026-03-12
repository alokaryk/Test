SQL> SELECT process, status, thread#, sequence#
FROM v$managed_standby;  2

PROCESS   STATUS          THREAD#  SEQUENCE#
--------- ------------ ---------- ----------
ARCH      CONNECTED             0          0
DGRD      ALLOCATED             0          0
DGRD      ALLOCATED             0          0
ARCH      CONNECTED             0          0
ARCH      CONNECTED             0          0
ARCH      CONNECTED             0          0
RFS       IDLE                  2      12337
MRP0      WAIT_FOR_LOG          2      12337
RFS       IDLE                  1      14754
RFS       IDLE                  0          0
RFS       IDLE                  0          0

PROCESS   STATUS          THREAD#  SEQUENCE#
--------- ------------ ---------- ----------
RFS       IDLE                  0          0
RFS       IDLE                  1          0

13 rows selected.

SQL> SELECT name, value, time_computed
FROM v$dataguard_stats
WHERE name IN ('transport lag', 'apply lag');  2    3

NAME
--------------------------------
VALUE
----------------------------------------------------------------
TIME_COMPUTED
------------------------------
transport lag
+00 00:54:39
03/11/2026 11:58:41

apply lag
+00 00:54:39
03/11/2026 11:58:41

NAME
--------------------------------
VALUE
----------------------------------------------------------------
TIME_COMPUTED
------------------------------

AT DC

[oracle@erpproddb1-dr ~]$ sqlplus sys/sys123@PRODCDB_PROD as sysdba

SQL*Plus: Release 19.0.0.0.0 - Production on Wed Mar 11 11:59:31 2026
Version 19.27.0.0.0

Copyright (c) 1982, 2024, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.27.0.0.0

SQL> archive log list;
Database log mode              Archive Mode
Automatic archival             Enabled
Archive destination            USE_DB_RECOVERY_FILE_DEST
Oldest online log sequence     14638
Next log sequence to archive   14640
Current log sequence           14640
SQL> exit
Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.27.0.0.0
[oracle@erpproddb1-dr ~]$
