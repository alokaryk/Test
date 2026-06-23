<p>Yes, what Varun is suggesting is the standard approach for an Oracle Data Guard DR drill when you need to run applications against the DR database.</p>
<p>Your current DR database is a <strong>Physical Standby</strong> with the PDB opened in <strong>READ ONLY</strong> mode. Oracle EBS cloning and application configuration generally require the database to be opened <strong>READ WRITE</strong>, which is not possible on a Physical Standby. Converting it to a <strong>Snapshot Standby</strong> allows temporary READ WRITE access while preserving the Data Guard configuration.</p>
<h3>Before Proceeding</h3>
<p>You should confirm the following with the DBA team:</p>
<ol>
<li>
<p>DR Drill window has been approved.</p>
</li>
<li>
<p>Production team understands that redo apply will stop during the drill.</p>
</li>
<li>
<p>Sufficient FRA (Fast Recovery Area) space exists because flashback logs will be generated.</p>
</li>
<li>
<p>Flashback Database is enabled.</p>
</li>
<li>
<p>Oracle DBA will perform the conversion (not application team).</p>
</li>
</ol>
<h3>Verify Current Status</h3>
<pre><code class="language-sql">SELECT DATABASE_ROLE, OPEN_MODE
FROM V$DATABASE;
</code></pre>
<p>Current output should be something similar to:</p>
<pre><code class="language-sql">DATABASE_ROLE     OPEN_MODE
----------------  --------------------
PHYSICAL STANDBY  READ ONLY WITH APPLY
</code></pre>
<p>or</p>
<pre><code class="language-sql">PHYSICAL STANDBY  READ ONLY
</code></pre>
<hr />
<h2>Step 1: Stop Managed Recovery</h2>
<p>On the standby database:</p>
<pre><code class="language-sql">ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;
</code></pre>
<p>Verify:</p>
<pre><code class="language-sql">SELECT PROCESS, STATUS
FROM V$MANAGED_STANDBY;
</code></pre>
<p>MRP process should stop.</p>
<hr />
<h2>Step 2: Convert Physical Standby to Snapshot Standby</h2>
<pre><code class="language-sql">ALTER DATABASE CONVERT TO SNAPSHOT STANDBY;
</code></pre>
<p>Shutdown and restart:</p>
<pre><code class="language-sql">SHUTDOWN IMMEDIATE;
STARTUP;
</code></pre>
<p>Verify:</p>
<pre><code class="language-sql">SELECT DATABASE_ROLE, OPEN_MODE
FROM V$DATABASE;
</code></pre>
<p>Expected:</p>
<pre><code class="language-sql">SNAPSHOT STANDBY   READ WRITE
</code></pre>
<hr />
<h2>&nbsp;</h2>
<h3 data-section-id="5sy56e" data-start="457" data-end="517">First, verify you are connected to the correct container</h3>
<p data-start="519" data-end="523">Run:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="contents">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="relative h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="">
<div class="relative">
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<pre class="cm-content q9tKkq_readonly m-0"><code>SHOW CON_NAME;</code></pre>
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
</div>
</div>
</div>
<p data-start="564" data-end="566">or</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="contents">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="relative h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="">
<div class="relative">
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<pre class="cm-content q9tKkq_readonly m-0"><code><span class="ͼg">SELECT</span> SYS_CONTEXT(<span class="ͼk">'USERENV'</span>,<span class="ͼk">'CON_NAME'</span>) <span class="ͼg">FROM</span> DUAL;</code></pre>
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
</div>
</div>
</div>
<p data-start="644" data-end="653">Expected:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="contents">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="relative">
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<pre class="cm-content q9tKkq_readonly m-0"><code>PROD</code></pre>
</div>
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
</div>
<p data-start="685" data-end="697">If it shows:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="contents">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1">&nbsp;</div>
<div class="relative">
<div class="pe-11 pt-3">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<pre class="cm-content q9tKkq_readonly m-0"><code>CDB$ROOT</code></pre>
</div>
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
</div>
<p data-start="733" data-end="811">then your query was executed in the root container, where APPS will not exist.</p>
<p data-start="813" data-end="826">In that case:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="contents">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="relative h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="">
<div class="relative">
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<pre class="cm-content q9tKkq_readonly m-0"><code><span class="ͼg">ALTER</span> <span class="ͼg">SESSION</span> <span class="ͼg">SET</span> CONTAINER<span class="ͼg">=</span>PROD;</code></pre>
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
</div>
</div>
</div>
<p data-start="886" data-end="897">then rerun:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="contents">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="relative h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="">
<div class="relative">
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<pre class="cm-content q9tKkq_readonly m-0"><code><span class="ͼg">SELECT</span> USERNAME, ACCOUNT_STATUS<br /><span class="ͼg">FROM</span> DBA_USERS<br /><span class="ͼg">WHERE</span> USERNAME<span class="ͼg">=</span><span class="ͼk">'APPS'</span>;</code></pre>
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
</div>
</div>
</div>
<h2>Step 3: Open PDB in Read Write Mode</h2>
<p>If this is a multitenant database:</p>
<pre><code class="language-sql">ALTER PLUGGABLE DATABASE PROD OPEN;
</code></pre>
<p>Verify:</p>
<pre><code class="language-sql">SHOW PDBS;
</code></pre>
<p>Expected:</p>
<pre><code class="language-sql">PROD    READ WRITE
</code></pre>
<hr />
<h2>Step 4: Configure EBS Application</h2>
<p>At this stage:</p>
<ul>
<li>
<p>Run adcfgclone.pl</p>
</li>
<li>
<p>Configure application tier</p>
</li>
<li>
<p>Validate APPS connectivity</p>
</li>
<li>
<p>Complete DR drill testing</p>
</li>
</ul>
<p>You can also reset APPS password if required because the database is now writable.</p>
<hr />
<h2>Step 5: Cleanup After DR Drill</h2>
<p>Once testing is complete:</p>
<p>Stop application services.</p>
<p>Close PDB:</p>
<pre><code class="language-sql">ALTER PLUGGABLE DATABASE PROD CLOSE IMMEDIATE;
</code></pre>
<p>Shutdown database:</p>
<pre><code class="language-sql">SHUTDOWN IMMEDIATE;
</code></pre>
<p>Start mount:</p>
<pre><code class="language-sql">STARTUP MOUNT;
</code></pre>
<hr />
<h2>Step 6: Convert Snapshot Standby Back to Physical Standby</h2>
<pre><code class="language-sql">ALTER DATABASE CONVERT TO PHYSICAL STANDBY;
</code></pre>
<p>Restart:</p>
<pre><code class="language-sql">SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
</code></pre>
<hr />
<h2>Step 7: Resume Redo Apply</h2>
<pre><code class="language-sql">ALTER DATABASE RECOVER MANAGED STANDBY DATABASE
USING CURRENT LOGFILE DISCONNECT;
</code></pre>
<p>or for 19c:</p>
<pre><code class="language-sql">ALTER DATABASE RECOVER MANAGED STANDBY DATABASE
DISCONNECT FROM SESSION;
</code></pre>
<hr />
<h2>Step 8: Verify Synchronization</h2>
<pre><code class="language-sql">SELECT DATABASE_ROLE, OPEN_MODE
FROM V$DATABASE;
</code></pre>
<p>Expected:</p>
<pre><code class="language-sql">PHYSICAL STANDBY   READ ONLY WITH APPLY
</code></pre>
<p>Check lag:</p>
<pre><code class="language-sql">SELECT NAME, VALUE, UNIT
FROM V$DATAGUARD_STATS
WHERE NAME IN ('transport lag','apply lag');
</code></pre>
<hr />
<p>SELECT DATABASE_ROLE, OPEN_MODE, FLASHBACK_ON FROM V$DATABASE;</p>
<p>SELECT<br />SPACE_LIMIT/1024/1024/1024 FRA_SIZE_GB,<br />SPACE_USED/1024/1024/1024 USED_GB<br />FROM V$RECOVERY_FILE_DEST;</p>
<p>SELECT NAME, VALUE<br />FROM V$DATAGUARD_STATS<br />WHERE NAME IN ('transport lag','apply lag');</p>
<!-- Comments are visible in the HTML source only -->
