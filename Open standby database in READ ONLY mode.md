<p data-start="0" data-end="38">This is correct and expected behavior.</p>
<p data-start="40" data-end="54">What happened:</p>
<ol data-start="56" data-end="75">
<li data-section-id="1kwm5ev" data-start="56" data-end="75">Database was in:</li>
</ol>
<div class="relative w-full mt-4 mb-1">
<div class="">
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
<pre class="cm-content q9tKkq_readonly m-0"><code>READ ONLY PHYSICAL STANDBY</code></pre>
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
<ol start="2" data-start="128" data-end="144">
<li data-section-id="da9yz" data-start="128" data-end="144">You executed:</li>
</ol>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="relative">
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<pre class="cm-content q9tKkq_readonly m-0"><code>SHUTDOWN <span class="ͼg">IMMEDIATE</span>;</code></pre>
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
<p data-start="189" data-end="232">So the standby instance stopped completely.</p>
<ol start="3" data-start="234" data-end="242">
<li data-section-id="136u2hh" data-start="234" data-end="242">Then:</li>
</ol>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="relative">
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<pre class="cm-content q9tKkq_readonly m-0"><code>STARTUP MOUNT;</code></pre>
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
<p data-start="282" data-end="332">Now standby is back to its normal Data Guard mode:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
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
<pre class="cm-content q9tKkq_readonly m-0"><code>MOUNTED PHYSICAL STANDBY</code></pre>
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
<p data-start="384" data-end="442">This is the correct final state for standby after testing.</p>
<p data-start="444" data-end="490">Now you should restart managed recovery again:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="relative">
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<pre class="cm-content q9tKkq_readonly m-0"><code><span class="ͼg">ALTER</span> DATABASE RECOVER MANAGED STANDBY DATABASE<br /><span class="ͼg">USING</span> <span class="ͼg">CURRENT</span> LOGFILE<br /><span class="ͼg">DISCONNECT</span> <span class="ͼg">FROM</span> <span class="ͼg">SESSION</span>;</code></pre>
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
<p data-start="611" data-end="623">Then verify:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="pointer-events-none absolute inset-x-4 top-12 bottom-4">&nbsp;</div>
<div class="relative">
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<pre class="cm-content q9tKkq_readonly m-0"><code><span class="ͼg">SELECT</span> process,status,thread#,sequence#<br /><span class="ͼg">FROM</span> gv$managed_standby<br /><span class="ͼg">ORDER</span> <span class="ͼg">BY</span> process;</code></pre>
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
<p data-start="731" data-end="735">and:</p>
<p data-start="731" data-end="735">&nbsp;</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
<div class="relative">
<div class="h-full min-h-0 min-w-0">
<div class="h-full min-h-0 min-w-0">
<div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl">
<div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback">
<div class="relative">
<div class="">
<div class="relative z-0 flex max-w-full">
<div id="code-block-viewer" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr" dir="ltr">
<div class="cm-scroller">
<pre class="cm-content q9tKkq_readonly m-0"><code><span class="ͼg">SELECT</span> open_mode,database_role<br /><span class="ͼg">FROM</span> v$database;</code></pre>
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
<p data-start="809" data-end="818">Expected:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
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
<pre class="cm-content q9tKkq_readonly m-0"><code>OPEN_MODE     DATABASE_ROLE<br />------------  ----------------<br />MOUNTED       PHYSICAL STANDBY</code></pre>
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
<p data-start="935" data-end="940">with:</p>
<div class="relative w-full mt-4 mb-1">
<div class="">
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
<pre class="cm-content q9tKkq_readonly m-0"><code>MRP0 APPLYING_LOG</code></pre>
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
<p data-start="984" data-end="1019" data-is-last-node="" data-is-only-node="">showing redo apply is active again.</p>
<p data-start="984" data-end="1019" data-is-last-node="" data-is-only-node="">&nbsp;</p>
<p>You can safely perform this on the DR standby database for the DR drill activity.<br />Below are the exact steps to execute carefully on <code>erpproddb1-dr</code> as oracle user.</p>
<p>First verify current database mode:</p>
<pre><code class="language-sql">sqlplus / as sysdba

