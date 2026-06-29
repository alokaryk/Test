<p>This confirms the diagnosis. The problem was <strong>never the APPS password</strong>&mdash;it was that the <strong><code>ebs_PROD</code> service was not active</strong>, so the listener could not route connections to the database.</p>
<h3>What changed</h3>
<p>Initially:</p>
<ul>
<li>
<p><code>sqlplus apps/prodpspcl456@PROD</code> returned:</p>
<pre><code>ORA-12514: TNS:listener does not currently know of service requested in connect descriptor
</code></pre>
</li>
<li>
<p><code>lsnrctl services</code> did <strong>not</strong> list <code>ebs_PROD</code>.</p>
</li>
<li>
<p><code>GV$ACTIVE_SERVICES</code> returned no rows for <code>ebs_PROD</code>.</p>
</li>
<li>
<p>EBS utilities (<code>txkSetADBConfig.pl</code>, <code>txkCfgUtlfileDir.pl</code>, AutoConfig) reported:</p>
<pre><code>Invalid APPS database user credentials
</code></pre>
<p>because they could not establish a database connection.</p>
</li>
</ul>
<hr />
<h3>What your latest output proves</h3>
<p>You checked:</p>
<pre><code class="language-sql">SELECT name,
       network_name,
       pdb,
       enabled
FROM dba_services
WHERE upper(name)='EBS_PROD';
</code></pre>
<p>Output:</p>
<pre><code>NAME          : ebs_PROD
NETWORK_NAME  : ebs_PROD
PDB           : PROD
ENABLED       : NO
</code></pre>
<p>The important point here is that <strong>ENABLED=NO does not mean the service cannot be started</strong>. It means it is not configured for automatic management. The service object exists in the data dictionary.</p>
<p>You then executed:</p>
<pre><code class="language-sql">EXEC DBMS_SERVICE.START_SERVICE('ebs_PROD');

ALTER SYSTEM REGISTER;
</code></pre>
<p>After that, the listener shows:</p>
<pre><code>Service "ebs_PROD" has 1 instance(s).

Instance "PRODCDB_DR"
status READY
</code></pre>
<p>This is exactly what was missing before.</p>
<hr />
<h2>Now test from the application node</h2>
<p>Run exactly:</p>
<pre><code class="language-bash">sqlplus apps/prodpspcl456@PROD
</code></pre>
<p>It should now connect successfully.</p>
<p>If it does, you've confirmed that the earlier "Invalid APPS credentials" errors were actually caused by <code>ORA-12514</code>.</p>
<hr />
<h2>Next steps</h2>
<p>Now rerun the failed utilities in order:</p>
<pre><code class="language-bash">txkSetADBConfig.pl
</code></pre>
<p>Then</p>
<pre><code class="language-bash">txkCfgUtlfileDir.pl
</code></pre>
<p>Then continue with</p>
<pre><code class="language-bash">adcfgclone.pl appsTier
</code></pre>
<p>or whichever cloning stage failed previously.</p>
<hr />
<h2>One important thing still needs to be fixed</h2>
<p>Although the service is running now, notice this:</p>
<pre><code>ENABLED = NO
</code></pre>
<p>and you had to manually execute:</p>
<pre><code class="language-sql">EXEC DBMS_SERVICE.START_SERVICE('ebs_PROD');
</code></pre>
<p>That means after the next database restart, <strong><code>ebs_PROD</code> will likely disappear again</strong> unless it is configured for automatic startup.</p>
<p>Since this is a RAC/Grid Infrastructure environment, the preferred solution is <strong>not</strong> to rely on <code>DBMS_SERVICE.START_SERVICE</code>, but to register the service with Clusterware using <code>srvctl</code>.</p>
<p>Please run these commands on the database server:</p>
<pre><code class="language-bash">srvctl config service -d PRODCDB_DR
</code></pre>
<p>or, if nothing is returned,</p>
<pre><code class="language-bash">srvctl config service -d PRODCDB
</code></pre>
<p>If <code>ebs_PROD</code> is not listed, it should be added and configured with <code>srvctl</code> so that it starts automatically whenever the database starts.</p>
<hr />
<h3>Conclusion</h3>
<p>You have identified the root cause:</p>
<ul>
<li>
<p>✅ APPS password is valid.</p>
</li>
<li>
<p>✅ Listener is healthy.</p>
</li>
<li>
<p>✅ <code>tnsnames.ora</code> is correct.</p>
</li>
<li>
<p>✅ <code>ebs_PROD</code> service was <strong>not registered with the listener</strong>, causing <code>ORA-12514</code>.</p>
</li>
<li>
<p>✅ Manually starting <code>ebs_PROD</code> and issuing <code>ALTER SYSTEM REGISTER</code> resolved the connectivity issue.</p>
</li>
</ul>
<p>The remaining task is to make the <code>ebs_PROD</code> service persistent across database restarts using <code>srvctl</code> (or the site's standard service management procedure).</p>
