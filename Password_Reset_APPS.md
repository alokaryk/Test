<p>Yes, for what you are trying to do (EBS application cloning/configuration), converting the DR database from <strong>Physical Standby</strong> to <strong>Snapshot Standby</strong> is the correct approach.</p>
<p>Right now:</p>
<pre><code class="language-sql">SQL&gt; SELECT DATABASE_ROLE, OPEN_MODE
     FROM V$DATABASE;

DATABASE_ROLE    OPEN_MODE
---------------- --------------------
PHYSICAL STANDBY READ ONLY
</code></pre>
<p>A Physical Standby in READ ONLY mode has the following limitations:</p>
<ul>
<li>
<p>No DML allowed</p>
</li>
<li>
<p>No updates to EBS metadata tables</p>
</li>
<li>
<p>No APPS password synchronization updates</p>
</li>
<li>
<p><code>adcfgclone.pl appsTier dualfs</code> cannot complete</p>
</li>
<li>
<p>AutoConfig cannot update configuration tables</p>
</li>
</ul>
<p>This explains why you are seeing:</p>
<pre><code class="language-text">RC-40201: Unable to connect to Database PROD
</code></pre>
<p>even though the listener and TNS are working.</p>
<p>Also, since you manually changed:</p>
<pre><code class="language-sql">ALTER USER APPS IDENTIFIED BY "prodpspcl456";
</code></pre>
<p>while the database has been converted back and forth between Snapshot and Physical Standby, I would not rely on that password until the database is back in READ WRITE mode.</p>
<h3>Convert back to Snapshot Standby</h3>
<p>First stop managed recovery:</p>
<pre><code class="language-sql">ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;
</code></pre>
<p>Shutdown:</p>
<pre><code class="language-sql">SHUTDOWN IMMEDIATE;
</code></pre>
<p>Mount:</p>
<pre><code class="language-sql">STARTUP MOUNT;
</code></pre>
<p>Since this is RAC, ensure only one instance is running before conversion:</p>
<pre><code class="language-bash">srvctl stop instance -d PRODCDB_DR -i PRODCDB_DR2
</code></pre>
<p>Verify:</p>
<pre><code class="language-bash">srvctl status database -d PRODCDB_DR
</code></pre>
<p>Expected:</p>
<pre><code class="language-text">PRODCDB_DR running on erpproddb1-dr
PRODCDB_DR2 not running
</code></pre>
<p>Then convert:</p>
<pre><code class="language-sql">ALTER DATABASE CONVERT TO SNAPSHOT STANDBY;
</code></pre>
<p>Open:</p>
<pre><code class="language-sql">ALTER DATABASE OPEN;
</code></pre>
<p>Verify:</p>
<pre><code class="language-sql">SELECT DATABASE_ROLE, OPEN_MODE
FROM V$DATABASE;
</code></pre>
<p>Expected:</p>
<pre><code class="language-text">SNAPSHOT STANDBY   READ WRITE
</code></pre>
<p>Open the PDB:</p>
<pre><code class="language-sql">ALTER SESSION SET CONTAINER=PROD;
ALTER PLUGGABLE DATABASE OPEN;
</code></pre>
<p>Verify:</p>
<pre><code class="language-sql">SHOW PDBS;
</code></pre>
<p>Expected:</p>
<pre><code class="language-text">PROD   READ WRITE
</code></pre>
<h3>Then test APPS</h3>
<p>Reset again if needed:</p>
<pre><code class="language-sql">ALTER SESSION SET CONTAINER=PROD;

ALTER USER APPS IDENTIFIED BY "prodpspcl456";
ALTER USER APPLSYS IDENTIFIED BY "prodpspcl456";
</code></pre>
<p>Test:</p>
<pre><code class="language-bash">sqlplus apps/prodpspcl456@PROD
</code></pre>
<h3>Then rerun clone</h3>
<p>After confirming:</p>
<pre><code class="language-text">DATABASE_ROLE = SNAPSHOT STANDBY
OPEN_MODE     = READ WRITE
PDB PROD      = READ WRITE
</code></pre>
<p>rerun:</p>
<pre><code class="language-bash">cd $COMMON_TOP/clone/bin

perl adcfgclone.pl appsTier dualfs
</code></pre>
<p>Before rerunning, I would also revisit Varun's observation regarding the application Oracle Inventory path. Verify whether the application tier should use:</p>
<pre><code class="language-text">/applprod/fs2/EBSapps/10.1.2/inventory
</code></pre>
<p>instead of:</p>
<pre><code class="language-text">/applprod/oraInventory
</code></pre>
<p>because the earlier OUI registration error strongly points to an inventory registration conflict during Tools 10.1.2 home cloning.</p>
<p>One more thing: when you converted from Snapshot Standby back to Physical Standby earlier, did you use SQL commands only, or was Data Guard Broker involved? Since <code>dg_broker_start</code> is currently FALSE, I want to make sure the exact conversion sequence is correct for your environment before you execute the conversion again.</p>
