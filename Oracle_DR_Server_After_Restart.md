<p data-path-to-node="0">Since <code data-path-to-node="0" data-index-in-node="6">crsctl check has</code> shows that the High Availability Services (HAS) are <strong data-path-to-node="0" data-index-in-node="75">online</strong>, but <code data-path-to-node="0" data-index-in-node="87">srvctl</code> is failing with communication errors, it indicates that the underlying CSS (Cluster Synchronization Services) daemon is likely not running or is stuck.</p>
<p data-path-to-node="1">Because you are working on <code data-path-to-node="1" data-index-in-node="27">erpproddb1-dr</code> (a Disaster Recovery node), it is common for these services to be down or in a "waiting" state.</p>
<p data-path-to-node="2">Here is how to resolve this:</p>
<h3 data-path-to-node="3">1. Start the services as ROOT</h3>
<p data-path-to-node="4">The <code data-path-to-node="4" data-index-in-node="4">grid</code> user does not have permission to stop or start the high-level cluster stack (<code data-path-to-node="4" data-index-in-node="86">has</code>). You must use the <strong data-path-to-node="4" data-index-in-node="109">full path</strong> as root.</p>
<p data-path-to-node="5"><strong data-path-to-node="5" data-index-in-node="0">Run these as root:</strong></p>
<div class="code-block ng-tns-c986751816-109 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwi5nIDNorqSAxUAAAAAHQAAAAAQsgE">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c986751816-109 ng-star-inserted"><span class="ng-tns-c986751816-109">Bash</span>
<div class="buttons ng-tns-c986751816-109 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c986751816-109">
<div class="animated-opacity ng-tns-c986751816-109">
<pre class="ng-tns-c986751816-109"><code class="code-container formatted ng-tns-c986751816-109" data-test-id="code-content"><span class="hljs-comment"># Force a restart of the High Availability stack</span>
/oracle/app/grid/19.3.0/gridhome_1/bin/crsctl stop has -f
/oracle/app/grid/19.3.0/gridhome_1/bin/crsctl start has
</code></pre>
</div>
</div>
</div>
<h3 data-path-to-node="7">2. Monitor the stack startup</h3>
<p data-path-to-node="8">Wait about 2&ndash;3 minutes, then run this command as <strong data-path-to-node="8" data-index-in-node="49">root</strong> to see which components are actually coming online:</p>
<div class="code-block ng-tns-c986751816-110 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwi5nIDNorqSAxUAAAAAHQAAAAAQswE">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c986751816-110 ng-star-inserted"><span class="ng-tns-c986751816-110">Bash</span>
<div class="buttons ng-tns-c986751816-110 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c986751816-110">
<div class="animated-opacity ng-tns-c986751816-110">
<pre class="ng-tns-c986751816-110"><code class="code-container formatted ng-tns-c986751816-110" data-test-id="code-content">/oracle/app/grid/19.3.0/gridhome_1/bin/crsctl <span class="hljs-built_in">stat</span> res -t
</code></pre>
</div>
</div>
</div>
<p data-path-to-node="10"><em data-path-to-node="10" data-index-in-node="0">Look for <code data-path-to-node="10" data-index-in-node="9">ora.asm</code> and <code data-path-to-node="10" data-index-in-node="21">ora.cssd</code>. They need to be <strong data-path-to-node="10" data-index-in-node="47">ONLINE</strong>.</em></p>
<hr data-path-to-node="11" />
<h3 data-path-to-node="12">3. If CSS is still down (The "PRKH-3003" fix)</h3>
<p data-path-to-node="13">If the command above shows <code data-path-to-node="13" data-index-in-node="27">ora.cssd</code> is OFFLINE or STARTING, you cannot start ASM. As root, try to start the specific resources:</p>
<div class="code-block ng-tns-c986751816-111 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwi5nIDNorqSAxUAAAAAHQAAAAAQtAE">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c986751816-111 ng-star-inserted"><span class="ng-tns-c986751816-111">Bash</span>
<div class="buttons ng-tns-c986751816-111 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c986751816-111">
<div class="animated-opacity ng-tns-c986751816-111">
<pre class="ng-tns-c986751816-111"><code class="code-container formatted ng-tns-c986751816-111" data-test-id="code-content"><span class="hljs-comment"># Start the CSS daemon (required for ASM)</span>
/oracle/app/grid/19.3.0/gridhome_1/bin/crsctl start resource ora.cssd

