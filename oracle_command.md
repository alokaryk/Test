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

<p>[oracle@erpproddb1-dr ~]$ export ORACLE_SID=PRODCDB_DR<br />[oracle@erpproddb1-dr ~]$ echo $ORACLE_SID<br />PRODCDB_DR<br />[oracle@erpproddb1-dr ~]$ sqlplus / as sysdba</p>
<p>SQL*Plus: Release 19.0.0.0.0 - Production on Fri Mar 13 11:55:14 2026<br />Version 19.27.0.0.0</p>
<p>Copyright (c) 1982, 2024, Oracle. All rights reserved.</p>
<p><br />Connected to:<br />Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production<br />Version 19.27.0.0.0</p>
<p>SQL&gt; SELECT name,open_mode,database_role FROM v$database;</p>
<p>NAME OPEN_MODE DATABASE_ROLE<br />--------- -------------------- ----------------<br />PRODCDB MOUNTED PHYSICAL STANDBY</p>
<p>SQL&gt; SELECT process,status,thread#,sequence#<br />FROM v$managed_standby; 2</p>
<p>PROCESS STATUS THREAD# SEQUENCE#<br />--------- ------------ ---------- ----------<br />ARCH CONNECTED 0 0<br />DGRD ALLOCATED 0 0<br />DGRD ALLOCATED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />RFS IDLE 1 0<br />RFS IDLE 1 14859<br />RFS IDLE 1 0<br />RFS IDLE 2 12380</p>
<p>10 rows selected.</p>
<p>SQL&gt;</p>
<p data-start="0" data-end="118">Your standby database is <strong data-start="25" data-end="46">mounted correctly</strong>, but from the output we can see that <strong data-start="84" data-end="117">redo apply is NOT running yet</strong>.</p>
<h3 data-section-id="1l26sq0" data-start="120" data-end="150">What your output indicates</h3>
<p data-start="152" data-end="157">From:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> process,status,thread#,sequence# <span class="ͼg">FROM</span> v$managed_standby;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="235" data-end="248">You only see:</p>
<ul data-start="250" data-end="346">
<li data-section-id="13p0e6q" data-start="250" data-end="284">
<p data-start="252" data-end="284"><code data-start="252" data-end="257">RFS</code> processes &rarr; Receiving redo</p>
</li>
<li data-section-id="118fkr6" data-start="285" data-end="322">
<p data-start="287" data-end="322"><code data-start="287" data-end="293">ARCH</code> processes &rarr; Archive handling</p>
</li>
<li data-section-id="1xiw6kd" data-start="323" data-end="346">
<p data-start="325" data-end="346"><strong data-start="325" data-end="346">No <code data-start="330" data-end="336">MRP0</code> process</strong></p>
</li>
</ul>
<p data-start="348" data-end="356">Example:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">RFS IDLE 1 14859<br />RFS IDLE 2 12380</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="401" data-end="412">This means:</p>
<ul data-start="414" data-end="568">
<li data-section-id="9tj2v6" data-start="414" data-end="448">
<p data-start="416" data-end="448">Standby is <strong data-start="427" data-end="448">mounted correctly</strong></p>
</li>
<li data-section-id="x0n7zv" data-start="449" data-end="481">
<p data-start="451" data-end="481">Redo transport may be arriving</p>
</li>
<li data-section-id="qjlzhf" data-start="482" data-end="568">
<p data-start="484" data-end="568"><strong data-start="484" data-end="533">Managed Recovery Process (MRP) is not started</strong>, so logs are <strong data-start="547" data-end="568">not being applied</strong></p>
</li>
</ul>
<hr data-start="570" data-end="573" />
<h1 data-section-id="38btle" data-start="575" data-end="616">Start Data Guard Apply (Important Step)</h1>
<p data-start="618" data-end="644">Run the following command:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">ALTER</span> DATABASE RECOVER MANAGED STANDBY DATABASE <span class="ͼg">USING</span> <span class="ͼg">CURRENT</span> LOGFILE <span class="ͼg">DISCONNECT</span> <span class="ͼg">FROM</span> <span class="ͼg">SESSION</span>;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="753" data-end="794">This will start <strong data-start="769" data-end="793">real-time redo apply</strong>.</p>
<hr data-start="796" data-end="799" />
<h1 data-section-id="nhp5fa" data-start="801" data-end="833">Verify After Starting Recovery</h1>
<p data-start="835" data-end="845">Run again:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> process,status,thread#,sequence#<br /><span class="ͼg">FROM</span> v$managed_standby;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="923" data-end="957">Now you should see something like:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">PROCESS STATUS THREAD# SEQUENCE#<br />------ ------------- ------- --------<br />MRP0 APPLYING_LOG 1 14859<br />RFS RECEIVING 1 14860<br />RFS RECEIVING 2 12381</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1158" data-end="1180">The important line is:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">MRP0 APPLYING_LOG</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1209" data-end="1253">This confirms <strong data-start="1223" data-end="1252">Data Guard sync is active</strong>.</p>
<hr data-start="1255" data-end="1258" />
<h1 data-section-id="wlj9i3" data-start="1260" data-end="1279">Check DR Sync Lag</h1>
<p data-start="1281" data-end="1285">Run:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> name,<span class="ͼg">value</span> <span class="ͼg">FROM</span> v$dataguard_stats<br /><span class="ͼg">WHERE</span> name <span class="ͼg">LIKE</span> <span class="ͼk">'%lag%'</span>;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1365" data-end="1373">Example:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">transport lag +00 00:01:10<br />apply lag +00 00:00:40</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1442" data-end="1484">Lag will start <strong data-start="1457" data-end="1483">reducing automatically</strong>.</p>
<hr data-start="1486" data-end="1489" />
<h1 data-section-id="1wxuanh" data-start="1491" data-end="1525">Quick Health Check (Recommended)</h1>
<p data-start="1527" data-end="1531">Run:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> thread#, MAX(sequence#)<br /><span class="ͼg">FROM</span> v$archived_log<br /><span class="ͼg">GROUP</span> <span class="ͼg">BY</span> thread#;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1614" data-end="1702">Compare this with <strong data-start="1632" data-end="1673">primary database archive log sequence</strong> to ensure DR is catching up.</p>
<hr data-start="1704" data-end="1707" />
<h1 data-section-id="1vty0gd" data-start="1709" data-end="1728">Current DR Status</h1>
<div class="TyagGW_tableContainer">
<div class="group TyagGW_tableWrapper flex flex-col-reverse w-fit" tabindex="-1">
<table class="w-fit min-w-(--thread-content-width)" data-start="1730" data-end="1935">
<thead data-start="1730" data-end="1752">
<tr data-start="1730" data-end="1752">
<th class="" data-start="1730" data-end="1742" data-col-size="sm">Component</th>
<th class="" data-start="1742" data-end="1752" data-col-size="sm">Status</th>
</tr>
</thead>
<tbody data-start="1763" data-end="1935">
<tr data-start="1763" data-end="1798">
<td data-start="1763" data-end="1785" data-col-size="sm">Grid Infrastructure</td>
<td data-start="1785" data-end="1798" data-col-size="sm">✅ Running</td>
</tr>
<tr data-start="1799" data-end="1818">
<td data-start="1799" data-end="1805" data-col-size="sm">ASM</td>
<td data-start="1805" data-end="1818" data-col-size="sm">✅ Running</td>
</tr>
<tr data-start="1819" data-end="1845">
<td data-start="1819" data-end="1832" data-col-size="sm">Diskgroups</td>
<td data-start="1832" data-end="1845" data-col-size="sm">✅ Mounted</td>
</tr>
<tr data-start="1846" data-end="1872">
<td data-start="1846" data-end="1859" data-col-size="sm">Standby DB</td>
<td data-start="1859" data-end="1872" data-col-size="sm">✅ Mounted</td>
</tr>
<tr data-start="1873" data-end="1903">
<td data-start="1873" data-end="1886" data-col-size="sm">Redo Apply</td>
<td data-start="1886" data-end="1903" data-col-size="sm">❌ Not Running</td>
</tr>
<tr data-start="1904" data-end="1935">
<td data-start="1904" data-end="1922" data-col-size="sm">Required Action</td>
<td data-start="1922" data-end="1935" data-col-size="sm">Start MRP</td>
</tr>
</tbody>
</table>
</div>
</div>
<hr data-start="1937" data-end="1940" />
<h1 data-section-id="4oddra" data-start="1942" data-end="1969">One Important Observation</h1>
<p data-start="1971" data-end="1986">Your RFS shows:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">Thread 1 &rarr; 14859<br />Thread 2 &rarr; 12380</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="2031" data-end="2102">This indicates <strong data-start="2046" data-end="2083">RAC primary with two redo threads</strong>, which is correct.</p>
<hr data-start="2104" data-end="2107" />
<p data-start="2109" data-end="2132">✅ <strong data-start="2111" data-end="2125">Next step:</strong><br data-start="2125" data-end="2128" /> Run:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">ALTER</span> DATABASE RECOVER MANAGED STANDBY DATABASE <span class="ͼg">USING</span> <span class="ͼg">CURRENT</span> LOGFILE <span class="ͼg">DISCONNECT</span> <span class="ͼg">FROM</span> <span class="ͼg">SESSION</span>;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="2241" data-end="2282">and then check <code data-start="2256" data-end="2275">v$managed_standby</code> again.</p>
<p data-start="2241" data-end="2282">&nbsp;</p>
<p>SQL&gt; ALTER DATABASE RECOVER MANAGED STANDBY DATABASE USING CURRENT LOGFILE DISCONNECT FROM SESSION;</p>
<p>Database altered.</p>
<p>SQL&gt; SELECT process,status,thread#,sequence#<br />FROM v$managed_standby; 2</p>
<p>PROCESS STATUS THREAD# SEQUENCE#<br />--------- ------------ ---------- ----------<br />ARCH CONNECTED 0 0<br />DGRD ALLOCATED 0 0<br />DGRD ALLOCATED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />RFS IDLE 1 0<br />RFS IDLE 1 14859<br />RFS IDLE 1 0<br />RFS IDLE 2 12380<br />MRP0 WAIT_FOR_GAP 2 12376</p>
<p>11 rows selected.</p>
<p>SQL&gt;</p>
<p>&nbsp;</p>
<p data-start="0" data-end="88">Your standby recovery <strong data-start="22" data-end="46">started successfully</strong>, but the status now shows a <strong data-start="75" data-end="87">redo gap</strong>.</p>
<h3 data-section-id="v4slsh" data-start="90" data-end="123">Current Status Interpretation</h3>
<p data-start="125" data-end="142">From your output:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">MRP0 WAIT_FOR_GAP 2 12376</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="188" data-end="196">Meaning:</p>
<ul data-start="198" data-end="365">
<li data-section-id="1qlvguo" data-start="198" data-end="250">
<p data-start="200" data-end="250"><strong data-start="200" data-end="208">MRP0</strong> &rarr; Managed Recovery Process is running ✅</p>
</li>
<li data-section-id="18daed5" data-start="251" data-end="316">
<p data-start="253" data-end="316"><strong data-start="253" data-end="269">WAIT_FOR_GAP</strong> &rarr; Some archive logs are <strong data-start="294" data-end="316">missing on standby</strong></p>
</li>
<li data-section-id="mp80w9" data-start="317" data-end="365">
<p data-start="319" data-end="365">Apply cannot continue until those logs arrive.</p>
</li>
</ul>
<p data-start="367" data-end="389">Your RFS entries show:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">Thread 1 last received : 14859<br />Thread 2 last received : 12380</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="462" data-end="485">But MRP is waiting for:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">Thread 2 sequence 12376</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="520" data-end="589">This means <strong data-start="531" data-end="588">some archive logs between sequences are missing on DR</strong>.</p>
<hr data-start="591" data-end="594" />
<h1 data-section-id="n1vuyl" data-start="596" data-end="640">Step 1 &mdash; Identify the Missing Archive Logs</h1>
<p data-start="642" data-end="661">Run this on <strong data-start="654" data-end="660">DR</strong>:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> thread#, low_sequence#, high_sequence#<br /><span class="ͼg">FROM</span> v$archive_gap;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="741" data-end="795">This will show exactly which archive logs are missing.</p>
<p data-start="797" data-end="812">Example output:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">THREAD# LOW_SEQUENCE# HIGH_SEQUENCE#<br />2 12376 12379</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<hr data-start="888" data-end="891" />
<h1 data-section-id="1jeyt1w" data-start="893" data-end="939">Step 2 &mdash; Check if Logs Exist on Primary (DC)</h1>
<p data-start="941" data-end="979">Login to <strong data-start="950" data-end="970">Primary Database</strong> and run:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> thread#, sequence#, archived<br /><span class="ͼg">FROM</span> v$archived_log<br /><span class="ͼg">WHERE</span> thread#<span class="ͼg">=</span><span class="ͼj">2</span><br /><span class="ͼg">AND</span> sequence# <span class="ͼg">BETWEEN</span> <span class="ͼj">12376</span> <span class="ͼg">AND</span> <span class="ͼj">12380</span>;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1104" data-end="1159">If they exist, Data Guard can fetch them automatically.</p>
<hr data-start="1161" data-end="1164" />
<h1 data-section-id="1of2jds" data-start="1166" data-end="1216">Step 3 &mdash; Force Archive Log Shipping from Primary</h1>
<p data-start="1218" data-end="1237">On <strong data-start="1221" data-end="1232">Primary</strong> run:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">ALTER</span> SYSTEM ARCHIVE LOG <span class="ͼg">CURRENT</span>;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1285" data-end="1306">Run it <strong data-start="1292" data-end="1305">2&ndash;3 times</strong>.</p>
<p data-start="1308" data-end="1345">This forces redo shipping to standby.</p>
<hr data-start="1347" data-end="1350" />
<h1 data-section-id="wcfxzv" data-start="1352" data-end="1384">Step 4 &mdash; Check if Gap Resolved</h1>
<p data-start="1386" data-end="1402">Run again on DR:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> process,status,thread#,sequence#<br /><span class="ͼg">FROM</span> v$managed_standby;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1480" data-end="1489">Expected:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">MRP0 APPLYING_LOG</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1518" data-end="1529">Instead of:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">WAIT_FOR_GAP</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<hr data-start="1553" data-end="1556" />
<h1 data-section-id="z80yiq" data-start="1558" data-end="1588">Step 5 &mdash; If Gap Still Exists</h1>
<p data-start="1590" data-end="1645">You may need to <strong data-start="1606" data-end="1644">manually copy missing archive logs</strong>.</p>
<ol data-start="1647" data-end="1683">
<li data-section-id="bgksk7" data-start="1647" data-end="1683">
<p data-start="1650" data-end="1683">Find archive location on primary:</p>
</li>
</ol>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">archive log list;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<ol start="2" data-start="1715" data-end="1736">
<li data-section-id="vvius5" data-start="1715" data-end="1736">
<p data-start="1718" data-end="1736">Copy missing logs:</p>
</li>
</ol>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">scp arch_2_12376*.arc oracle@DR_SERVER:/archive_location/</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<ol start="3" data-start="1809" data-end="1832">
<li data-section-id="199wdrf" data-start="1809" data-end="1832">
<p data-start="1812" data-end="1832">Register logs on DR:</p>
</li>
</ol>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">ALTER</span> DATABASE REGISTER LOGFILE <span class="ͼk">'/path/arch_2_12376.arc'</span>;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1904" data-end="1942">Then recovery continues automatically.</p>
<hr data-start="1944" data-end="1947" />
<h1 data-section-id="1wsarfi" data-start="1949" data-end="1981">Step 6 &mdash; Verify Data Guard Lag</h1>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> name,<span class="ͼg">value</span> <span class="ͼg">FROM</span> v$dataguard_stats<br /><span class="ͼg">WHERE</span> name <span class="ͼg">LIKE</span> <span class="ͼk">'%lag%'</span>;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="2061" data-end="2087">Lag should begin reducing.</p>
<hr data-start="2089" data-end="2092" />
<h1 data-section-id="1fytbbj" data-start="2094" data-end="2109">Quick Summary</h1>
<div class="TyagGW_tableContainer">
<div class="group TyagGW_tableWrapper flex flex-col-reverse w-fit" tabindex="-1">
<table class="w-fit min-w-(--thread-content-width)" data-start="2111" data-end="2273">
<thead data-start="2111" data-end="2133">
<tr data-start="2111" data-end="2133">
<th class="" data-start="2111" data-end="2123" data-col-size="sm">Component</th>
<th class="" data-start="2123" data-end="2133" data-col-size="sm">Status</th>
</tr>
</thead>
<tbody data-start="2144" data-end="2273">
<tr data-start="2144" data-end="2172">
<td data-start="2144" data-end="2161" data-col-size="sm">Standby Database</td>
<td data-start="2161" data-end="2172" data-col-size="sm">Mounted</td>
</tr>
<tr data-start="2173" data-end="2188">
<td data-start="2173" data-end="2177" data-col-size="sm">MRP</td>
<td data-start="2177" data-end="2188" data-col-size="sm">Running</td>
</tr>
<tr data-start="2189" data-end="2211">
<td data-start="2189" data-end="2200" data-col-size="sm">Redo Apply</td>
<td data-start="2200" data-end="2211" data-col-size="sm">Waiting</td>
</tr>
<tr data-start="2212" data-end="2233">
<td data-start="2212" data-end="2218" data-col-size="sm">Issue</td>
<td data-start="2218" data-end="2233" data-col-size="sm">Archive Gap</td>
</tr>
<tr data-start="2234" data-end="2273">
<td data-start="2234" data-end="2241" data-col-size="sm">Action</td>
<td data-start="2241" data-end="2273" data-col-size="sm">Resolve missing archive logs</td>
</tr>
</tbody>
</table>
</div>
</div>
<hr data-start="2275" data-end="2278" />
<p data-start="2280" data-end="2434">💡 <strong data-start="2283" data-end="2302">Very important:</strong><br data-start="2302" data-end="2305" /> Since your DR node rebooted earlier, <strong data-start="2342" data-end="2402">some archive logs might have been missed during downtime</strong>, which is why the gap occurred.</p>
<p data-start="2280" data-end="2434">&nbsp;</p>
<p>on DR SQL&gt; SELECT process,status,thread#,sequence#<br />FROM v$managed_standby; 2</p>
<p>PROCESS STATUS THREAD# SEQUENCE#<br />--------- ------------ ---------- ----------<br />ARCH CONNECTED 0 0<br />DGRD ALLOCATED 0 0<br />DGRD ALLOCATED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />RFS IDLE 1 0<br />RFS IDLE 1 14859<br />RFS IDLE 1 0<br />RFS IDLE 2 12380<br />MRP0 WAIT_FOR_GAP 2 12376</p>
<p>11 rows selected.</p>
<p>SQL&gt; SELECT thread#, low_sequence#, high_sequence#<br />FROM v$archive_gap; 2</p>
<p>no rows selected</p>
<p>SQL&gt; on primary DC sqlplus sys/sys123@PRODCDB_PROD as sysdba</p>
<p>SQL*Plus: Release 19.0.0.0.0 - Production on Fri Mar 13 12:00:14 2026<br />Version 19.27.0.0.0</p>
<p>Copyright (c) 1982, 2024, Oracle. All rights reserved.</p>
<p><br />Connected to:<br />Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production<br />Version 19.27.0.0.0</p>
<p>SQL&gt; SELECT thread#, sequence#, archived<br />FROM v$archived_log<br />WHERE thread#=2<br />AND sequence# BETWEEN 12376 AND 12380; 2 3 4</p>
<p>THREAD# SEQUENCE# ARC<br />---------- ---------- ---<br /> 2 12376 YES<br /> 2 12377 YES<br /> 2 12378 YES<br /> 2 12379 YES</p>
<p>SQL&gt;</p>
<p>&nbsp;</p>
<p data-start="0" data-end="128">Your situation is <strong data-start="18" data-end="42">very close to normal</strong>, but there is a small <strong data-start="65" data-end="88">redo shipping issue</strong>. Let&rsquo;s interpret the results carefully.</p>
<hr data-start="130" data-end="133" />
<h1 data-section-id="1pyxs3t" data-start="135" data-end="154">Current Situation</h1>
<h3 data-section-id="6z80i9" data-start="156" data-end="165">On DR</h3>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">MRP0 WAIT_FOR_GAP 2 12376</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="201" data-end="209">Meaning:</p>
<ul data-start="211" data-end="295">
<li data-section-id="zjarpy" data-start="211" data-end="242">
<p data-start="213" data-end="242">Managed Recovery is running ✅</p>
</li>
<li data-section-id="r9gi5j" data-start="243" data-end="295">
<p data-start="245" data-end="295">Standby is waiting for <strong data-start="268" data-end="295">Thread 2 sequence 12376</strong></p>
</li>
</ul>
<p data-start="297" data-end="318">But when you checked:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> thread#, low_sequence#, high_sequence# <span class="ͼg">FROM</span> v$archive_gap;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="398" data-end="405">Result:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">no rows selected</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="433" data-end="550">This means <strong data-start="444" data-end="491">Oracle does not detect a formal archive gap</strong>, but the <strong data-start="501" data-end="549">redo transport has not delivered the log yet</strong>.</p>
<hr data-start="552" data-end="555" />
<h1 data-section-id="zpzk7h" data-start="557" data-end="574">On Primary (DC)</h1>
<p data-start="576" data-end="589">You verified:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">THREAD# SEQUENCE#<br /><span class="ͼj">2</span> <span class="ͼj">12376</span><br /><span class="ͼj">2</span> <span class="ͼj">12377</span><br /><span class="ͼj">2</span> <span class="ͼj">12378</span><br /><span class="ͼj">2</span> <span class="ͼj">12379</span></div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="677" data-end="722">All logs <strong data-start="686" data-end="706">exist on primary</strong>, which is good.</p>
<p data-start="724" data-end="747">So the issue is simply:</p>
<p data-start="749" data-end="804"><strong data-start="749" data-end="804">Redo logs have not yet been shipped to the standby.</strong></p>
<hr data-start="806" data-end="809" />
<h1 data-section-id="m1luf4" data-start="811" data-end="858">Step 1 &mdash; Force Redo Shipping (Most Important)</h1>
<p data-start="860" data-end="884">Run this on <strong data-start="872" data-end="883">Primary</strong>:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">ALTER</span> SYSTEM ARCHIVE LOG <span class="ͼg">CURRENT</span>;<br /><span class="ͼg">ALTER</span> SYSTEM ARCHIVE LOG <span class="ͼg">CURRENT</span>;<br /><span class="ͼg">ALTER</span> SYSTEM ARCHIVE LOG <span class="ͼg">CURRENT</span>;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1000" data-end="1021">Run it <strong data-start="1007" data-end="1020">3&ndash;4 times</strong>.</p>
<p data-start="1023" data-end="1065">This forces LGWR to ship redo immediately.</p>
<hr data-start="1067" data-end="1070" />
<h1 data-section-id="16b5t6r" data-start="1072" data-end="1100">Step 2 &mdash; Check on DR Again</h1>
<p data-start="1102" data-end="1106">Run:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> process,status,thread#,sequence#<br /><span class="ͼg">FROM</span> v$managed_standby;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1184" data-end="1200">Expected change:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">MRP0 APPLYING_LOG</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1229" data-end="1240">instead of:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">WAIT_FOR_GAP</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<hr data-start="1264" data-end="1267" />
<h1 data-section-id="1g7psqx" data-start="1269" data-end="1300">Step 3 &mdash; Confirm Logs Arrived</h1>
<p data-start="1302" data-end="1308">Check:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> thread#,sequence#,applied<br /><span class="ͼg">FROM</span> v$archived_log<br /><span class="ͼg">ORDER</span> <span class="ͼg">BY</span> sequence#;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1395" data-end="1410">You should see:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">12376 APPLIED<br />12377 APPLIED</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<hr data-start="1451" data-end="1454" />
<h1 data-section-id="2dodaz" data-start="1456" data-end="1486">Step 4 &mdash; Check Transport Lag</h1>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> name,<span class="ͼg">value</span> <span class="ͼg">FROM</span> v$dataguard_stats<br /><span class="ͼg">WHERE</span> name <span class="ͼg">LIKE</span> <span class="ͼk">'%lag%'</span>;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1566" data-end="1594">Lag should start decreasing.</p>
<hr data-start="1596" data-end="1599" />
<h1 data-section-id="1lveao5" data-start="1601" data-end="1620">Why This Happened</h1>
<p data-start="1622" data-end="1678">Your server <strong data-start="1634" data-end="1654">rebooted earlier</strong>, so during that period:</p>
<ul data-start="1680" data-end="1761">
<li data-section-id="1mqod74" data-start="1680" data-end="1702">
<p data-start="1682" data-end="1702">Redo shipping paused</p>
</li>
<li data-section-id="1n11umi" data-start="1703" data-end="1731">
<p data-start="1705" data-end="1731">Standby recovery restarted</p>
</li>
<li data-section-id="5kot0l" data-start="1732" data-end="1761">
<p data-start="1734" data-end="1761">MRP waited for missing logs</p>
</li>
</ul>
<p data-start="1763" data-end="1824">Now the fix is simply <strong data-start="1785" data-end="1823">forcing primary to resend archives</strong>.</p>
<hr data-start="1826" data-end="1829" />
<h1 data-section-id="14bds48" data-start="1831" data-end="1857">If It Still Doesn't Ship</h1>
<p data-start="1859" data-end="1888">Then we check redo transport:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">SELECT</span> dest_id,status,error <span class="ͼg">FROM</span> v$archive_dest <span class="ͼg">WHERE</span> target<span class="ͼg">=</span><span class="ͼk">'STANDBY'</span>;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="1974" data-end="2016">This will show if transport has any error.</p>
<hr data-start="2018" data-end="2021" />
<h1 data-section-id="190ncm8" data-start="2023" data-end="2048">What Should Happen Next</h1>
<p data-start="2050" data-end="2072">After forcing archive:</p>
<p data-start="2074" data-end="2093">Expected DR status:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly">MRP0 APPLYING_LOG<br />RFS RECEIVING</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="2136" data-end="2173">Then sequences will start increasing.</p>
<hr data-start="2175" data-end="2178" />
<h1 data-section-id="eovskt" data-start="2180" data-end="2216">Quick Command to Run Now (Primary)</h1>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<div class="cm-content q9tKkq_readonly"><span class="ͼg">ALTER</span> SYSTEM ARCHIVE LOG <span class="ͼg">CURRENT</span>;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="">&nbsp;</div>
</div>
</div>
</div>
<p data-start="2264" data-end="2304">Run it <strong data-start="2271" data-end="2282">3 times</strong>, then check DR again.</p>
<!-- Comments are visible in the HTML source only -->

Troubleshooting MRP0 Issues:
If MRP0 is STOPPED or FAILED:

-- Start MRP0 (Real-Time Apply)
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE DISCONNECT FROM SESSION;
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE USING CURRENT LOGFILE DISCONNECT FROM SESSION;

-- Check errors
SELECT message FROM v$dataguard_status WHERE timestamp > SYSDATE - 1/24;
