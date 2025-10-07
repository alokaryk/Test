<p>Here's a solution that combines creating a MySQL user with limited permissions, scheduling daily backups at 1:00 PM and 9:00 PM, and providing a script for the backup process:</p>
<p><strong>1. Create a MySQL User for Backups:</strong></p>
<div class="code-block ng-tns-c1485810703-535 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="3">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-535 ng-star-inserted"><span class="ng-tns-c1485810703-535">Bash</span>
<div class="buttons ng-tns-c1485810703-535 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-535">
<div class="animated-opacity ng-tns-c1485810703-535">
<pre class="ng-tns-c1485810703-535"><code class="code-container formatted ng-tns-c1485810703-535" data-test-id="code-content">mysql -u root -p
</code></pre>
</div>
</div>
</div>
<p>Enter your root password when prompted.</p>
<div class="code-block ng-tns-c1485810703-536 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="5">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-536 ng-star-inserted"><span class="ng-tns-c1485810703-536">SQL</span>
<div class="buttons ng-tns-c1485810703-536 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-536">
<div class="animated-opacity ng-tns-c1485810703-536">
<pre class="ng-tns-c1485810703-536"><code class="code-container formatted ng-tns-c1485810703-536" data-test-id="code-content"><span class="hljs-keyword">CREATE</span> <span class="hljs-keyword">USER</span> backup_user<span class="hljs-variable">@localhost</span> IDENTIFIED <span class="hljs-keyword">BY</span> <span class="hljs-string">'strong_password'</span>;
</code></pre>
</div>
</div>
</div>
<p>Replace <code>'strong_password'</code> with a secure password for the new user.</p>
<p><strong>2. Grant Limited Permissions:</strong></p>
<div class="code-block ng-tns-c1485810703-537 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="7">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-537 ng-star-inserted"><span class="ng-tns-c1485810703-537">SQL</span>
<div class="buttons ng-tns-c1485810703-537 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-537">
<div class="animated-opacity ng-tns-c1485810703-537">
<pre class="ng-tns-c1485810703-537"><code class="code-container formatted ng-tns-c1485810703-537" data-test-id="code-content"><span class="hljs-keyword">GRANT</span> <span class="hljs-keyword">SELECT</span>, LOCK TABLES, FILE <span class="hljs-keyword">ON</span> <span class="hljs-operator">*</span>.<span class="hljs-operator">*</span> <span class="hljs-keyword">TO</span> backup_user<span class="hljs-variable">@localhost</span>;
</code></pre>
</div>
</div>
</div>
<p>This grants the following privileges to <code>backup_user</code>:</p>
<ul>
<li>
<p><code>SELECT</code>: Allows reading data from tables.</p>
</li>
<li>
<p><code>LOCK TABLES</code>: Enables locking tables during the backup process.</p>
</li>
<li>
<p><code>FILE</code>: Grants access to perform file-related operations required for <code>mysqldump</code>.</p>
</li>
</ul>
<p><strong>3. Flush Privileges:</strong></p>
<div class="code-block ng-tns-c1485810703-538 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="9">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-538 ng-star-inserted"><span class="ng-tns-c1485810703-538">SQL</span>
<div class="buttons ng-tns-c1485810703-538 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-538">
<div class="animated-opacity ng-tns-c1485810703-538">
<pre class="ng-tns-c1485810703-538"><code class="code-container formatted ng-tns-c1485810703-538" data-test-id="code-content">FLUSH PRIVILEGES;
</code></pre>
</div>
</div>
</div>
<p><strong>4. Create Backup Script (save as <code>backup.sh</code>):</strong></p>
<div class="code-block ng-tns-c1485810703-539 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="11">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-539 ng-star-inserted"><span class="ng-tns-c1485810703-539">Bash</span>
<div class="buttons ng-tns-c1485810703-539 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-539">
<div class="animated-opacity ng-tns-c1485810703-539">
<pre class="ng-tns-c1485810703-539"><code class="code-container formatted ng-tns-c1485810703-539" data-test-id="code-content"><span class="hljs-meta">#!/bin/bash</span>

<span class="hljs-comment"># Set database name and credentials</span>
DB_NAME=<span class="hljs-string">"prod_db"</span>
DB_USER=<span class="hljs-string">"backup_user"</span>
DB_PASSWORD=<span class="hljs-string">"strong_password"</span>
BACKUP_DIR=<span class="hljs-string">"/path/to/backups"</span>  <span class="hljs-comment"># Replace with your desired backup directory</span>

<span class="hljs-comment"># Get date and timestamp for filenames</span>
DATE=$(date +%Y-%m-%d)
TIMESTAMP=$(date +%H-%M)

<span class="hljs-comment"># Create backup directory if it doesn't exist</span>
mkdir -p <span class="hljs-string">"<span class="hljs-variable">$BACKUP_DIR</span>"</span>

