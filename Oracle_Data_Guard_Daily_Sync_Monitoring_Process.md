<p class="demoTitle">&nbsp; &nbsp; &nbsp;</p>
<h1>Oracle Data Guard Daily Sync Monitoring Process</h1>
<h2>Purpose</h2>
<p>This document explains the Oracle Database DR synchronization monitoring process to be followed by&nbsp;Oracle Administrator for daily monitoring and status reporting.</p>
<p>The objective is to:</p>
<ul>
<li>
<p>Verify Primary and DR standby synchronization status</p>
</li>
<li>
<p>Monitor archive log shipping and apply status</p>
</li>
<li>
<p>Identify transport or apply lag issues</p>
</li>
<li>
<p>Share daily DR sync health status with the team</p>
</li>
</ul>
<hr />
<h1>Environment Details</h1>
<h2>Primary Database (DC)</h2>
<ul>
<li>
<p>Database Name: PRODCDB</p>
</li>
<li>
<p>Database Role: PRIMARY</p>
</li>
</ul>
<h2>DR Database</h2>
<ul>
<li>
<p>Database Name: PRODCDB_DR</p>
</li>
<li>
<p>Database Role: PHYSICAL STANDBY</p>
</li>
</ul>
<h2>Oracle Home</h2>
<pre><code class="language-bash">/oracle/app/oracle/database/19.3.0/dbhome_1
</code></pre>
<hr />
<h1>Daily Monitoring Activities</h1>
<p>The following checks must be performed daily.</p>
<hr />
<h1>1. Verify Database Role</h1>
<h2>On Primary Database</h2>
<p>Login as oracle user:</p>
<pre><code class="language-bash">sqlplus / as sysdba
</code></pre>
<p>Run:</p>
<pre><code class="language-sql">SELECT NAME,DATABASE_ROLE,OPEN_MODE FROM V$DATABASE;
</code></pre>
<p>Expected Output:</p>
<pre><code class="language-sql">NAME      DATABASE_ROLE   OPEN_MODE
--------- ---------------- --------------------
PRODCDB   PRIMARY         READ WRITE
</code></pre>
<hr />
<h2>On DR Database</h2>
<pre><code class="language-bash">sqlplus / as sysdba
</code></pre>
<p>Run:</p>
<pre><code class="language-sql">SELECT NAME,DATABASE_ROLE,OPEN_MODE FROM V$DATABASE;
</code></pre>
<p>Expected Output:</p>
<pre><code class="language-sql">NAME         DATABASE_ROLE     OPEN_MODE
------------ ----------------- --------------------
PRODCDB      PHYSICAL STANDBY  MOUNTED
</code></pre>
<hr />
<h1>2. Verify Managed Recovery Process (MRP)</h1>
<p>Run on DR database:</p>
<pre><code class="language-sql">SELECT PROCESS,STATUS,THREAD#,SEQUENCE#
FROM V$MANAGED_STANDBY;
</code></pre>
<p>Expected:</p>
<ul>
<li>
<p>MRP0 process should be visible</p>
</li>
<li>
<p>Status should show APPLYING_LOG</p>
</li>
</ul>
<p>Example:</p>
<pre><code class="language-sql">PROCESS STATUS         THREAD# SEQUENCE#
------- -------------- ------- ----------
MRP0    APPLYING_LOG   1       12345
</code></pre>
<hr />
<h1>3. Verify Archive Log Sync Status</h1>
<p>Run on DR database:</p>
<pre><code class="language-sql">SELECT THREAD#, MAX(SEQUENCE#)
FROM V$ARCHIVED_LOG
WHERE APPLIED='YES'
GROUP BY THREAD#;
</code></pre>
<p>Run on Primary database:</p>
<pre><code class="language-sql">SELECT THREAD#, MAX(SEQUENCE#)
FROM V$ARCHIVED_LOG
GROUP BY THREAD#;
</code></pre>
<p>Compare both outputs.</p>
<p>Expected:</p>
<ul>
<li>
<p>DR applied sequence should be near current primary sequence</p>
</li>
<li>
<p>Small gap is acceptable</p>
</li>
</ul>
<hr />
<h1>4. Check Data Guard Lag</h1>
<p>Run on DR:</p>
<pre><code class="language-sql">SELECT NAME,VALUE,TIME_COMPUTED
FROM V$DATAGUARD_STATS
WHERE NAME IN ('transport lag','apply lag');
</code></pre>
<p>Expected:</p>
<ul>
<li>
<p>Lag should be minimal</p>
</li>
<li>
<p>Ideally less than few minutes</p>
</li>
</ul>
<p>Example:</p>
<pre><code class="language-sql">NAME            VALUE
--------------- ----------------
transport lag   +00 00:00:10
apply lag       +00 00:00:20
</code></pre>
<hr />
<h1>5. Verify Archive Destination Status</h1>
<p>Run on Primary database:</p>
<pre><code class="language-sql">SELECT DEST_ID,STATUS,ERROR,DESTINATION
FROM V$ARCHIVE_DEST
WHERE TARGET='STANDBY';
</code></pre>
<p>Expected:</p>
<pre><code class="language-sql">STATUS = VALID
ERROR  = NULL
</code></pre>
<p>If ERROR column shows issue, inform DBA team immediately.</p>
<hr />
<h1>6. Verify Listener Status</h1>
<p>Run on both nodes:</p>
<pre><code class="language-bash">lsnrctl status
</code></pre>
<p>Expected:</p>
<ul>
<li>
<p>Listener should be running</p>
</li>
<li>
<p>Database services should be registered</p>
</li>
</ul>
<hr />
<h1>7. Verify Cluster Resources</h1>
<p>Run as grid user:</p>
<pre><code class="language-bash">crsctl status resource -t
</code></pre>
<p>Verify:</p>
<ul>
<li>
<p>ASM should be ONLINE</p>
</li>
<li>
<p>VIP should be ONLINE</p>
</li>
<li>
<p>Listener should be ONLINE</p>
</li>
<li>
<p>Database should be ONLINE</p>
</li>
</ul>
<hr />
<h1>8. Verify RAC Instance Status</h1>
<p>Run as grid user:</p>
<pre><code class="language-bash">srvctl status database -db PRODCDB_DR
</code></pre>
<p>Expected:</p>
<pre><code class="language-bash">Instance PRODCDB_DR1 is running on node erpproddb1-dr
Instance PRODCDB_DR2 is running on node erpproddb2-dr
</code></pre>
<hr />
<h1>9. Verify ASM Diskgroup Status</h1>
<p>Run as grid user:</p>
<pre><code class="language-bash">asmcmd lsdg
</code></pre>
<p>Expected:</p>
<ul>
<li>
<p>DATA diskgroup mounted</p>
</li>
<li>
<p>OCRVOTE diskgroup mounted</p>
</li>
</ul>
<hr />
<h1>Common Troubleshooting</h1>
<h2>A. MRP Not Running</h2>
<p>Check:</p>
<pre><code class="language-sql">SELECT PROCESS,STATUS FROM V$MANAGED_STANDBY;
</code></pre>
<p>Start recovery:</p>
<pre><code class="language-sql">ALTER DATABASE RECOVER MANAGED STANDBY DATABASE DISCONNECT FROM SESSION;
</code></pre>
<hr />
<h2>B. Archive Gap Found</h2>
<p>Run:</p>
<pre><code class="language-sql">SELECT * FROM V$ARCHIVE_GAP;
</code></pre>
<p>Inform DBA team if gap exists.</p>
<hr />
<h2>C. Transport Lag Increasing</h2>
<p>Check:</p>
<ul>
<li>
<p>Network connectivity</p>
</li>
<li>
<p>Listener status</p>
</li>
<li>
<p>Archive destination status</p>
</li>
<li>
<p>Disk space</p>
</li>
</ul>
<p>Commands:</p>
<pre><code class="language-bash">ping &lt;remote-host&gt;
</code></pre>
<pre><code class="language-bash">tnsping &lt;service_name&gt;
</code></pre>
<hr />
<h2>D. Listener Issue</h2>
<p>Restart listener:</p>
<pre><code class="language-bash">lsnrctl stop
lsnrctl start
</code></pre>
<hr />
<h2>E. CRS Resource Failure</h2>
<p>Check cluster resources:</p>
<pre><code class="language-bash">crsctl status resource -t
</code></pre>
<p>Restart resource if required.</p>
<hr />
<h1>Daily Status Reporting Format</h1>
<p>Raveen should share daily status in below format:</p>
<hr />
<h2>Daily DR Sync Status Report</h2>
<p>Date: DD-MM-YYYY<br />Time: HH:MM</p>
<h3>Database Status</h3>
<ul>
<li>
<p>Primary DB: UP</p>
</li>
<li>
<p>DR DB: UP</p>
</li>
<li>
<p>Managed Recovery: RUNNING</p>
</li>
</ul>
<h3>Archive Sync Status</h3>
<ul>
<li>
<p>Primary Sequence: XXXXX</p>
</li>
<li>
<p>DR Applied Sequence: XXXXX</p>
</li>
<li>
<p>Gap: 0</p>
</li>
</ul>
<h3>Data Guard Lag</h3>
<ul>
<li>
<p>Transport Lag: 0 Seconds</p>
</li>
<li>
<p>Apply Lag: 0 Seconds</p>
</li>
</ul>
<h3>Cluster Status</h3>
<ul>
<li>
<p>ASM: ONLINE</p>
</li>
<li>
<p>VIP: ONLINE</p>
</li>
<li>
<p>Listener: ONLINE</p>
</li>
<li>
<p>Database Instances: ONLINE</p>
</li>
</ul>
<h3>Observations</h3>
<ul>
<li>
<p>DR synchronization working normally.</p>
</li>
</ul>
<p>OR</p>
<ul>
<li>
<p>Observed transport lag of 5 minutes.</p>
</li>
<li>
<p>DBA team informed.</p>
</li>
</ul>
<hr />
<h1>Escalation Conditions</h1>
<p>Immediately inform DBA team if:</p>
<ul>
<li>
<p>Managed recovery stopped</p>
</li>
<li>
<p>Archive gap exists</p>
</li>
<li>
<p>Apply lag continuously increasing</p>
</li>
<li>
<p>Listener down</p>
</li>
<li>
<p>ASM dismounted</p>
</li>
<li>
<p>CRS resource OFFLINE</p>
</li>
<li>
<p>Database inaccessible</p>
</li>
<li>
<p>Redo transport failure</p>
</li>
</ul>
<hr />
<h1>Important Commands Quick Reference</h1>
<h2>Database Role</h2>
<pre><code class="language-sql">SELECT NAME,DATABASE_ROLE,OPEN_MODE FROM V$DATABASE;
</code></pre>
<h2>MRP Status</h2>
<pre><code class="language-sql">SELECT PROCESS,STATUS,THREAD#,SEQUENCE# FROM V$MANAGED_STANDBY;
</code></pre>
<h2>Lag Status</h2>
<pre><code class="language-sql">SELECT NAME,VALUE FROM V$DATAGUARD_STATS;
</code></pre>
<h2>Archive Destination</h2>
<pre><code class="language-sql">SELECT DEST_ID,STATUS,ERROR FROM V$ARCHIVE_DEST WHERE TARGET='STANDBY';
</code></pre>
<h2>CRS Status</h2>
<pre><code class="language-bash">crsctl status resource -t
</code></pre>
<h2>RAC Database Status</h2>
<pre><code class="language-bash">srvctl status database -db PRODCDB_DR
</code></pre>
<h2>ASM Status</h2>
<pre><code class="language-bash">asmcmd lsdg
</code></pre>
<hr />
<h1>Conclusion</h1>
<p>This monitoring process ensures that the Oracle DR environment remains synchronized with the Primary database and any issue is identified proactively.</p>
<p>Daily monitoring and reporting must be performed consistently to maintain DR readiness and database availability.</p>
<!-- Comments are visible in the HTML source only -->
