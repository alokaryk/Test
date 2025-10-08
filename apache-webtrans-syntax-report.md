<div id="model-response-message-contentr_bbd984cb46bac92a" class="markdown markdown-main-panel enable-updated-hr-color" dir="ltr">
<p>&nbsp;</p>
<h2>Case Report: Apache Startup Failure (Syntax Error) - webtrans.chlsoftech.com</h2>
<p>&nbsp;</p>
<p>This case report details a critical failure preventing the Apache web server (<code>httpd</code>) from starting on the cPanel server, leading to an <strong><code>ERR_CONNECTION_REFUSED</code></strong> error for the domain <code>webtrans.chlsoftech.com</code>.</p>
<table>
<thead>
<tr>
<td>Field</td>
<td>Detail</td>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Case ID</strong></td>
<td>APACHE-WEBTRANS-SYNTAX-0925</td>
</tr>
<tr>
<td><strong>Date Opened</strong></td>
<td>2025-09-25</td>
</tr>
<tr>
<td><strong>Date Closed</strong></td>
<td>2025-09-26</td>
</tr>
<tr>
<td><strong>Server OS</strong></td>
<td>AlmaLinux 8.10 (Cerulean Leopard)</td>
</tr>
<tr>
<td><strong>Affected Service</strong></td>
<td>Apache (httpd.service) / cPanel EasyApache</td>
</tr>
<tr>
<td><strong>Issue Severity</strong></td>
<td>Critical (Full Site Outage)</td>
</tr>
</tbody>
</table>
<hr />
<p>&nbsp;</p>
<h2>I. Problem Description</h2>
<p>&nbsp;</p>
<p>The Apache web server failed to start (<code>Active: failed (Result: exit-code)</code>), immediately resulting in the error <strong><code>ERR_CONNECTION_REFUSED</code></strong> when accessing the website.</p>
<p>Initial investigation via <code>systemctl status httpd -l</code> revealed two distinct, interacting issues:</p>
<ol start="1">
<li>
<p><strong>Syntax Error (Line 25):</strong> An <code>Options</code> directive in a virtual host include file was using inconsistent syntax, causing the configuration parser to crash.</p>
</li>
<li>
<p><strong>Module Loading Error (Fatal):</strong> The crucial <strong>Passenger module</strong> was failing to load or was loading too late, causing the command <code>PassengerEnvVar</code> to be identified as an <code>Invalid command</code> by Apache.</p>
</li>
<li>
<p><strong>File Lock Error:</strong> An immutable attribute (<code>+i</code>) on <code>/etc/apache2/conf/httpd.conf</code> prevented any configuration rebuild attempts.</p>
</li>
</ol>
<hr />
<p>&nbsp;</p>
<h2>II. Root Cause Analysis</h2>
<p>&nbsp;</p>
<p>The failure was a cascade of configuration issues, exacerbated by manual changes:</p>
<ol start="1">
<li>
<p><strong>Primary Cause (Blocking Rebuild):</strong> The file <code>/etc/apache2/conf/httpd.conf</code> had the <strong>immutable (<code>+i</code>) attribute</strong> set, causing the cPanel script <code>/scripts/rebuildhttpdconf</code> to fail with an <strong><code>Operation not permitted</code></strong> error, locking the faulty configuration in place.</p>
</li>
<li>
<p><strong>Secondary Cause (Syntax Crash):</strong> After fixing the file lock, the server crashed due to syntax issues in the user's custom include file: <code>/etc/apache2/conf.d/userdata/std/2_4/webtranschlsofte/webtrans.chlsoftech.com/passenger.conf</code>.</p>
</li>
<li>
<p><strong>Tertiary Cause (Module/Syntax Conflict):</strong> The configuration file was failing because it contained an <code>Options</code> directive with mixed syntax (e.g., <code>Options FollowSymLinks -MultiViews</code>), violating Apache's requirement for uniformity (<code>Options +FollowSymLinks -MultiViews</code>).</p>
</li>
</ol>
<hr />
<p>&nbsp;</p>
<h2>III. Resolution and Steps Taken</h2>
<p>&nbsp;</p>
<p>The resolution required a multi-step approach, ensuring both file permissions and configuration syntax were validated by cPanel scripts.</p>
<table>
<thead>
<tr>
<td>Step</td>
<td>Action Taken</td>
<td>Command(s) Executed</td>
</tr>
</thead>
<tbody>
<tr>
<td><strong>1. Remove Immutable Flag</strong></td>
<td>Removed the immutable attribute from the main configuration file, allowing cPanel to attempt the rebuild.</td>
<td><code>sudo chattr -i /etc/apache2/conf/httpd.conf</code></td>
</tr>
<tr>
<td><strong>2. Fix Options Syntax</strong></td>
<td>Edited the problematic Passenger configuration file to enforce uniform <code>Options</code> syntax, fixing the primary syntax crash.</td>
<td><em>Manual edit of <code>passenger.conf</code> to change:</em> <code>Options FollowSymLinks -MultiViews</code> <em>to</em> <code>Options +FollowSymLinks -MultiViews</code></td>
</tr>
<tr>
<td><strong>3. Fix Module Loading Order</strong></td>
<td>Corrected the Passenger module load order to ensure the necessary module commands were defined before they were used in the Virtual Host includes.</td>
<td><code>sudo mv /etc/apache2/conf.modules.d/passenger.conf /etc/apache2/conf.modules.d/00-passenger.conf</code></td>
</tr>
<tr>
<td><strong>4. Final Configuration Rebuild</strong></td>
<td>Executed the cPanel scripts to integrate all corrected configuration files and restart the web server.</td>
<td><code>sudo /usr/local/cpanel/scripts/rebuildhttpdconf</code> <code>sudo /usr/local/cpanel/scripts/restartsrv_httpd</code></td>
</tr>
</tbody>
</table>
<hr />
<p>&nbsp;</p>
<h2>IV. Conclusion</h2>
<p>&nbsp;</p>
<p>The application of the corrective actions successfully resolved the syntax and module loading errors. The Apache web server (httpd) restarted successfully, the <code>httpd.service</code> status changed to <strong><code>Active: active (running)</code></strong>, and website access was restored.</p>
<p>The primary takeaway is that manual configuration edits on cPanel servers must be meticulously validated, and the immutable attribute should be reviewed on critical system files.</p>
<hr />
<p><strong>File:</strong> <code>apache-webtrans-syntax-report.md</code></p>
</div>