<span class="hljs-comment"># Once CSS is online, start ASM</span>
/oracle/app/grid/19.3.0/gridhome_1/bin/crsctl start resource ora.asm</code></pre>
</div>
</div>
</div>
<!-- Comments are visible in the HTML source only -->

[grid@erpproddb1-dr ~]$ asmcmd du /DATA/PRODCDB_DR/ARCHIVELOG/
Used_MB      Mirror_used_MB
 221228              221228

<p dir="auto">[oracle@erpproddb1-dr ~]$ export ORACLE_SID=PRODCDB_DR<br />[oracle@erpproddb1-dr ~]$ echo $ORACLE_SID<br />PRODCDB_DR<br />[oracle@erpproddb1-dr ~]$ sqlplus / as sysdba</p>
<p dir="auto">SQL*Plus: Release 19.0.0.0.0 - Production on Fri Mar 13 11:55:14 2026<br />Version 19.27.0.0.0</p>
<p dir="auto">Copyright (c) 1982, 2024, Oracle. All rights reserved.</p>
<p dir="auto"><br />Connected to:<br />Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production<br />Version 19.27.0.0.0</p>
<p dir="auto">SQL&gt; SELECT name,open_mode,database_role FROM v$database;</p>
<p dir="auto">NAME OPEN_MODE DATABASE_ROLE<br />--------- -------------------- ----------------<br />PRODCDB MOUNTED PHYSICAL STANDBY</p>
<p dir="auto">SQL&gt; SELECT process,status,thread#,sequence#<br />FROM v$managed_standby; 2</p>
<p dir="auto">PROCESS STATUS THREAD# SEQUENCE#<br />--------- ------------ ---------- ----------<br />ARCH CONNECTED 0 0<br />DGRD ALLOCATED 0 0<br />DGRD ALLOCATED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />RFS IDLE 1 0<br />RFS IDLE 1 14859<br />RFS IDLE 1 0<br />RFS IDLE 2 12380</p>
<p dir="auto">10 rows selected.</p>
<p dir="auto">SQL&gt;</p>
<p dir="auto">Your standby database is&nbsp;<strong>mounted correctly</strong>, but from the output we can see that&nbsp;<strong>redo apply is NOT running yet</strong>.</p>
<div class="markdown-heading" dir="auto">
<h3 class="heading-element" dir="auto" tabindex="-1">What your output indicates</h3>
<a id="user-content-what-your-output-indicates" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#what-your-output-indicates"></a></div>
<p dir="auto">From:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;process,status,thread#,sequence#&nbsp;FROM&nbsp;v$managed_standby;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">You only see:</p>
<ul dir="auto">
<li>
<p dir="auto"><code>RFS</code>&nbsp;processes &rarr; Receiving redo</p>
</li>
<li>
<p dir="auto"><code>ARCH</code>&nbsp;processes &rarr; Archive handling</p>
</li>
<li>
<p dir="auto"><strong>No&nbsp;<code>MRP0</code>&nbsp;process</strong></p>
</li>
</ul>
<p dir="auto">Example:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">RFS IDLE 1 14859<br />RFS IDLE 2 12380</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">This means:</p>
<ul dir="auto">
<li>
<p dir="auto">Standby is&nbsp;<strong>mounted correctly</strong></p>
</li>
<li>
<p dir="auto">Redo transport may be arriving</p>
</li>
<li>
<p dir="auto"><strong>Managed Recovery Process (MRP) is not started</strong>, so logs are&nbsp;<strong>not being applied</strong></p>
</li>
</ul>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Start Data Guard Apply (Important Step)</h1>
<a id="user-content-start-data-guard-apply-important-step" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#start-data-guard-apply-important-step"></a></div>
<p dir="auto">Run the following command:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">ALTER&nbsp;DATABASE RECOVER MANAGED STANDBY DATABASE&nbsp;USING&nbsp;CURRENT&nbsp;LOGFILE&nbsp;DISCONNECT&nbsp;FROM&nbsp;SESSION;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">This will start&nbsp;<strong>real-time redo apply</strong>.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Verify After Starting Recovery</h1>
<a id="user-content-verify-after-starting-recovery" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#verify-after-starting-recovery"></a></div>
<p dir="auto">Run again:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;process,status,thread#,sequence#<br />FROM&nbsp;v$managed_standby;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Now you should see something like:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">PROCESS STATUS THREAD# SEQUENCE#<br />------ ------------- ------- --------<br />MRP0 APPLYING_LOG 1 14859<br />RFS RECEIVING 1 14860<br />RFS RECEIVING 2 12381</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">The important line is:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">MRP0 APPLYING_LOG</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">This confirms&nbsp;<strong>Data Guard sync is active</strong>.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Check DR Sync Lag</h1>
<a id="user-content-check-dr-sync-lag" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#check-dr-sync-lag"></a></div>
<p dir="auto">Run:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;name,value&nbsp;FROM&nbsp;v$dataguard_stats<br />WHERE&nbsp;name&nbsp;LIKE&nbsp;'%lag%';</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Example:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">transport lag +00 00:01:10<br />apply lag +00 00:00:40</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Lag will start&nbsp;<strong>reducing automatically</strong>.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Quick Health Check (Recommended)</h1>
<a id="user-content-quick-health-check-recommended" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#quick-health-check-recommended"></a></div>
<p dir="auto">Run:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;thread#, MAX(sequence#)<br />FROM&nbsp;v$archived_log<br />GROUP&nbsp;BY&nbsp;thread#;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Compare this with&nbsp;<strong>primary database archive log sequence</strong>&nbsp;to ensure DR is catching up.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Current DR Status</h1>
<a id="user-content-current-dr-status" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#current-dr-status"></a></div>
<div dir="auto">
<div dir="auto" tabindex="-1">
<table>
<thead>
<tr>
<th>Component</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td>Grid Infrastructure</td>
<td>✅ Running</td>
</tr>
<tr>
<td>ASM</td>
<td>✅ Running</td>
</tr>
<tr>
<td>Diskgroups</td>
<td>✅ Mounted</td>
</tr>
<tr>
<td>Standby DB</td>
<td>✅ Mounted</td>
</tr>
<tr>
<td>Redo Apply</td>
<td>❌ Not Running</td>
</tr>
<tr>
<td>Required Action</td>
<td>Start MRP</td>
</tr>
</tbody>
</table>
</div>
</div>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">One Important Observation</h1>
<a id="user-content-one-important-observation" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#one-important-observation"></a></div>
<p dir="auto">Your RFS shows:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">Thread 1 &rarr; 14859<br />Thread 2 &rarr; 12380</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">This indicates&nbsp;<strong>RAC primary with two redo threads</strong>, which is correct.</p>
<hr />
<p dir="auto">✅&nbsp;<strong>Next step:</strong><br />Run:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">ALTER&nbsp;DATABASE RECOVER MANAGED STANDBY DATABASE&nbsp;USING&nbsp;CURRENT&nbsp;LOGFILE&nbsp;DISCONNECT&nbsp;FROM&nbsp;SESSION;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">and then check&nbsp;<code>v$managed_standby</code>&nbsp;again.</p>
<p dir="auto">&nbsp;</p>
<p dir="auto">SQL&gt; ALTER DATABASE RECOVER MANAGED STANDBY DATABASE USING CURRENT LOGFILE DISCONNECT FROM SESSION;</p>
<p dir="auto">Database altered.</p>
<p dir="auto">SQL&gt; SELECT process,status,thread#,sequence#<br />FROM v$managed_standby; 2</p>
<p dir="auto">PROCESS STATUS THREAD# SEQUENCE#<br />--------- ------------ ---------- ----------<br />ARCH CONNECTED 0 0<br />DGRD ALLOCATED 0 0<br />DGRD ALLOCATED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />RFS IDLE 1 0<br />RFS IDLE 1 14859<br />RFS IDLE 1 0<br />RFS IDLE 2 12380<br />MRP0 WAIT_FOR_GAP 2 12376</p>
<p dir="auto">11 rows selected.</p>
<p dir="auto">SQL&gt;</p>
<p dir="auto">&nbsp;</p>
<p dir="auto">Your standby recovery&nbsp;<strong>started successfully</strong>, but the status now shows a&nbsp;<strong>redo gap</strong>.</p>
<div class="markdown-heading" dir="auto">
<h3 class="heading-element" dir="auto" tabindex="-1">Current Status Interpretation</h3>
<a id="user-content-current-status-interpretation" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#current-status-interpretation"></a></div>
<p dir="auto">From your output:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">MRP0 WAIT_FOR_GAP 2 12376</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Meaning:</p>
<ul dir="auto">
<li>
<p dir="auto"><strong>MRP0</strong>&nbsp;&rarr; Managed Recovery Process is running ✅</p>
</li>
<li>
<p dir="auto"><strong>WAIT_FOR_GAP</strong>&nbsp;&rarr; Some archive logs are&nbsp;<strong>missing on standby</strong></p>
</li>
<li>
<p dir="auto">Apply cannot continue until those logs arrive.</p>
</li>
</ul>
<p dir="auto">Your RFS entries show:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">Thread 1 last received : 14859<br />Thread 2 last received : 12380</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">But MRP is waiting for:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">Thread 2 sequence 12376</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">This means&nbsp;<strong>some archive logs between sequences are missing on DR</strong>.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Step 1 &mdash; Identify the Missing Archive Logs</h1>
<a id="user-content-step-1--identify-the-missing-archive-logs" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#step-1--identify-the-missing-archive-logs"></a></div>
<p dir="auto">Run this on&nbsp;<strong>DR</strong>:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;thread#, low_sequence#, high_sequence#<br />FROM&nbsp;v$archive_gap;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">This will show exactly which archive logs are missing.</p>
<p dir="auto">Example output:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">THREAD# LOW_SEQUENCE# HIGH_SEQUENCE#<br />2 12376 12379</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Step 2 &mdash; Check if Logs Exist on Primary (DC)</h1>
<a id="user-content-step-2--check-if-logs-exist-on-primary-dc" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#step-2--check-if-logs-exist-on-primary-dc"></a></div>
<p dir="auto">Login to&nbsp;<strong>Primary Database</strong>&nbsp;and run:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;thread#, sequence#, archived<br />FROM&nbsp;v$archived_log<br />WHERE&nbsp;thread#=2<br />AND&nbsp;sequence#&nbsp;BETWEEN&nbsp;12376&nbsp;AND&nbsp;12380;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">If they exist, Data Guard can fetch them automatically.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Step 3 &mdash; Force Archive Log Shipping from Primary</h1>
<a id="user-content-step-3--force-archive-log-shipping-from-primary" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#step-3--force-archive-log-shipping-from-primary"></a></div>
<p dir="auto">On&nbsp;<strong>Primary</strong>&nbsp;run:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">ALTER&nbsp;SYSTEM ARCHIVE LOG&nbsp;CURRENT;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Run it&nbsp;<strong>2&ndash;3 times</strong>.</p>
<p dir="auto">This forces redo shipping to standby.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Step 4 &mdash; Check if Gap Resolved</h1>
<a id="user-content-step-4--check-if-gap-resolved" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#step-4--check-if-gap-resolved"></a></div>
<p dir="auto">Run again on DR:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;process,status,thread#,sequence#<br />FROM&nbsp;v$managed_standby;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Expected:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">MRP0 APPLYING_LOG</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Instead of:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">WAIT_FOR_GAP</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Step 5 &mdash; If Gap Still Exists</h1>
<a id="user-content-step-5--if-gap-still-exists" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#step-5--if-gap-still-exists"></a></div>
<p dir="auto">You may need to&nbsp;<strong>manually copy missing archive logs</strong>.</p>
<ol dir="auto">
<li>
<p dir="auto">Find archive location on primary:</p>
</li>
</ol>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">archive log list;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<ol dir="auto" start="2">
<li>
<p dir="auto">Copy missing logs:</p>
</li>
</ol>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">scp arch_2_12376*.arc oracle@DR_SERVER:/archive_location/</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<ol dir="auto" start="3">
<li>
<p dir="auto">Register logs on DR:</p>
</li>
</ol>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">ALTER&nbsp;DATABASE REGISTER LOGFILE&nbsp;'/path/arch_2_12376.arc';</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Then recovery continues automatically.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Step 6 &mdash; Verify Data Guard Lag</h1>
<a id="user-content-step-6--verify-data-guard-lag" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#step-6--verify-data-guard-lag"></a></div>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;name,value&nbsp;FROM&nbsp;v$dataguard_stats<br />WHERE&nbsp;name&nbsp;LIKE&nbsp;'%lag%';</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Lag should begin reducing.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Quick Summary</h1>
<a id="user-content-quick-summary" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#quick-summary"></a></div>
<div dir="auto">
<div dir="auto" tabindex="-1">
<table>
<thead>
<tr>
<th>Component</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td>Standby Database</td>
<td>Mounted</td>
</tr>
<tr>
<td>MRP</td>
<td>Running</td>
</tr>
<tr>
<td>Redo Apply</td>
<td>Waiting</td>
</tr>
<tr>
<td>Issue</td>
<td>Archive Gap</td>
</tr>
<tr>
<td>Action</td>
<td>Resolve missing archive logs</td>
</tr>
</tbody>
</table>
</div>
</div>
<hr />
<p dir="auto">💡&nbsp;<strong>Very important:</strong><br />Since your DR node rebooted earlier,&nbsp;<strong>some archive logs might have been missed during downtime</strong>, which is why the gap occurred.</p>
<p dir="auto">&nbsp;</p>
<p dir="auto">on DR SQL&gt; SELECT process,status,thread#,sequence#<br />FROM v$managed_standby; 2</p>
<p dir="auto">PROCESS STATUS THREAD# SEQUENCE#<br />--------- ------------ ---------- ----------<br />ARCH CONNECTED 0 0<br />DGRD ALLOCATED 0 0<br />DGRD ALLOCATED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />ARCH CONNECTED 0 0<br />RFS IDLE 1 0<br />RFS IDLE 1 14859<br />RFS IDLE 1 0<br />RFS IDLE 2 12380<br />MRP0 WAIT_FOR_GAP 2 12376</p>
<p dir="auto">11 rows selected.</p>
<p dir="auto">SQL&gt; SELECT thread#, low_sequence#, high_sequence#<br />FROM v$archive_gap; 2</p>
<p dir="auto">no rows selected</p>
<p dir="auto">SQL&gt; on primary DC sqlplus sys/sys123@PRODCDB_PROD as sysdba</p>
<p dir="auto">SQL*Plus: Release 19.0.0.0.0 - Production on Fri Mar 13 12:00:14 2026<br />Version 19.27.0.0.0</p>
<p dir="auto">Copyright (c) 1982, 2024, Oracle. All rights reserved.</p>
<p dir="auto"><br />Connected to:<br />Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production<br />Version 19.27.0.0.0</p>
<p dir="auto">SQL&gt; SELECT thread#, sequence#, archived<br />FROM v$archived_log<br />WHERE thread#=2<br />AND sequence# BETWEEN 12376 AND 12380; 2 3 4</p>
<p dir="auto">THREAD# SEQUENCE# ARC<br />---------- ---------- ---<br />2 12376 YES<br />2 12377 YES<br />2 12378 YES<br />2 12379 YES</p>
<p dir="auto">SQL&gt;</p>
<p dir="auto">&nbsp;</p>
<p dir="auto">Your situation is&nbsp;<strong>very close to normal</strong>, but there is a small&nbsp;<strong>redo shipping issue</strong>. Let&rsquo;s interpret the results carefully.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Current Situation</h1>
<a id="user-content-current-situation" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#current-situation"></a></div>
<div class="markdown-heading" dir="auto">
<h3 class="heading-element" dir="auto" tabindex="-1">On DR</h3>
<a id="user-content-on-dr" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#on-dr"></a></div>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">MRP0 WAIT_FOR_GAP 2 12376</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Meaning:</p>
<ul dir="auto">
<li>
<p dir="auto">Managed Recovery is running ✅</p>
</li>
<li>
<p dir="auto">Standby is waiting for&nbsp;<strong>Thread 2 sequence 12376</strong></p>
</li>
</ul>
<p dir="auto">But when you checked:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;thread#, low_sequence#, high_sequence#&nbsp;FROM&nbsp;v$archive_gap;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Result:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">no rows selected</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">This means&nbsp;<strong>Oracle does not detect a formal archive gap</strong>, but the&nbsp;<strong>redo transport has not delivered the log yet</strong>.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">On Primary (DC)</h1>
<a id="user-content-on-primary-dc" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#on-primary-dc"></a></div>
<p dir="auto">You verified:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">THREAD# SEQUENCE#<br />2&nbsp;12376<br />2&nbsp;12377<br />2&nbsp;12378<br />2&nbsp;12379</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">All logs&nbsp;<strong>exist on primary</strong>, which is good.</p>
<p dir="auto">So the issue is simply:</p>
<p dir="auto"><strong>Redo logs have not yet been shipped to the standby.</strong></p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Step 1 &mdash; Force Redo Shipping (Most Important)</h1>
<a id="user-content-step-1--force-redo-shipping-most-important" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#step-1--force-redo-shipping-most-important"></a></div>
<p dir="auto">Run this on&nbsp;<strong>Primary</strong>:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">ALTER&nbsp;SYSTEM ARCHIVE LOG&nbsp;CURRENT;<br />ALTER&nbsp;SYSTEM ARCHIVE LOG&nbsp;CURRENT;<br />ALTER&nbsp;SYSTEM ARCHIVE LOG&nbsp;CURRENT;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Run it&nbsp;<strong>3&ndash;4 times</strong>.</p>
<p dir="auto">This forces LGWR to ship redo immediately.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Step 2 &mdash; Check on DR Again</h1>
<a id="user-content-step-2--check-on-dr-again" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#step-2--check-on-dr-again"></a></div>
<p dir="auto">Run:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;process,status,thread#,sequence#<br />FROM&nbsp;v$managed_standby;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Expected change:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">MRP0 APPLYING_LOG</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">instead of:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">WAIT_FOR_GAP</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Step 3 &mdash; Confirm Logs Arrived</h1>
<a id="user-content-step-3--confirm-logs-arrived" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#step-3--confirm-logs-arrived"></a></div>
<p dir="auto">Check:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;thread#,sequence#,applied<br />FROM&nbsp;v$archived_log<br />ORDER&nbsp;BY&nbsp;sequence#;</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">You should see:</p>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">12376 APPLIED<br />12377 APPLIED</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Step 4 &mdash; Check Transport Lag</h1>
<a id="user-content-step-4--check-transport-lag" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#step-4--check-transport-lag"></a></div>
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">
<div dir="auto">&nbsp;</div>
<div dir="auto">
<div dir="auto">
<div id="user-content-code-block-viewer" dir="ltr">
<div dir="auto">
<div dir="auto">SELECT&nbsp;name,value&nbsp;FROM&nbsp;v$dataguard_stats<br />WHERE&nbsp;name&nbsp;LIKE&nbsp;'%lag%';</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div dir="auto">&nbsp;</div>
</div>
</div>
</div>
<p dir="auto">Lag should start decreasing.</p>
<hr />
<div class="markdown-heading" dir="auto">
<h1 class="heading-element" dir="auto" tabindex="-1">Why This Happened</h1>
<a id="user-content-why-this-happened" class="anchor" href="https://github.com/alokaryk/Test/blob/main/oracle_command.md#why-this-happened"></a></div>
<p dir="auto">Your server&nbsp;<strong>rebooted earlier</strong>, so during that period:</p>
<ul dir="auto">
<li>
<p dir="auto">Redo shipping paused</p>
</li>
<li>
<p dir="auto">Standby recovery restarted</p>
</li>
<li>
<p dir="auto">MRP waited for missing logs</p>
</li>
</ul>
<p dir="auto">Now the fix is simply&nbsp;<strong>forcing primary to resend archives</strong>.</p>
<!-- Comments are visible in the HTML source only -->
