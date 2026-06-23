<p>Yes, but only if your intention is to <strong>access the EBS application during the DR drill</strong>. Since your standby has already been converted back to <strong>PHYSICAL STANDBY</strong> and MRP is running, you cannot open the PDB while redo apply is active.</p>
<p>Current state:</p>
<pre><code class="language-sql">DATABASE_ROLE : PHYSICAL STANDBY
OPEN_MODE     : MOUNTED
PDB PROD      : MOUNTED
MRP0          : APPLYING_LOG
</code></pre>
<p>This is the correct Data Guard state.</p>
<h3>If you want EBS users to access the application</h3>
<p>You must stop redo apply and open the standby read-only.</p>
<h4>1. Cancel MRP</h4>
<pre><code class="language-sql">ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;
</code></pre>
<p>Verify:</p>
<pre><code class="language-sql">SELECT PROCESS, STATUS
FROM V$MANAGED_STANDBY
WHERE PROCESS LIKE 'MRP%';
</code></pre>
<p>No rows should return.</p>
<h4>2. Open standby read-only</h4>
<pre><code class="language-sql">ALTER DATABASE OPEN;
</code></pre>
<p>Verify:</p>
<pre><code class="language-sql">SELECT OPEN_MODE, DATABASE_ROLE
FROM V$DATABASE;
</code></pre>
<p>Expected:</p>
<pre><code class="language-text">OPEN_MODE      DATABASE_ROLE
-------------- ----------------
READ ONLY      PHYSICAL STANDBY
</code></pre>
<h4>3. Open the PDB</h4>
<pre><code class="language-sql">ALTER SESSION SET CONTAINER=CDB$ROOT;

ALTER PLUGGABLE DATABASE PROD OPEN READ ONLY;
</code></pre>
<p>Verify:</p>
<pre><code class="language-sql">SHOW PDBS;
</code></pre>
<p>Expected:</p>
<pre><code class="language-text">CON_ID CON_NAME OPEN MODE
------ -------- ----------
4      PROD     READ ONLY
</code></pre>
<h4>4. Check APPS account</h4>
<p>Your query had a syntax error because spaces were missing.</p>
<p>Use:</p>
<pre><code class="language-sql">ALTER SESSION SET CONTAINER=PROD;

SELECT USERNAME,
       ACCOUNT_STATUS
FROM DBA_USERS
WHERE USERNAME='APPS';
</code></pre>
<p>You should see:</p>
<pre><code class="language-text">USERNAME ACCOUNT_STATUS
-------- --------------
APPS     OPEN
</code></pre>
<h3>Important</h3>
<p>Opening a Physical Standby read-only:</p>
<ul>
<li>
<p>✅ Allows EBS login/testing.</p>
</li>
<li>
<p>✅ Does not require Snapshot Standby conversion.</p>
</li>
<li>
<p>❌ Redo apply stops while it is open read-only.</p>
</li>
<li>
<p>❌ Data Guard will fall behind until you restart MRP.</p>
</li>
</ul>
<p>After the DR drill, return it to apply mode:</p>
<pre><code class="language-sql">SHUTDOWN IMMEDIATE;
STARTUP MOUNT;

ALTER DATABASE RECOVER MANAGED STANDBY DATABASE
USING CURRENT LOGFILE DISCONNECT;
</code></pre>
<p>Then verify:</p>
<pre><code class="language-sql">SELECT PROCESS, STATUS
FROM V$MANAGED_STANDBY
WHERE PROCESS LIKE 'MRP%';
</code></pre>
<p>and</p>
<pre><code class="language-sql">SELECT NAME, VALUE
FROM V$DATAGUARD_STATS
WHERE NAME IN ('transport lag','apply lag');
</code></pre>
<p>Since your DR drill objective is to bring up the EBS application, confirm first whether the customer wants:</p>
<ol>
<li>
<p><strong>Read-only DR validation/testing</strong> (open Physical Standby READ ONLY), or</p>
</li>
<li>
<p><strong>Full DR activation with EBS transactions</strong> (convert to Snapshot Standby or activate the standby).</p>
</li>
</ol>
<p>The required database mode is different for each scenario.</p>
<!-- Comments are visible in the HTML source only -->
