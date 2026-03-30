<h3 dir="auto">Step-by-Step Cleanup (Run Now)</h3>
<p dir="auto">Run these commands <strong>exactly</strong> in sequence as the oracle user:</p>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash</div>
<div>&nbsp;</div>
<div>
<pre tabindex="0"><code>rman target /</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<p dir="auto">Inside RMAN, paste the following block:</p>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>rman</div>
<div>&nbsp;</div>
<div>&nbsp;
<p dir="auto">Please run these exact commands in your current RMAN session:</p>
<p>DELETE NOPROMPT ARCHIVELOG ALL COMPLETED BEFORE 'SYSDATE-7';</p>
<p>CROSSCHECK ARCHIVELOG ALL;<br />DELETE EXPIRED ARCHIVELOG ALL;</p>
<p dir="auto">This is the safe and standard way for a standby database. It will delete all archive logs older than 7 days and free up ~300&ndash;400 GB of space.</p>
<p dir="auto">Then run this to check the space:</p>
<p dir="auto">asmcmd du +DATA/PRODCDB_DR/ARCHIVELOG<br />asmcmd lsdg</p>
</div>
</div>
</div>
</div>
<!-- Comments are visible in the HTML source only -->
