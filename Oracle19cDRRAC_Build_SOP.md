<h1>Oracle 19c DR RAC Build and RAC Conversion SOP</h1>
<h2>Objective</h2>
<p>This document provides a detailed Standard Operating Procedure (SOP) for:</p>
<ol>
<li>
<p>Building the Oracle 19c Disaster Recovery (DR) standby database using RMAN Duplicate from Active Database.</p>
</li>
<li>
<p>Configuring Oracle Grid Infrastructure and RAC on DR.</p>
</li>
<li>
<p>Resolving listener, TNS, VIP, and cluster configuration issues.</p>
</li>
<li>
<p>Adding the second RAC node to the DR cluster.</p>
</li>
<li>
<p>Preparing for conversion of the standby database from single-instance to 2-node RAC.</p>
</li>
<li>
<p>Validating Data Guard synchronization and RAC functionality.</p>
</li>
</ol>
<hr />
<h1>Environment Details</h1>
<h2>Primary Environment (DC)</h2>
<ul>
<li>
<p>Oracle Database Version: 19c</p>
</li>
<li>
<p>Patch Level: 19.27.0.0.0</p>
</li>
<li>
<p>Database Type: RAC</p>
</li>
</ul>
<h2>DR Environment</h2>
<h3>Node 1</h3>
<ul>
<li>
<p>Hostname: erpproddb1-dr.pspcl.in</p>
</li>
<li>
<p>Public IP: 10.61.185.221</p>
</li>
<li>
<p>VIP: 10.61.185.228</p>
</li>
<li>
<p>Private IP: 192.168.1.21</p>
</li>
</ul>
<h3>Node 2</h3>
<ul>
<li>
<p>Hostname: erpproddb2-dr.pspcl.in</p>
</li>
<li>
<p>Public IP: 10.61.185.222</p>
</li>
<li>
<p>VIP: 10.61.185.229</p>
</li>
<li>
<p>Private IP: 192.168.1.22</p>
</li>
</ul>
<h3>Database Details</h3>
<ul>
<li>
<p>DB_NAME: PRODCDB</p>
</li>
<li>
<p>DB_UNIQUE_NAME: PRODCDB_DR</p>
</li>
<li>
<p>Grid Home: /oracle/app/grid/19.3.0/gridhome_1</p>
</li>
<li>
<p>DB Home: /oracle/app/oracle/database/19.3.0/dbhome_1</p>
</li>
</ul>
<hr />
<h1>Phase 1 &ndash; DR Standby Database Build</h1>
<h2>Step 1 &ndash; Prepare Oracle Homes and ASM</h2>
<h3>Verify ASM Disk Groups</h3>
<pre><code class="language-bash">asmcmd lsdg
</code></pre>
<p>Expected Disk Groups:</p>
<ul>
<li>
<p>DATA</p>
</li>
<li>
<p>OCRVOTE</p>
</li>
</ul>
<h3>Verify Grid Infrastructure</h3>
<pre><code class="language-bash">crsctl stat res -t<br /><br />[grid@erpproddb2-dr ~]$ crsctl stat res -t<br />--------------------------------------------------------------------------------<br />Name Target State Server State details<br />--------------------------------------------------------------------------------<br />Local Resources<br />--------------------------------------------------------------------------------<br />ora.LISTENER.lsnr<br /> ONLINE ONLINE erpproddb1-dr STABLE<br /> ONLINE ONLINE erpproddb2-dr STABLE<br />ora.chad<br /> ONLINE ONLINE erpproddb1-dr STABLE<br /> ONLINE ONLINE erpproddb2-dr STABLE<br />ora.net1.network<br /> ONLINE ONLINE erpproddb1-dr STABLE<br /> ONLINE ONLINE erpproddb2-dr STABLE<br />ora.ons<br /> ONLINE ONLINE erpproddb1-dr STABLE<br /> ONLINE ONLINE erpproddb2-dr STABLE<br />--------------------------------------------------------------------------------<br />Cluster Resources<br />--------------------------------------------------------------------------------<br />ora.ASMNET1LSNR_ASM.lsnr(ora.asmgroup)<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br /> 2 ONLINE ONLINE erpproddb2-dr STABLE<br /> 3 ONLINE OFFLINE STABLE<br />ora.DATA.dg(ora.asmgroup)<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br /> 2 ONLINE ONLINE erpproddb2-dr STABLE<br /> 3 OFFLINE OFFLINE STABLE<br />ora.LISTENER_SCAN1.lsnr<br /> 1 ONLINE ONLINE erpproddb2-dr STABLE<br />ora.LISTENER_SCAN2.lsnr<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.LISTENER_SCAN3.lsnr<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.OCRVOTE.dg(ora.asmgroup)<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br /> 2 ONLINE ONLINE erpproddb2-dr STABLE<br /> 3 OFFLINE OFFLINE STABLE<br />ora.asm(ora.asmgroup)<br /> 1 ONLINE ONLINE erpproddb1-dr Started,STABLE<br /> 2 ONLINE ONLINE erpproddb2-dr Started,STABLE<br /> 3 OFFLINE OFFLINE STABLE<br />ora.asmnet1.asmnetwork(ora.asmgroup)<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br /> 2 ONLINE ONLINE erpproddb2-dr STABLE<br /> 3 OFFLINE OFFLINE STABLE<br />ora.cvu<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.erpproddb1-dr.vip<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.erpproddb2-dr.vip<br /> 1 ONLINE ONLINE erpproddb2-dr STABLE<br />ora.prodcdb_dr.db<br /> 1 ONLINE INTERMEDIATE erpproddb1-dr Mounted (Closed),HOM<br /> E=/oracle/app/oracle<br /> /database/19.3.0/dbh<br /> ome_1,STABLE<br />ora.qosmserver<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.scan1.vip<br /> 1 ONLINE ONLINE erpproddb2-dr STABLE<br />ora.scan2.vip<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.scan3.vip<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />--------------------------------------------------------------------------------<br />[grid@erpproddb2-dr ~]$<br />
</code></pre>
<hr />
<h2>Step 2 &ndash; Configure Passwordless SSH Between Nodes</h2>
<h3>Generate SSH Keys</h3>
<p>As grid user:</p>
<pre><code class="language-bash">ssh-keygen -t rsa
</code></pre>
<h3>Copy Keys</h3>
<pre><code class="language-bash">ssh-copy-id erpproddb2-dr
ssh-copy-id erpproddb1-dr
</code></pre>
<h3>Validate SSH</h3>
<pre><code class="language-bash">ssh erpproddb2-dr hostname
ssh erpproddb1-dr hostname
</code></pre>
<h3>Validate CVU User Equivalence</h3>
<pre><code class="language-bash">/oracle/app/grid/19.3.0/gridhome_1/bin/cluvfy comp admprv \
-o user_equiv \
-n erpproddb1-dr,erpproddb2-dr \
-verbose
</code></pre>
<hr />
<h2>Step 3 &ndash; Configure Network and Hosts File</h2>
<h3>Configure /etc/hosts</h3>
<pre><code class="language-bash">10.61.185.221   erpproddb1-dr.pspcl.in  erpproddb1-dr
10.61.185.222   erpproddb2-dr.pspcl.in  erpproddb2-dr