<span class="hljs-comment"># Dump database</span>
mysqldump -u <span class="hljs-string">"<span class="hljs-variable">$DB_USER</span>"</span> -p<span class="hljs-string">"<span class="hljs-variable">$DB_PASSWORD</span>"</span> <span class="hljs-string">"<span class="hljs-variable">$DB_NAME</span>"</span> &gt; <span class="hljs-string">"<span class="hljs-variable">$BACKUP_DIR</span>/<span class="hljs-variable">$DB_NAME</span>-<span class="hljs-variable">$DATE</span>-<span class="hljs-variable">$TIMESTAMP</span>.sql"</span>

<span class="hljs-built_in">echo</span> <span class="hljs-string">"Backup of <span class="hljs-variable">$DB_NAME</span> completed at <span class="hljs-subst">$(date)</span>"</span>
</code></pre>
</div>
</div>
</div>
<p><strong>5. Schedule the Script:</strong></p>
<p>Use cron to schedule the script to run at 1:00 PM and 9:00 PM. Edit the crontab using:</p>
<div class="code-block ng-tns-c1485810703-540 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="13">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-540 ng-star-inserted"><span class="ng-tns-c1485810703-540">Bash</span>
<div class="buttons ng-tns-c1485810703-540 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-540">
<div class="animated-opacity ng-tns-c1485810703-540">
<pre class="ng-tns-c1485810703-540"><code class="code-container formatted ng-tns-c1485810703-540" data-test-id="code-content">crontab -e
</code></pre>
</div>
</div>
</div>
<p>Add the following lines (replace <code>/path/to/backup.sh</code> with the actual path to your script):</p>
<div class="code-block ng-tns-c1485810703-541 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="15">
<div class="formatted-code-block-internal-container ng-tns-c1485810703-541">
<div class="animated-opacity ng-tns-c1485810703-541">
<pre class="ng-tns-c1485810703-541"><code class="code-container formatted ng-tns-c1485810703-541 no-decoration-radius" data-test-id="code-content">0 13,21 * * * /path/to/backup.sh  # Runs at 1:00 PM and 9:00 PM daily
</code></pre>
</div>
</div>
</div>
<p><strong>Explanation of the Script:</strong></p>
<ul>
<li>
<p>The script sets the database name, user credentials, and backup directory path.</p>
</li>
<li>
<p>It gets the current date and timestamp for filenames.</p>
</li>
<li>
<p>It creates the backup directory if it doesn't exist (<code>mkdir -p</code>).</p>
</li>
<li>
<p>It uses <code>mysqldump</code> with the provided credentials to dump the database and stores the backup in a file named with the date and timestamp.</p>
</li>
<li>
<p>The script prints a success message with the current date and time.</p>
</li>
</ul>
<p><strong>Remember:</strong></p>
<ul>
<li>
<p>Replace placeholders like <code>'strong_password'</code> and <code>/path/to/backups</code> with your actual values.</p>
</li>
<li>
<p>Ensure the script has execute permissions (<code>chmod +x backup.sh</code>).</p>
</li>
<li>
<p>Consider rotating backups to manage storage space (e.g., keep only the last week's backups).</p>
</li>
</ul>
<p class="query-text-line ng-star-inserted">root@AppServer:~# bash -x mysql_backup.sh</p>
<p class="query-text-line ng-star-inserted">+ DB_NAME=prod_db</p>
<p class="query-text-line ng-star-inserted">+ DB_USER=backup_user</p>
<p class="query-text-line ng-star-inserted">+ DB_PASSWORD=TA6c98#</p>
<p class="query-text-line ng-star-inserted">+ BACKUP_DIR=/mysql_backup</p>
<p class="query-text-line ng-star-inserted">++ date +%Y-%m-%d</p>
<p class="query-text-line ng-star-inserted">+ DATE=2024-07-19</p>
<p class="query-text-line ng-star-inserted">++ date +%H-%M</p>
<p class="query-text-line ng-star-inserted">+ TIMESTAMP=13-26</p>
<p class="query-text-line ng-star-inserted">+ mkdir -p /mysql_backup</p>
<p class="query-text-line ng-star-inserted">+ mysqldump -u backup_user -pTA6c98# prod_db</p>
<p class="query-text-line ng-star-inserted">mysqldump: [Warning] Using a password on the command line interface can be insecure.</p>
<p class="query-text-line ng-star-inserted">mysqldump: Got error: 1045: Access denied for user 'backup_user'@'localhost' (using password: YES) when trying to connect</p>
<p class="query-text-line ng-star-inserted">++ date</p>
<p class="query-text-line ng-star-inserted">+ echo 'Backup of prod_db completed at Fri Jul 19 01:26:53 PM IST 2024'</p>
<p class="query-text-line ng-star-inserted">Backup of prod_db completed at Fri Jul 19 01:26:53 PM IST 2024</p>
<p class="query-text-line ng-star-inserted">&nbsp;</p>
<p>The output shows the script is running correctly but encountering an access denied error (error 1045) when trying to connect to the MySQL server using <code>backup_user</code>. Here's how to troubleshoot:</p>
<p><strong>1. Verify MySQL User Privileges:</strong></p>
<ul>
<li>
<p>Double-check if you granted the necessary privileges to <code>backup_user</code> using the steps in the previous response. You need <code>SELECT</code>, <code>LOCK TABLES</code>, and <code>FILE</code> privileges for backups.</p>
</li>
<li>
<p>Reconnect to MySQL as <code>root</code> and run:</p>
</li>
</ul>
<div class="code-block ng-tns-c1485810703-542 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="3">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-542 ng-star-inserted"><span class="ng-tns-c1485810703-542">SQL</span>
<div class="buttons ng-tns-c1485810703-542 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-542">
<div class="animated-opacity ng-tns-c1485810703-542">
<pre class="ng-tns-c1485810703-542"><code class="code-container formatted ng-tns-c1485810703-542" data-test-id="code-content"><span class="hljs-keyword">SHOW</span> GRANTS <span class="hljs-keyword">FOR</span> backup_user<span class="hljs-variable">@localhost</span>;
</code></pre>
</div>
</div>
</div>
<p>This should display the granted privileges. Ensure it includes <code>SELECT</code>, <code>LOCK TABLES</code>, and <code>FILE</code>.</p>
<p><strong>2. Check Password:</strong></p>
<ul>
<li>
<p>The script reveals the password (<code>TA6c98#</code>) in the output, which can be insecure. It's recommended to avoid including the password directly in the script. Consider using environment variables or a separate configuration file to store credentials securely.</p>
</li>
<li>
<p>Verify that the password you entered when creating <code>backup_user</code> matches the one used in the script (<code>DB_PASSWORD</code>).</p>
</li>
</ul>
<p><strong>3. MySQL User Connection Method:</strong></p>
<ul>
<li>
<p>While the script uses <code>-p</code> to prompt for the password, it's generally discouraged for security reasons. Explore alternative methods like environment variables or a separate configuration file for password storage.</p>
</li>
</ul>
<p>Here's a revised approach that addresses these points:</p>
<p><strong>Revised Script (save as <code>backup.sh</code>):</strong></p>
<div class="code-block ng-tns-c1485810703-543 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="5">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-543 ng-star-inserted"><span class="ng-tns-c1485810703-543">Bash</span>
<div class="buttons ng-tns-c1485810703-543 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-543">
<div class="animated-opacity ng-tns-c1485810703-543">
<pre class="ng-tns-c1485810703-543"><code class="code-container formatted ng-tns-c1485810703-543" data-test-id="code-content"><span class="hljs-meta">#!/bin/bash</span>

<span class="hljs-comment"># Set database name and credentials (store password securely, not directly in script)</span>
DB_NAME=<span class="hljs-string">"prod_db"</span>
DB_USER=<span class="hljs-string">"backup_user"</span>

<span class="hljs-comment"># Backup directory path</span>
BACKUP_DIR=<span class="hljs-string">"/mysql_backup"</span>

<span class="hljs-comment"># Get date and timestamp for filenames</span>
DATE=$(date +%Y-%m-%d)
TIMESTAMP=$(date +%H-%M)

<span class="hljs-comment"># Create backup directory if it doesn't exist</span>
mkdir -p <span class="hljs-string">"<span class="hljs-variable">$BACKUP_DIR</span>"</span>

<span class="hljs-comment"># Use environment variable or separate configuration file for password</span>
<span class="hljs-comment"># (not shown here)</span>
DB_PASSWORD=$(&lt;path/to/password.txt)  <span class="hljs-comment"># Example (replace with your method)</span>

<span class="hljs-comment"># Dump database</span>
mysqldump -u <span class="hljs-string">"<span class="hljs-variable">$DB_USER</span>"</span> -p<span class="hljs-string">"<span class="hljs-variable">$DB_PASSWORD</span>"</span> <span class="hljs-string">"<span class="hljs-variable">$DB_NAME</span>"</span> &gt; <span class="hljs-string">"<span class="hljs-variable">$BACKUP_DIR</span>/<span class="hljs-variable">$DB_NAME</span>-<span class="hljs-variable">$DATE</span>-<span class="hljs-variable">$TIMESTAMP</span>.sql"</span>

<span class="hljs-built_in">echo</span> <span class="hljs-string">"Backup of <span class="hljs-variable">$DB_NAME</span> completed at <span class="hljs-subst">$(date)</span>"</span>
</code></pre>
</div>
</div>
</div>
<p><strong>Secure Password Storage:</strong></p>
<ul>
<li>
<p>Create a file (e.g., <code>password.txt</code>) to store the password outside the script with appropriate permissions (readable only by you).</p>
</li>
<li>
<p>Update the script to read the password from this file using <code>$(&lt;path/to/password.txt)</code>.</p>
</li>
<li>
<p>Consider using tools like <code>keychain</code> or system-specific password managers for more secure storage.</p>
</li>
</ul>
<p>Remember to adjust the script paths and password storage method based on your environment. Once the script is updated and MySQL user privileges are verified, you can re-run the script or schedule it using cron as mentioned before.</p>
