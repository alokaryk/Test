<p class="demoTitle"><a href="https://serhatcelik.wordpress.com/2021/02/15/step-by-step-oracle-19c-rac-installation-on-oracle-linux-7-9/">Step By Step Oracle 19C RAC Installation on Oracle Linux 7.9 Part-1 OS | SERHAT CELIK DATABASE BLOG</a></p>
<p class="demoTitle">&nbsp;</p>
<p class="demoTitle"><a href="https://www.br8dba.com/create-rac-physical-standby-database/">Create RAC Physical Standby Database</a></p>
<p class="demoTitle">&nbsp;</p>
<p class="demoTitle"><a href="https://oracledbwr.com/duplicate-a-database-using-rman-in-oracle-database-19c-release/">Duplicate a Database Using RMAN in Oracle Database 19c Release | Oracledbwr</a></p>
<p class="demoTitle">&nbsp;</p>
<p class="demoTitle">&nbsp;</p>
<p class="demoTitle"><a href="https://ace2oracle.com/listing-detail.php?title=creating-an-oracle-19c-physical-standby-rac-database-from-a-primary-rac-database-using-active-duplication&amp;id=113">Ace2oracle | Creating An Oracle 19c Physical Standby Rac Database From A Primary Rac Database Using Active Duplication</a></p>
<p class="demoTitle">&nbsp;</p>
<p class="demoTitle">&nbsp;</p>
<p class="demoTitle">&nbsp;</p>
<p>Step by Step Guide on Creating Physical Standby Using RMAN DUPLICATE...FROM ACTIVE DATABASE</p>
<p>KB135553</p>
<p>Last Updated</p>
<p>Dec 19, 2025</p>
<p>Service</p>
<p>Oracle Database - Enterprise Edition</p>
<p>1.0</p>
<p><strong>Authoring Instructions</strong></p>
<p>This is a crossover article and must be edited only in Legacy MOS for Document ID : 1075908.1. Any changes made here will be lost in the next update.</p>
<p><strong>Applies To</strong></p>
<p>All Users</p>
<p><strong>Summary</strong></p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>Step by step guide on how to create a physical standby database using RMAN DUPLICATE FROM ACTIVE DATABASE command without shutting down the primary and using primary active database files (No need to take a backup)</p>
<p>This is feature available in Oracle11g onwards.</p>
<p><br /> Database Name : chicago<br /> Primary db_unique_name : chicago<br /> standby db_unique_name : Boston</p>
<p><strong>NOTE: In the images and/or the document content below, the user information and environment data used represents fictitious data from the Oracle sample schema(s), Public Documentation delivered with an Oracle database product or other training material. Any similarity to actual environments, actual persons, living or dead, is purely coincidental and not intended in any manner.</strong></p>
<p><br /> For duplicating a NON-Standby database, see :<br /> &nbsp;<a href="https://fa-etmi-saasfaprod1.fa.ocs.oraclecloud.com/fscmUI/redwood/myknowledge/content/container/main/article?answerId=452868">KB137059</a>&nbsp;: &nbsp;RMAN 'Duplicate From Active Database' Feature in 11G</p>
<p>&nbsp;</p>
<p><strong>Solution</strong></p>
<ol>
<li>Make the necessary changes to the primary database.&nbsp;<br /> &nbsp; &nbsp; a. Enable force logging.<br /> &nbsp;&nbsp;&nbsp; b. Creating the password file if one does not exist.<br /> &nbsp;&nbsp;&nbsp; c. Create standby redologs.<br /> &nbsp;&nbsp;&nbsp; d. Modify the parameter file suitable for Dataguard.</li>
<li>Ensure that the sql*net connectivity is working fine.</li>
<li>Create the standby database over the network using the active(primary) database files.<br /> &nbsp;&nbsp;&nbsp; a. Create the password file<br /> &nbsp;&nbsp;&nbsp; b. Create the initialization parameter file for the standby database (auxiliary database)<br /> &nbsp;&nbsp;&nbsp; c. Create the necessary mount points or the folders for the database files<br /> &nbsp;&nbsp;&nbsp; d. Run the standby creation ON STANDBY by connecting to primary as target database.</li>
</ol>
<p>&nbsp;</p>
<p>DUPLICATE TARGET DATABASE&nbsp;&nbsp;&nbsp;<br /> FOR STANDBY&nbsp;&nbsp;<br /> FROM ACTIVE DATABASE&nbsp;&nbsp;<br /> SPFILE&nbsp;&nbsp;<br /> &nbsp;&nbsp; PARAMETER_VALUE_CONVERT '', ''&nbsp;&nbsp;<br /> &nbsp;&nbsp; SET DB_FILE_NAME_CONVERT '', ''&nbsp;&nbsp;<br /> &nbsp;&nbsp; SET LOG_FILE_NAME_CONVERT '', ''&nbsp;&nbsp;<br /> &nbsp;&nbsp; SET SGA_MAX_SIZE 200M&nbsp;&nbsp;<br /> &nbsp;&nbsp; SET SGA_TARGET 125M;</p>
<ol start="4">
<li>Check the log shipping and apply.</li>
</ol>
<p><strong>PROCEDURE:</strong></p>
<p>While creating the standby database we use the active database files i.e., this command will be useful in creating the physical standby database using active database files over the network.</p>
<ol>
<li><strong> Prepare the production database to be the primary database</strong></li>
<li>Ensure that the database is in archivelog mode .</li>
</ol>
<p>SQL&gt; select log_mode from v$database;&nbsp;&nbsp;<br /> <br /> LOG_MODE&nbsp;&nbsp;<br /> ------------&nbsp;&nbsp;<br /> ARCHIVELOG</p>
<ol>
<li><br /> Enable force logging</li>
</ol>
<p>SQL&gt; ALTER DATABASE FORCE LOGGING;</p>
<ol>
<li><br /> Create standby redologs</li>
</ol>
<p>SQL&gt; alter database add standby logfile '&lt;name&gt;' size &lt;size&gt;;</p>
<ol>
<li><br /> Modify the primary initialization parameter for dataguard on primary,</li>
</ol>
<p>SQL&gt; alter system set LOG_ARCHIVE_CONFIG='DG_CONFIG=(chicago,boston)';&nbsp;&nbsp;<br /> System altered.&nbsp;&nbsp;<br /> <br /> SQL&gt; alter system set LOG_ARCHIVE_DEST_1='LOCATION=/&lt;path&gt;/chicago/redo/ VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME=chicago';&nbsp;&nbsp;<br /> System altered.&nbsp;&nbsp;<br /> <br /> SQL&gt; alter system set LOG_ARCHIVE_DEST_2='SERVICE=boston LGWR ASYNC VALID_FOR=(ONLINE_LOGFILE,PRIMARY_ROLE) DB_UNIQUE_NAME=boston';&nbsp;&nbsp;<br /> System altered.&nbsp;&nbsp;<br /> <br /> SQL&gt; alter system set LOG_ARCHIVE_DEST_STATE_1=ENABLE;&nbsp;&nbsp;<br /> System altered.&nbsp;&nbsp;<br /> <br /> SQL&gt; alter system set FAL_SERVER=boston;&nbsp;&nbsp;<br /> System altered.&nbsp;&nbsp;<br /> <br /> SQL&gt; alter system set FAL_CLIENT=chicago;&nbsp;&nbsp;<br /> System altered.&nbsp;&nbsp;<br /> <br /> SQL&gt; alter system set DB_FILE_NAME_CONVERT='/&lt;path&gt;/boston/data/','/&lt;path&gt;/chicago/data' scope=spfile;&nbsp;&nbsp;<br /> System altered.&nbsp;&nbsp;<br /> <br /> SQL&gt; alter system set LOG_FILE_NAME_CONVERT='/&lt;path&gt;/boston/redo/','/&lt;path&gt;/chicago/redo' scope=spfile;&nbsp;&nbsp;<br /> System altered.</p>
<ol start="2">
<li><strong> Ensure that the sql*net connectivity is working fine.</strong></li>
</ol>
<p><em>Insert a static entry for Boston in the listener.ora file of the standby system.</em></p>
<p><em>NOTE : For non default port set the REMOTE_LISTENER&nbsp;<br /> <br /> &nbsp;</em></p>
<p>SID_LIST_LISTENER =&nbsp;<br /> &nbsp; (SID_LIST =&nbsp;<br /> &nbsp;&nbsp;&nbsp; (SID_DESC =&nbsp;<br /> &nbsp;&nbsp;&nbsp;&nbsp; (GLOBAL_DBNAME = boston.&lt;domain&gt;)&nbsp;<br /> &nbsp;&nbsp;&nbsp;&nbsp; (ORACLE_HOME = /&lt;oracle_home path&gt;)&nbsp;<br /> &nbsp;&nbsp;&nbsp;&nbsp; (SID_NAME = boston)&nbsp;<br /> &nbsp;&nbsp;&nbsp; )&nbsp;<br /> &nbsp;&nbsp; )&nbsp;<br /> <br /> LISTENER =&nbsp;<br /> &nbsp; (DESCRIPTION =&nbsp;<br /> &nbsp;&nbsp;&nbsp; (ADDRESS = (PROTOCOL = TCP)(HOST = &lt;auxiliary host&gt;)(PORT = 1521))&nbsp;<br /> &nbsp; )&nbsp;</p>
<p>&nbsp;</p>
<p><em>TNSNAMES.ORA for the Primary and Standby should have BOTH entries&nbsp;<br /> <br /> &nbsp;</em></p>
<p>CHICAGO =&nbsp;<br /> &nbsp; (DESCRIPTION =&nbsp;<br /> &nbsp;&nbsp;&nbsp; (ADDRESS_LIST =&nbsp;<br /> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (ADDRESS = (PROTOCOL = TCP)(HOST = &lt;target host&gt;)(PORT = 1521))&nbsp;<br /> &nbsp;&nbsp;&nbsp; )&nbsp;<br /> &nbsp;&nbsp;&nbsp; (CONNECT_DATA = (SERVICE_NAME = chicago.&lt;domain&gt;))&nbsp;<br /> &nbsp; )&nbsp;&nbsp;<br /> <br /> BOSTON =&nbsp;<br /> &nbsp; (DESCRIPTION =&nbsp;<br /> &nbsp;&nbsp;&nbsp; (ADDRESS_LIST =&nbsp;<br /> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (ADDRESS = (PROTOCOL = TCP)(HOST = &lt;auxiliary host&gt;)(PORT = 1521))&nbsp;<br /> &nbsp;&nbsp;&nbsp; )&nbsp;<br /> &nbsp;&nbsp;&nbsp; (CONNECT_DATA = (SERVICE_NAME = boston.&lt;domain&gt;))&nbsp;<br /> &nbsp; )</p>
<p><em>Check with the SQL*Net configuration using the following commands on the Primary AND Standby</em></p>
<p>% tnsping chicago&nbsp;&nbsp;<br /> % tnsping boston</p>
<ol start="3">
<li><strong> Create the standby database</strong></li>
<li>Copy the password file from the primary $ORACLE_HOME/dbs and rename it to the standby database name.</li>
</ol>
<p><em>The username is required to be SYS and the password needs to be the same on the Primary and Standby.&nbsp;<br /> The best practice for this is to copy the password file as suggested.&nbsp;<br /> The password file name must match the instance name/SID used at the standby site, not the DB_NAME.</em></p>
<ol>
<li>Create a initialization parameter with only one parameter DB_NAME.</li>
</ol>
<p><em>DB_NAME=chicago&nbsp;<br /> DB_UNIQUE_NAME=boston&nbsp;<br /> DB_BLOCK_SIZE=&lt;same as primary&gt;</em></p>
<ol>
<li>Create the necessary directories in the standby location to place database files and trace files ($ADR_HOME).<br /> <br /> d. Set the environment variable ORACLE_SID to the standby service and start the standby-instance.</li>
</ol>
<p>% export ORACLE_SID=boston&nbsp;&nbsp;<br /> % sqlplus "/ as sysdba"&nbsp;&nbsp;<br /> SQL&gt; startup nomount pfile=$ORACLE_HOME/dbs/initcore1.ora</p>
<p>NOTE : Use either PFILE or SPFILE&nbsp;&nbsp;<br /> <br /> # Addtl. comment&nbsp;&nbsp;<br /> # If DUPLICATE without TARGET connection is used you cannot use SPFILE&nbsp;&nbsp;<br /> # else getting&nbsp;&nbsp;<br /> <br /> RMAN-05537: DUPLICATE without TARGET connection when auxiliary instance is started with spfile cannot use SPFILE clause</p>
<ol>
<li><br /> Verify if the connection 'AS SYSDBA' is working</li>
</ol>
<p>% sqlplus /nolog&nbsp;&nbsp;<br /> SQL&gt; connect sys/&lt;passwd&lt;@boston AS SYSDBA&nbsp;&nbsp;<br /> SQL&gt; connect sys/&lt;passwd&gt;@chicago AS SYSDBA</p>
<ol>
<li><br /> On the primary system invoke the RMAN executable and connect to the primary and the auxiliary database ( i.e., the standby)</li>
</ol>
<p>$ rman target sys/&lt;password&gt;@chicago auxiliary sys/&lt;password&gt;@boston&nbsp;&nbsp;<br /> <br /> connected to target database: CHICAGO (DBID=761464750)&nbsp;&nbsp;<br /> connected to auxiliary database:&nbsp;CHICAGO (not mounted)&nbsp;&nbsp;<br /> <br /> RMAN&gt; run {&nbsp;&nbsp;<br /> allocate channel prmy1 type disk;&nbsp;&nbsp;<br /> allocate channel prmy2 type disk;&nbsp;&nbsp;<br /> allocate channel prmy3 type disk;&nbsp;&nbsp;<br /> allocate channel prmy4 type disk;&nbsp;&nbsp;<br /> allocate auxiliary channel stby type disk;&nbsp;&nbsp;<br /> <br /> duplicate target database for standby from active database&nbsp;&nbsp;<br /> spfile&nbsp;&nbsp;<br /> &nbsp; parameter_value_convert 'chicago','boston'&nbsp;&nbsp;<br /> &nbsp; set db_unique_name='boston'&nbsp;&nbsp;<br /> &nbsp; set db_file_name_convert='/chicago/','/boston/'&nbsp;&nbsp;<br /> &nbsp; set log_file_name_convert='/chicago/','/boston/'&nbsp;&nbsp;<br /> &nbsp; set control_files='/&lt;path&gt;/control01.ctl'&nbsp;&nbsp;<br /> &nbsp; set log_archive_max_processes='5'&nbsp;&nbsp;<br /> &nbsp; set fal_client='boston'&nbsp;&nbsp;<br /> &nbsp; set fal_server='chicago'&nbsp;&nbsp;<br /> &nbsp; set standby_file_management='MANUAL'&nbsp;&nbsp;<br /> &nbsp; set log_archive_config='dg_config=(chicago,boston)'&nbsp;&nbsp;<br /> &nbsp; set log_archive_dest_2='service=chicago ASYNC valid_for=(ONLINE_LOGFILE,PRIMARY_ROLE) db_unique_name=chicago'&nbsp;&nbsp;<br /> ;&nbsp;&nbsp;<br /> }&nbsp;&nbsp;<br /> <br /> using target database control file instead of recovery catalog&nbsp;&nbsp;<br /> allocated channel: prmy1&nbsp;&nbsp;<br /> channel prmy1: SID=147 device type=DISK&nbsp;&nbsp;<br /> <br /> allocated channel: prmy2&nbsp;&nbsp;<br /> channel prmy2: SID=130 device type=DISK&nbsp;&nbsp;<br /> <br /> allocated channel: prmy3&nbsp;&nbsp;<br /> channel prmy3: SID=137 device type=DISK&nbsp;&nbsp;<br /> <br /> allocated channel: prmy4&nbsp;&nbsp;<br /> channel prmy4: SID=170 device type=DISK&nbsp;&nbsp;<br /> <br /> allocated channel: stby&nbsp;&nbsp;<br /> channel stby: SID=98 device type=DISK&nbsp;&nbsp;<br /> <br /> Starting Duplicate Db at 19-MAY-08&nbsp;&nbsp;<br /> <br /> contents of Memory Script:&nbsp;&nbsp;<br /> {&nbsp;&nbsp;<br /> backup as copy reuse&nbsp;&nbsp;<br /> file '/&lt;oracle_home path&gt;/dbs/orapwcore' auxiliary format '/&lt;oracle_home path&gt;/dbs/orapwcore1'&nbsp;&nbsp;<br /> file'/&lt;oracle_home path&gt;/dbs/spfilecore.ora' auxiliary format '/&lt;oracle_home path&gt;/dbs/spfilecore1.ora' ;&nbsp;&nbsp;<br /> sql clone "alter system set spfile= ''/&lt;oracle_home path&gt;/dbs/spfilecore1.ora''";&nbsp;&nbsp;<br /> }&nbsp;&nbsp;<br /> executing Memory Script&nbsp;&nbsp;<br /> <br /> Starting backup at 19-MAY-08&nbsp;&nbsp;<br /> Finished backup at 19-MAY-08&nbsp;&nbsp;<br /> <br /> sql statement: alter system set spfile= ''/&lt;oracle_home path&gt;/dbs/spfilecore1.ora''&nbsp;&nbsp;<br /> <br /> contents of Memory Script:&nbsp;&nbsp;<br /> {&nbsp;&nbsp;<br /> sql clone "alter system set audit_file_dest =''/&lt;path&gt;/boston/adump'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set dispatchers =''(PROTOCOL=TCP) (SERVICE=core1XDB)'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set log_archive_dest_2 =''service=core11 arch async VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) db_unique_name=boston'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set db_unique_name =''boston'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set db_file_name_convert =''/chicago/'', ''/boston/'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set log_file_name_convert =''/chicago/'', ''/boston/'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set control_files =''/&lt;path&gt;/control01.ctl'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set log_archive_max_processes =5 comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set fal_client =''boston'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set fal_server =''chicago'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set standby_file_management =''MANUAL'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set log_archive_config =''dg_config=(chicago,boston)'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> sql clone "alter system set log_archive_dest_2 =''service=chicago ASYNC valid_for=(ONLINE_LOGFILE,PRIMARY_ROLE) db_unique_name=chicago'' comment='''' scope=spfile";&nbsp;&nbsp;<br /> shutdown clone immediate;&nbsp;&nbsp;<br /> startup clone nomount ;&nbsp;&nbsp;<br /> }&nbsp;&nbsp;<br /> executing Memory Script&nbsp;&nbsp;<br /> <br /> sql statement: alter system set audit_file_dest = ''/&lt;path&gt;/boston/adump'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set dispatchers = ''(PROTOCOL=TCP) (SERVICE=core1XDB)'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set log_archive_dest_2 = ''service=core11 arch async VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) db_unique_name=boston'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set db_unique_name = ''boston'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set db_file_name_convert = ''/chicago/'', ''/boston/'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set log_file_name_convert = ''/chicago/'', ''/boston/'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set control_files = ''/&lt;path&gt;/control01.ctl'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set log_archive_max_processes = 5 comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set fal_client = ''boston'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set fal_server = ''chicago'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set standby_file_management = ''AUTO'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set log_archive_config = ''dg_config=(chicago,boston)'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> sql statement: alter system set log_archive_dest_2 = ''service=chicago ASYNC valid_for=(ONLINE_LOGFILE,PRIMARY_ROLE) db_unique_name=chicago'' comment= '''' scope=spfile&nbsp;&nbsp;<br /> <br /> Oracle instance shut down&nbsp;&nbsp;<br /> <br /> connected to auxiliary database (not started)&nbsp;&nbsp;<br /> Oracle instance started&nbsp;&nbsp;<br /> <br /> Total System Global Area 845348864 bytes&nbsp;&nbsp;<br /> <br /> Fixed Size 1303188 bytes&nbsp;&nbsp;<br /> Variable Size 482348396 bytes&nbsp;&nbsp;<br /> Database Buffers 356515840 bytes&nbsp;&nbsp;<br /> Redo Buffers 5181440 bytes&nbsp;&nbsp;<br /> <br /> contents of Memory Script:&nbsp;&nbsp;<br /> {&nbsp;&nbsp;<br /> backup as copy current controlfile for standby auxiliary format '/&lt;path&gt;/control01.ctl';&nbsp;&nbsp;<br /> sql clone 'alter database mount standby database';&nbsp;&nbsp;<br /> }&nbsp;&nbsp;<br /> executing Memory Script&nbsp;&nbsp;<br /> <br /> Starting backup at 19-MAY-08&nbsp;&nbsp;<br /> channel prmy1: starting datafile copy&nbsp;&nbsp;<br /> copying standby control file&nbsp;&nbsp;<br /> output file name=/&lt;oracle_home path&gt;/dbs/snapcf_chicago.f tag=TAG20080519T173406 RECID=2 STAMP=655148053&nbsp;&nbsp;<br /> channel prmy1: datafile copy complete, elapsed time: 00:00:03&nbsp;&nbsp;<br /> Finished backup at 19-MAY-08&nbsp;&nbsp;<br /> <br /> sql statement: alter database mount standby database&nbsp;&nbsp;<br /> <br /> contents of Memory Script:&nbsp;&nbsp;<br /> {&nbsp;&nbsp;<br /> set newname for tempfile 1 to"/&lt;path&gt;/boston/temp01.dbf";&nbsp;&nbsp;<br /> switch clone tempfile all;&nbsp;&nbsp;<br /> set newname for datafile 1 to "/&lt;path&gt;/boston/system01.dbf";&nbsp;&nbsp;<br /> set newname for datafile 2 to "/&lt;path&gt;/boston/sysaux01.dbf";&nbsp;&nbsp;<br /> set newname for datafile 3 to "/&lt;path&gt;/boston/undotbs01.dbf";&nbsp;&nbsp;<br /> set newname for datafile 4 to "/&lt;path&gt;/boston/users01.dbf";&nbsp;&nbsp;<br /> backup as copy reuse&nbsp;&nbsp;<br /> datafile 1 auxiliary format "/&lt;path&gt;/boston/system01.dbf"&nbsp;&nbsp;<br /> datafile 2 auxiliary format "/&lt;path&gt;/boston/sysaux01.dbf"&nbsp;&nbsp;<br /> datafile 3 auxiliary format "/&lt;path&gt;/boston/undotbs01.dbf"&nbsp;&nbsp;<br /> datafile 4 auxiliary format "/&lt;path&gt;/boston/users01.dbf" ;&nbsp;&nbsp;<br /> sql 'alter system archive log current';&nbsp;&nbsp;<br /> }&nbsp;&nbsp;<br /> executing Memory Script&nbsp;&nbsp;<br /> <br /> executing command: SET NEWNAME&nbsp;&nbsp;<br /> <br /> renamed tempfile 1 to /&lt;path&gt;/boston/temp01.dbf in control file&nbsp;&nbsp;<br /> <br /> executing command: SET NEWNAME&nbsp;&nbsp;<br /> executing command: SET NEWNAME&nbsp;&nbsp;<br /> executing command: SET NEWNAME&nbsp;&nbsp;<br /> executing command: SET NEWNAME&nbsp;&nbsp;<br /> <br /> Starting backup at 19-MAY-08&nbsp;&nbsp;<br /> channel prmy1: starting datafile copy&nbsp;&nbsp;<br /> input datafile file number=00001 name=/&lt;path&gt;chicago/system01.dbf&nbsp;&nbsp;<br /> channel prmy2: starting datafile copy&nbsp;&nbsp;<br /> input datafile file number=00002 name=/&lt;path&gt;/chicago/sysaux01.dbf&nbsp;&nbsp;<br /> channel prmy3: starting datafile copy&nbsp;&nbsp;<br /> input datafile file number=00003 name=/&lt;path&gt;/chicago/undotbs01.dbf&nbsp;&nbsp;<br /> channel prmy4: starting datafile copy&nbsp;&nbsp;<br /> input datafile file number=00004 name=/&lt;path&gt;/chicago/users01.dbf&nbsp;&nbsp;<br /> output file name=/&lt;path&gt;/boston/undotbs01.dbf tag=TAG20080519T173421 RECID=0 STAMP=0&nbsp;&nbsp;<br /> channel prmy3: datafile copy complete, elapsed time: 00:00:24&nbsp;&nbsp;<br /> output file name=/&lt;path&gt;/boston/users01.dbf tag=TAG20080519T173421 RECID=0 STAMP=0&nbsp;&nbsp;<br /> channel prmy4: datafile copy complete, elapsed time: 00:00:16&nbsp;&nbsp;<br /> output file name=/&lt;path&gt;/boston/system01.dbf tag=TAG20080519T173421 RECID=0 STAMP=0&nbsp;&nbsp;<br /> channel prmy1: datafile copy complete, elapsed time: 00:02:32&nbsp;&nbsp;<br /> output file name=/&lt;path&gt;/boston/sysaux01.dbf tag=TAG20080519T173421 RECID=0 STAMP=0&nbsp;&nbsp;<br /> channel prmy2: datafile copy complete, elapsed time: 00:02:32&nbsp;&nbsp;<br /> Finished backup at 19-MAY-08&nbsp;&nbsp;<br /> <br /> sql statement: alter system archive log current&nbsp;&nbsp;<br /> <br /> contents of Memory Script:&nbsp;&nbsp;<br /> {&nbsp;&nbsp;<br /> switch clone datafile all;&nbsp;&nbsp;<br /> }&nbsp;&nbsp;<br /> executing Memory Script&nbsp;&nbsp;<br /> <br /> datafile 1 switched to datafile copy&nbsp;&nbsp;<br /> input datafile copy RECID=2 STAMP=655148231 file name=/&lt;path&gt;/boston/system01.dbf&nbsp;&nbsp;<br /> datafile 2 switched to datafile copy&nbsp;&nbsp;<br /> input datafile copy RECID=3 STAMP=655148231 file name=/&lt;path&gt;/boston/sysaux01.dbf&nbsp;&nbsp;<br /> datafile 3 switched to datafile copy&nbsp;&nbsp;<br /> input datafile copy RECID=4 STAMP=655148231 file name=/&lt;path&gt;/boston/undotbs01.dbf&nbsp;&nbsp;<br /> datafile 4 switched to datafile copy&nbsp;&nbsp;<br /> input datafile copy RECID=5 STAMP=655148231 file name=/&lt;path&gt;/boston/users01.dbf&nbsp;&nbsp;<br /> Finished Duplicate Db at 19-MAY-08&nbsp;&nbsp;<br /> released channel: prmy1&nbsp;&nbsp;<br /> released channel: prmy2&nbsp;&nbsp;<br /> released channel: prmy3&nbsp;&nbsp;<br /> released channel: prmy4</p>
<ol start="4">
<li><strong> Start managed recovery</strong></li>
</ol>
<p>Connect to standby using SQL*Plus and start the MRP (Managed Recovery Process). Compare the primary last sequence and MRP (Managed Recovery Process) applying sequence.</p>
<p>Example :</p>
<p>SQL&gt; alter database recover managed standby database disconnect from session;</p>
<ol start="5">
<li><strong> Open standby database in Read Only (active dataguard)</strong></li>
</ol>
<p>If you are licensed to use Active Dataguard (ADG) than open the Standby Database in READ ONLY and start the recovery.</p>
<p>SQL&gt; alter database recover managed standby database cancel;&nbsp;&nbsp;<br /> SQL&gt; alter database open;&nbsp;&nbsp;<br /> SQL&gt; alter database recover managed standby database disconnect;</p>
<ol start="6">
<li>DGMGRL,</li>
</ol>
<p>From standby server, Connect to Standby,</p>
<p>DGMGRL&gt;connect sys/&lt;password&gt;@&lt;standby connect string&gt;</p>
<p>If Standby is already in mount mode and MRP is running (NOTE : DG broker automatically start the MRP if standby DB in mount mode)</p>
<p>DGMGRL&gt;edit database &lt;standby&gt; set state=apply-off;</p>
<p>DGMGRL&gt;shutdown</p>
<p>DGMGRL&gt;edit database &lt;standby&gt; set state=apply-on;</p>
<p>DGMGRL&gt;startup&nbsp; &lt;&lt;&lt;&lt;MRP will get started automatically</p>
<p>&nbsp;</p>
<p>&nbsp;Please note if standby_file_management was set to manual. Ensure you set it back to Auto on the standby database once the standby creation is completed</p>
<p><br /> <strong>Attachments :&nbsp;</strong><br /> &nbsp;</p>
<p><strong>References</strong></p>
<p>MOS Document Id: 1075908.1</p>
<p><strong>Internal Only</strong></p>
<p>Created from &lt;SR 3-1430829871&gt;</p>
<p>&nbsp;</p>