10.61.185.228   erpproddb1-dr-vip.pspcl.in  erpproddb1-dr-vip
10.61.185.229   erpproddb2-dr-vip.pspcl.in  erpproddb2-dr-vip

192.168.1.21    erpproddb1-dr-priv.pspcl.in erpproddb1-dr-priv
192.168.1.22    erpproddb2-dr-priv.pspcl.in erpproddb2-dr-priv
</code></pre>
<h3>Validate Name Resolution</h3>
<pre><code class="language-bash">ping erpproddb2-dr
ping erpproddb2-dr-vip
nslookup 10.61.185.222
</code></pre>
<hr />
<h2>Step 4 &ndash; RMAN Duplicate Standby Database</h2>
<h3>Start Auxiliary Instance in NOMOUNT</h3>
<pre><code class="language-sql">startup nomount;
</code></pre>
<h3>RMAN Duplicate Command</h3>
<pre><code class="language-rman">DUPLICATE TARGET DATABASE
FOR STANDBY
FROM ACTIVE DATABASE
DORECOVER
SPFILE
SET db_unique_name='PRODCDB_DR'
NOFILENAMECHECK;
</code></pre>
<hr />
<h1>Phase 2 &ndash; Data Guard Configuration</h1>
<h2>Step 1 &ndash; Configure Listener and TNS</h2>
<h3>Verify Listener</h3>
<pre><code class="language-bash">lsnrctl status
</code></pre>
<h3>Verify TNS Connectivity</h3>
<pre><code class="language-bash">tnsping PRODCDB_DR
</code></pre>
<h3>Update LOG_ARCHIVE_DEST_2</h3>
<pre><code class="language-sql">ALTER SYSTEM SET LOG_ARCHIVE_DEST_2=
'SERVICE=PRODCDB_DR ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=PRODCDB_DR';
</code></pre>
<hr />
<h2>Step 2 &ndash; Enable Managed Recovery</h2>
<h3>Start Redo Apply</h3>
<pre><code class="language-sql">ALTER DATABASE RECOVER MANAGED STANDBY DATABASE USING CURRENT LOGFILE DISCONNECT;
</code></pre>
<h3>Validate Recovery</h3>
<pre><code class="language-sql">SELECT PROCESS, STATUS, THREAD#, SEQUENCE#
FROM V$MANAGED_STANDBY;
</code></pre>
<h3>Validate Database Role</h3>
<pre><code class="language-sql">SELECT DATABASE_ROLE, OPEN_MODE FROM V$DATABASE;
</code></pre>
<p>Expected:</p>
<ul>
<li>
<p>DATABASE_ROLE = PHYSICAL STANDBY</p>
</li>
<li>
<p>OPEN_MODE = MOUNTED</p>
</li>
</ul>
<hr />
<h1>Phase 3 &ndash; Add Second RAC Node</h1>
<h2>Step 1 &ndash; Validate Cluster Status</h2>
<pre><code class="language-bash">olsnodes -n
crsctl status resource -t
</code></pre>
<p>Initially only node1 was visible.</p>
<hr />
<h2>Step 2 &ndash; Resolve VIP and Duplicate Host Issues</h2>
<h3>Issues Encountered</h3>
<ul>
<li>
<p>INS-41006 Virtual hostnames are not specified</p>
</li>
<li>
<p>INS-40912 Virtual host name assigned to another system</p>
</li>
<li>
<p>INS-40906 Duplicate host name provided</p>
</li>
</ul>
<h3>Root Cause</h3>
<p>VIP addresses were active on node1 and caused Oracle installer validation failures.</p>
<h3>Temporary Removal of Incorrect VIP IPs</h3>
<pre><code class="language-bash">ip addr del 10.61.185.223/24 dev ens192
ip addr del 10.61.185.224/24 dev ens192
ip addr del 10.61.185.225/24 dev ens192
ip addr del 10.61.185.229/24 dev ens192
</code></pre>
<h3>Restart Network</h3>
<pre><code class="language-bash">systemctl restart network
</code></pre>
<h3>Verify IP Cleanup</h3>
<pre><code class="language-bash">ip a
</code></pre>
<hr />
<h2>Step 3 &ndash; Deconfigure Partial CRS Setup on Node2</h2>
<p>As root on node2:</p>
<pre><code class="language-bash">/oracle/app/grid/19.3.0/gridhome_1/crs/install/rootcrs.sh -deconfig -force
</code></pre>
<h3>Cleanup Residual Files</h3>
<pre><code class="language-bash">rm -rf /etc/oracle
rm -rf /var/tmp/.oracle
rm -f /etc/oraInst.loc
</code></pre>
<hr />
<h2>Step 4 &ndash; Fix Inventory Permissions</h2>
<h3>Create Oracle Inventory</h3>
<pre><code class="language-bash">mkdir -p /oracle/app/grid/oraInventory
chown -R grid:oinstall /oracle/app/grid/oraInventory
chmod -R 775 /oracle/app/grid/oraInventory
</code></pre>
<h3>Configure oraInst.loc</h3>
<pre><code class="language-bash">cat &gt; /etc/oraInst.loc &lt;&lt;EOF
inventory_loc=/oracle/app/grid/oraInventory
inst_group=oinstall
EOF
</code></pre>
<hr />
<h2>Step 5 &ndash; Install cvuqdisk on Node2</h2>
<h3>Copy RPM</h3>
<pre><code class="language-bash">scp /oracle/app/grid/19.3.0/gridhome_1/cv/rpm/cvuqdisk-1.0.10-1.rpm root@erpproddb2-dr:/tmp
</code></pre>
<h3>Install RPM</h3>
<pre><code class="language-bash">rpm -ivh /tmp/cvuqdisk-1.0.10-1.rpm
</code></pre>
<h3>Verify</h3>
<pre><code class="language-bash">rpm -qa | grep cvuqdisk
</code></pre>
<hr />
<h2>Step 6 &ndash; Add Node Using Grid Setup GUI</h2>
<h3>Launch Installer</h3>
<pre><code class="language-bash">/oracle/app/grid/19.3.0/gridhome_1/gridSetup.sh
</code></pre>
<h3>Select</h3>
<ul>
<li>
<p>Add more nodes to existing cluster</p>
</li>
</ul>
<h3>Provide</h3>
<ul>
<li>
<p>Public hostname</p>
</li>
<li>
<p>VIP hostname</p>
</li>
<li>
<p>Private hostname</p>
</li>
</ul>
<h3>Execute root.sh on node2</h3>
<pre><code class="language-bash">/oracle/app/grid/19.3.0/gridhome_1/root.sh
</code></pre>
<h3>Successful Completion Message</h3>
<pre><code class="language-text">CLSRSC-325: Configure Oracle Grid Infrastructure for a Cluster ... succeeded
</code></pre>
<hr />
<h2>Step 7 &ndash; Validate Cluster After Addnode</h2>
<h3>Verify Nodes</h3>
<pre><code class="language-bash">olsnodes -n
</code></pre>
<p>[grid@erpproddb2-dr ~]$ olsnodes -n<br />erpproddb1-dr 1<br />erpproddb2-dr 2</p>
<h3>Verify Cluster Resources</h3>
<pre><code class="language-bash">crsctl status resource -t<br /><br />[grid@erpproddb2-dr ~]$ crsctl status resource -t<br />--------------------------------------------------------------------------------<br />Name Target State Server State details<br />--------------------------------------------------------------------------------<br />Local Resources<br />--------------------------------------------------------------------------------<br />ora.LISTENER.lsnr<br /> ONLINE ONLINE erpproddb1-dr STABLE<br /> ONLINE ONLINE erpproddb2-dr STABLE<br />ora.chad<br /> ONLINE ONLINE erpproddb1-dr STABLE<br /> ONLINE ONLINE erpproddb2-dr STABLE<br />ora.net1.network<br /> ONLINE ONLINE erpproddb1-dr STABLE<br /> ONLINE ONLINE erpproddb2-dr STABLE<br />ora.ons<br /> ONLINE ONLINE erpproddb1-dr STABLE<br /> ONLINE ONLINE erpproddb2-dr STABLE<br />--------------------------------------------------------------------------------<br />Cluster Resources<br />--------------------------------------------------------------------------------<br />ora.ASMNET1LSNR_ASM.lsnr(ora.asmgroup)<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br /> 2 ONLINE ONLINE erpproddb2-dr STABLE<br /> 3 ONLINE OFFLINE STABLE<br />ora.DATA.dg(ora.asmgroup)<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br /> 2 ONLINE ONLINE erpproddb2-dr STABLE<br /> 3 OFFLINE OFFLINE STABLE<br />ora.LISTENER_SCAN1.lsnr<br /> 1 ONLINE ONLINE erpproddb2-dr STABLE<br />ora.LISTENER_SCAN2.lsnr<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.LISTENER_SCAN3.lsnr<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.OCRVOTE.dg(ora.asmgroup)<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br /> 2 ONLINE ONLINE erpproddb2-dr STABLE<br /> 3 OFFLINE OFFLINE STABLE<br />ora.asm(ora.asmgroup)<br /> 1 ONLINE ONLINE erpproddb1-dr Started,STABLE<br /> 2 ONLINE ONLINE erpproddb2-dr Started,STABLE<br /> 3 OFFLINE OFFLINE STABLE<br />ora.asmnet1.asmnetwork(ora.asmgroup)<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br /> 2 ONLINE ONLINE erpproddb2-dr STABLE<br /> 3 OFFLINE OFFLINE STABLE<br />ora.cvu<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.erpproddb1-dr.vip<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.erpproddb2-dr.vip<br /> 1 ONLINE ONLINE erpproddb2-dr STABLE<br />ora.prodcdb_dr.db<br /> 1 ONLINE INTERMEDIATE erpproddb1-dr Mounted (Closed),HOM<br /> E=/oracle/app/oracle<br /> /database/19.3.0/dbh<br /> ome_1,STABLE<br />ora.qosmserver<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.scan1.vip<br /> 1 ONLINE ONLINE erpproddb2-dr STABLE<br />ora.scan2.vip<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />ora.scan3.vip<br /> 1 ONLINE ONLINE erpproddb1-dr STABLE<br />--------------------------------------------------------------------------------<br />[grid@erpproddb2-dr ~]$<br />
</code></pre>
<p>Verify:</p>
<ul>
<li>
<p>VIPs online</p>
</li>
<li>
<p>ASM online on both nodes</p>
</li>
<li>
<p>LISTENER online on both nodes</p>
</li>
<li>
<p>OCRVOTE mounted</p>
</li>
<li>
<p>DATA mounted</p>
</li>
</ul>
<hr />
<h1>Phase 4 &ndash; Oracle 19c RU Patching</h1>
<h2>Objective</h2>
<p>Patch DR database Oracle Home to match Primary patch level.</p>
<h3>Patch Applied</h3>
<ul>
<li>
<p>37642901 &ndash; Database Release Update 19.27.0.0.0</p>
</li>
<li>
<p>OPatch Version: 12.2.0.1.46</p>
</li>
</ul>
<hr />
<h2>Step 1 &ndash; Verify OPatch Version</h2>
<pre><code class="language-bash">$ORACLE_HOME/OPatch/opatch version
</code></pre>
<hr />
<h2>Step 2 &ndash; Stop Database and Listener</h2>
<h3>Stop Listener on Node2</h3>
<pre><code class="language-bash">srvctl stop listener -node erpproddb2-dr
</code></pre>
<h3>Verify</h3>
<pre><code class="language-bash">srvctl status listener
</code></pre>
<hr />
<h2>Step 3 &ndash; Apply Patch on Node1</h2>
<pre><code class="language-bash">cd 37642901
opatch apply
</code></pre>
<h3>Verify</h3>
<pre><code class="language-bash">opatch lspatches
</code></pre>
<hr />
<h2>Step 4 &ndash; Resolve Node2 Inventory Issue</h2>
<h3>Problem</h3>
<pre><code class="language-text">Inventory load failed
RawInventory gets null OracleHomeInfo
</code></pre>
<h3>Root Cause</h3>
<p>Incorrect oraInst.loc pointing to Grid inventory.</p>
<h3>Fix</h3>
<p>As root:</p>
<pre><code class="language-bash">cat &gt; /etc/oraInst.loc &lt;&lt;EOF
inventory_loc=/oracle/app/oraInventory
inst_group=oinstall
EOF
</code></pre>
<h3>Create Inventory</h3>
<pre><code class="language-bash">mkdir -p /oracle/app/oraInventory
chown -R oracle:oinstall /oracle/app/oraInventory
chmod -R 775 /oracle/app/oraInventory
</code></pre>
<h3>Attach DB Home</h3>
<p>As oracle user:</p>
<pre><code class="language-bash">cd $ORACLE_HOME/oui/bin
./attachHome.sh
</code></pre>
<h3>Verify</h3>
<pre><code class="language-bash">opatch lsinventory
opatch lspatches
</code></pre>
<hr />
<h2>Step 5 &ndash; Apply Patch on Node2</h2>
<pre><code class="language-bash">cd 37642901
opatch apply
</code></pre>
<h3>Successful Output</h3>
<pre><code class="language-text">Patch 37642901 successfully applied.
OPatch succeeded.
</code></pre>
<hr />
<h2>&nbsp;</h2>
<hr />
<h2>Step 7 &ndash; Verify Database Version</h2>
<pre><code class="language-sql">select banner_full from v$version;
</code></pre>
<p><code class="language-text">19.27.0.0.0 </code></p>
<h3>Verify SQL Patch Registry</h3>
<pre><code class="language-sql">select patch_id,status,action,action_time
from dba_registry_sqlpatch
order by action_time;
</code></pre>
<hr />
<h1>Phase 5 &ndash; Oracle EBS Rapid Clone Notes</h1>
<h2>Important SID Clarification</h2>
<p>Rapid Clone accepts only SID values of maximum 8 characters.</p>
<h3>Correct Values</h3>
<table>
<thead>
<tr>
<th>Parameter</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>DB_NAME</td>
<td>PRODCDB</td>
</tr>
<tr>
<td>DB_UNIQUE_NAME</td>
<td>PRODCDB_DR</td>
</tr>
</tbody>
</table>
<h3>During adcfgclone</h3>
<p>Use:</p>
<pre><code class="language-text">PRODCDB
</code></pre>
<p>Do NOT use:</p>
<pre><code class="language-text">PRODCDB_DR
</code></pre>
<hr />
<h1>Upcoming Activity &ndash; Convert Standby to 2-Node RAC Database</h1>
<h1>Phase 6 &ndash; RAC Conversion Plan</h1>
<h2>Objective</h2>
<p>Convert the current single-instance standby database into a 2-node RAC standby database.</p>
<hr />
<h2>Step 1 &ndash; Add Standby Redo Logs for Additional Thread</h2>
<h3>Check Existing Threads</h3>
<pre><code class="language-sql">select thread#, status, enabled from v$thread;
</code></pre>
<h3>Add Redo Thread for Node2</h3>
<pre><code class="language-sql">ALTER DATABASE ADD LOGFILE THREAD 2
GROUP 11 ('+DATA') SIZE 1G,
GROUP 12 ('+DATA') SIZE 1G,
GROUP 13 ('+DATA') SIZE 1G;
</code></pre>
<h3>Enable Thread</h3>
<pre><code class="language-sql">ALTER DATABASE ENABLE PUBLIC THREAD 2;
</code></pre>
<hr />
<h2>Step 2 &ndash; Create Undo Tablespace for Instance 2</h2>
<pre><code class="language-sql">CREATE UNDO TABLESPACE UNDOTBS2
DATAFILE '+DATA'
SIZE 10G AUTOEXTEND ON;
</code></pre>
<hr />
<h2>Step 3 &ndash; Configure RAC Parameters</h2>
<h3>Configure Instance Number</h3>
<pre><code class="language-sql">ALTER SYSTEM SET instance_number=2 SID='PRODCDB2' SCOPE=SPFILE;
</code></pre>
<h3>Configure Thread</h3>
<pre><code class="language-sql">ALTER SYSTEM SET thread=2 SID='PRODCDB2' SCOPE=SPFILE;
</code></pre>
<h3>Configure Undo</h3>
<pre><code class="language-sql">ALTER SYSTEM SET undo_tablespace='UNDOTBS2' SID='PRODCDB2' SCOPE=SPFILE;
</code></pre>
<hr />
<h2>Step 4 &ndash; Add Database Instance in RAC</h2>
<p>As grid user:</p>
<pre><code class="language-bash">srvctl add instance \
-db PRODCDB_DR \
-instance PRODCDB2 \
-node erpproddb2-dr
</code></pre>
<hr />
<h2>Step 5 &ndash; Start Instance 2</h2>
<pre><code class="language-bash">srvctl start instance -db PRODCDB_DR -instance PRODCDB2
</code></pre>
<hr />
<h2>Step 6 &ndash; Verify RAC Status</h2>
<pre><code class="language-bash">srvctl status database -db PRODCDB_DR
</code></pre>
<p>Expected:</p>
<pre><code class="language-text">Instance PRODCDB1 is running on node erpproddb1-dr
Instance PRODCDB2 is running on node erpproddb2-dr
</code></pre>
<hr />
<h2>Step 7 &ndash; Validate Data Guard Synchronization</h2>
<h3>Verify Managed Recovery</h3>
<pre><code class="language-sql">SELECT PROCESS, STATUS, THREAD#, SEQUENCE#
FROM V$MANAGED_STANDBY;
</code></pre>
<h3>Verify Archive Gap</h3>
<pre><code class="language-sql">SELECT * FROM V$ARCHIVE_GAP;
</code></pre>
<h3>Verify Data Guard Stats</h3>
<pre><code class="language-sql">SELECT NAME, VALUE, UNIT
FROM V$DATAGUARD_STATS;
</code></pre>
<hr />
<h1>Troubleshooting Reference</h1>
<h2>INS-41006</h2>
<p>Virtual hostnames not specified for all nodes.</p>
<h3>Resolution</h3>
<p>Specify VIP hostname for all nodes.</p>
<hr />
<h2>INS-40912</h2>
<p>Virtual hostname already assigned.</p>
<h3>Resolution</h3>
<p>Ensure VIP IP is not active on any interface.</p>
<hr />
<h2>INS-40906</h2>
<p>Duplicate hostnames provided.</p>
<h3>Resolution</h3>
<p>Clean partial CRS configuration and validate /etc/hosts.</p>
<hr />
<h2>INS-06006</h2>
<p>Passwordless SSH connectivity not set up.</p>
<h3>Resolution</h3>
<p>Validate SSH equivalence using cluvfy.</p>
<hr />
<h2>ORA-12547</h2>
<p>TNS:lost contact.</p>
<h3>Resolution</h3>
<p>Stop conflicting listener and validate Oracle binary ownership.</p>
<hr />
<h1>Validation Checklist</h1>
<table>
<thead>
<tr>
<th>Validation</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td>ASM Diskgroups Mounted</td>
<td>Completed</td>
</tr>
<tr>
<td>Standby Database Created</td>
<td>Completed</td>
</tr>
<tr>
<td>Redo Apply Working</td>
<td>Completed</td>
</tr>
<tr>
<td>Data Guard Sync Verified</td>
<td>Completed</td>
</tr>
<tr>
<td>RAC Node Added</td>
<td>Completed</td>
</tr>
<tr>
<td>Grid Infrastructure Healthy</td>
<td>Completed</td>
</tr>
<tr>
<td>Oracle RU 19.27 Applied</td>
<td>Completed</td>
</tr>
<tr>
<td>Datapatch Completed</td>
<td>Completed</td>
</tr>
<tr>
<td>Node2 RAC Instance Creation</td>
<td>Pending</td>
</tr>
<tr>
<td>RAC Data Guard Validation</td>
<td>Pending</td>
</tr>
</tbody>
</table>
<hr />
<h1>Final Status</h1>
<p>The DR environment has been successfully established with:</p>
<ul>
<li>
<p>Oracle 19c RAC Infrastructure</p>
</li>
<li>
<p>Standby Database configured using RMAN duplicate</p>
</li>
<li>
<p>Data Guard redo transport operational</p>
</li>
<li>
<p>Oracle 19.27 RU patch level aligned with primary site</p>
</li>
<li>
<p>RAC node2 successfully added to cluster</p>
</li>
<li>
<p>Environment ready for RAC standby database expansion</p>
</li>
</ul>
<p>Further activity pending:</p>
<ul>
<li>
<p>Add second RAC database instance</p>
</li>
<li>
<p>Configure redo threads and undo tablespace</p>
</li>
<li>
<p>Validate RAC-based standby recovery</p>
</li>
</ul>
<!-- Comments are visible in the HTML source only -->
