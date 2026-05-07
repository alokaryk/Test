<h3 dir="auto">Is it mandatory to connect with sqlplus before running adcfgclone.pl?</h3>
<p dir="auto"><strong>Yes, it is practically mandatory.</strong></p>
<p dir="auto">The adcfgclone.pl script connects to the target database (PRODCDB_DR) multiple times during execution to:</p>
<ul dir="auto">
<li>Validate the APPS user</li>
<li>Read schema information</li>
<li>Generate context files</li>
</ul>
<p dir="auto">If it cannot connect, the cloning will fail.</p>
<hr />
<h3 dir="auto">Current Issue</h3>
<p dir="auto">You are getting ORA-12154: TNS:could not resolve the connect identifier because there is <strong>no TNS entry</strong> for PRODCDB_DR on the application server.</p>
<h3 dir="auto">Solution: Create TNS Entry</h3>
<p dir="auto">Run the following commands as oracle user:</p>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>su - oracle

# Create TNS directory if not exists
mkdir -p /applprod/fs1/inst/apps/PRODCDB_DR_erpapp-dr/ora/10.1.0/network/admin

# Create tnsnames.ora
cat &gt; /applprod/fs1/inst/apps/PRODCDB_DR_erpapp-dr/ora/10.1.0/network/admin/tnsnames.ora &lt;&lt; EOF
PRODCDB_DR =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = erpproddb-scan)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = PRODCDB_DR)
    )
  )

EOF

# Set TNS_ADMIN
export TNS_ADMIN=/applprod/fs1/inst/apps/PRODCDB_DR_erpapp-dr/ora/10.1.0/network/admin
echo "export TNS_ADMIN=$TNS_ADMIN" &gt;&gt; ~/.bash_profile</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto">Test Connectivity Again</h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>sqlplus apps/sys123@PRODCDB_DR</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<p dir="auto">If it connects successfully, you can proceed with cloning.</p>
<p dir="auto">&nbsp;</p>
<p dir="auto">&nbsp;</p>
<p>[oracle@erpapp-dr ~]$ cat .bash_profile<br /># .bash_profile</p>
<p># Get the aliases and functions<br />if [ -f ~/.bashrc ]; then<br /> . ~/.bashrc<br />fi</p>
<p># User specific environment and startup programs</p>
<p>PATH=$PATH:$HOME/.local/bin:$HOME/bin</p>
<p>export PATH<br />export APPL_TOP=/applprod/fs1/EBSapps/appl<br />export COMMON_TOP=/applprod/fs1/EBSapps/comn<br />export ORACLE_HOME=/applprod/fs1/EBSapps/10.1.2<br />export PATH=$ORACLE_HOME/bin:$PATH<br />alias adcfg='cd /applprod/fs1/EBSapps/comn/clone/bin'</p>
<p># Correct Oracle 10.1.2 Environment for EBS<br />export ORACLE_HOME=/applprod/fs1/EBSapps/10.1.2<br />export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH<br />export PATH=$ORACLE_HOME/bin:$PATH<br />export TNS_ADMIN=/applprod/fs1/inst/apps/PRODCDB_DR_erpapp-dr/ora/10.1.0/network/admin</p>
<p>[oracle@erpapp-dr ~]$</p>
<!-- Comments are visible in the HTML source only -->