SELECT open_mode,database_role FROM v$database;
</code></pre>
<p>If database is in <code>MOUNTED</code> mode, stop managed recovery first:</p>
<pre><code class="language-sql">ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;
</code></pre>
<p>Open standby database in READ ONLY mode:</p>
<pre><code class="language-sql">ALTER DATABASE OPEN READ ONLY;
</code></pre>
<p>Now switch to the PDB:</p>
<pre><code class="language-sql">ALTER SESSION SET CONTAINER=PROD;
</code></pre>
<p>Verify current container:</p>
<pre><code class="language-sql">SHOW CON_NAME;
</code></pre>
<p>Expected output:</p>
<pre><code class="language-sql">PROD
</code></pre>
<p>Open PDB explicitly in READ ONLY mode:</p>
<pre><code class="language-sql">ALTER PLUGGABLE DATABASE PROD OPEN READ ONLY;
</code></pre>
<p>Now create the service:</p>
<pre><code class="language-sql">BEGIN
  DBMS_SERVICE.CREATE_SERVICE(
    service_name =&gt; 'prod',
    network_name =&gt; 'prod');
END;
/
</code></pre>
<p>If service already exists, you may get:<br /><code>ORA-44303: service name exists</code></p>
<p>In that case ignore and continue.</p>
<p>Start the service:</p>
<pre><code class="language-sql">EXEC DBMS_SERVICE.START_SERVICE('prod');
</code></pre>
<p>Force listener registration:</p>
<pre><code class="language-sql">ALTER SYSTEM REGISTER;
</code></pre>
<p>Now verify service registration from OS prompt:</p>
<pre><code class="language-bash">lsnrctl status
</code></pre>
<p>You should now see something similar:</p>
<pre><code class="language-text">Service "prod" has 1 instance(s).
  Instance "PRODCDB_DR", status READY
</code></pre>
<p>Then test locally from DR DB server:</p>
<pre><code class="language-bash">sqlplus apps/password@//10.61.185.221:1521/prod
</code></pre>
<p>And from application server:</p>
<pre><code class="language-bash">sqlplus apps/password@PRODCDB
</code></pre>
<p>Important:</p>
<ul>
<li>
<p>Since this is a physical standby, these service changes are temporary and mainly for DR drill/application connectivity.</p>
</li>
<li>
<p>After DR activity completes and standby is restored back to managed recovery mode, you may need to recreate/start the service again after restart.</p>
</li>
</ul>
<p>After application validation is completed, re-enable managed recovery:</p>
<pre><code class="language-sql">ALTER DATABASE RECOVER MANAGED STANDBY DATABASE
USING CURRENT LOGFILE
DISCONNECT FROM SESSION;
</code></pre>
<p>Finally verify recovery:</p>
<pre><code class="language-sql">SELECT process,status,thread#,sequence#
FROM v$managed_standby;
</code></pre>
<p data-start="984" data-end="1019" data-is-last-node="" data-is-only-node="">&nbsp;</p>
<p data-start="984" data-end="1019" data-is-last-node="" data-is-only-node="">&nbsp;</p>
<p>No, that is not a proper database shutdown.</p>
<p>The message:</p>
<pre><code class="language-text">Pluggable Database closed.
</code></pre>
<p>means you were connected inside the PDB (<code>PROD</code>), not in <code>CDB$ROOT</code>.</p>
<p>So only the PDB got closed &mdash; the standby database instance is still running.</p>
<p>First switch back to root container:</p>
<pre><code class="language-sql">SHOW CON_NAME;
</code></pre>
<p>If it shows:</p>
<pre><code class="language-text">PROD
</code></pre>
<p>then run:</p>
<pre><code class="language-sql">ALTER SESSION SET CONTAINER=CDB$ROOT;
</code></pre>
<p>Verify:</p>
<pre><code class="language-sql">SHOW CON_NAME;
</code></pre>
<p>It should show:</p>
<pre><code class="language-text">CDB$ROOT
</code></pre>
<p>Then stop managed recovery if running:</p>
<pre><code class="language-sql">ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;
</code></pre>
<p>Now shutdown the standby database properly:</p>
<pre><code class="language-sql">SHUTDOWN IMMEDIATE;
</code></pre>
<p>You should see messages like:</p>
<pre><code class="language-text">Database closed.
Database dismounted.
ORACLE instance shut down.
</code></pre>
<p>Then mount again:</p>
<pre><code class="language-sql">STARTUP MOUNT;
</code></pre>
<p>Restart managed recovery:</p>
<pre><code class="language-sql">ALTER DATABASE RECOVER MANAGED STANDBY DATABASE
USING CURRENT LOGFILE
DISCONNECT FROM SESSION;
</code></pre>
<p>Finally verify:</p>
<pre><code class="language-sql">SELECT open_mode,database_role FROM v$database;
</code></pre>
<p>Expected:</p>
<pre><code class="language-text">MOUNTED   PHYSICAL STANDBY
</code></pre>
<p>and:</p>
<pre><code class="language-sql">SELECT process,status,thread#,sequence#
FROM v$managed_standby;
</code></pre>
<p>should show <code>MRP0 APPLYING_LOG</code>.</p>
<!-- Comments are visible in the HTML source only -->
