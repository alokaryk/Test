<p dir="auto"><strong>Excellent Interview-Oriented Note on ASM Diskgroup Redundancy</strong></p>
<h3 dir="auto"><strong>ASM Redundancy Types in Oracle</strong></h3>
<p dir="auto">In Oracle ASM (Automatic Storage Management), <strong>Redundancy</strong> defines how many copies of data are maintained for protection against disk failure.</p>
<div>
<div>
<div dir="auto">
<table dir="auto">
<thead>
<tr>
<th data-col-size="md">Redundancy Type</th>
<th data-col-size="xs">Copies of Data</th>
<th data-col-size="lg">Failure Tolerance</th>
<th data-col-size="xs">Minimum Disks Required</th>
<th data-col-size="xl">Recommended Use Case</th>
<th data-col-size="lg">Common in</th>
</tr>
</thead>
<tbody>
<tr>
<td data-col-size="md"><strong>EXTERNAL</strong></td>
<td data-col-size="xs">1</td>
<td data-col-size="lg">0 (No mirroring by ASM)</td>
<td data-col-size="xs">1</td>
<td data-col-size="xl">When storage already has RAID (SAN/RAID-5/6)</td>
<td data-col-size="lg"><strong>DR Servers, Test/Dev, Single Node</strong></td>
</tr>
<tr>
<td data-col-size="md"><strong>NORMAL</strong></td>
<td data-col-size="xs">2</td>
<td data-col-size="lg">1 disk failure</td>
<td data-col-size="xs">2</td>
<td data-col-size="xl">Production environments (Balanced)</td>
<td data-col-size="lg">Most Production RAC</td>
</tr>
<tr>
<td data-col-size="md"><strong>HIGH</strong></td>
<td data-col-size="xs">3</td>
<td data-col-size="lg">2 disk failures</td>
<td data-col-size="xs">3</td>
<td data-col-size="xl">Very critical systems</td>
<td data-col-size="lg">High Availability</td>
</tr>
</tbody>
</table>
</div>
</div>
<div>&nbsp;</div>
<div>&nbsp;</div>
</div>
<hr />
<h3 dir="auto"><strong>Key Interview Questions &amp; Answers</strong></h3>
<p dir="auto"><strong>1. What is the difference between EXTERNAL and NORMAL redundancy?</strong></p>
<ul dir="auto">
<li><strong>EXTERNAL</strong>: ASM does <strong>not</strong> mirror the data. It relies on the underlying storage array (SAN) for redundancy (RAID). &rarr; Fastest performance, less space consumption. &rarr; Used when storage already provides high redundancy.</li>
<li><strong>NORMAL</strong>: ASM maintains <strong>2 copies</strong> (mirroring) of every extent. &rarr; Can survive 1 disk failure. &rarr; Most commonly used in production RAC environments.</li>
</ul>
<p dir="auto"><strong>2. Which redundancy you used in your recent DR project and why?</strong></p>
<blockquote dir="auto">
<p dir="auto">In our recent DR project for INFRADB (Single Node), we used <strong>EXTERNAL</strong> redundancy because:</p>
<ul dir="auto">
<li>It is a Single Node setup.</li>
<li>We are using local disks (not shared SAN with RAID).</li>
<li>Performance was prioritized and we had limited disks.</li>
<li>For DR, we accept slightly lower protection level than Production.</li>
</ul>
</blockquote>
<p dir="auto"><strong>3. What is the first redundancy you should choose and why?</strong></p>
<ul dir="auto">
<li><strong>EXTERNAL</strong> is usually the <strong>first choice</strong> for:
<ul dir="auto">
<li>Single Node / Standalone / Oracle Restart setups</li>
<li>DR Servers</li>
<li>When using RAID-enabled storage</li>
</ul>
</li>
</ul>
<p dir="auto"><strong>4. What are the pros &amp; cons?</strong></p>
<p dir="auto"><strong>EXTERNAL Redundancy:</strong></p>
<ul dir="auto">
<li>Pros: Better performance, less space used</li>
<li>Cons: No protection if a disk fails (depends on storage RAID)</li>
</ul>
<p dir="auto"><strong>NORMAL Redundancy:</strong></p>
<ul dir="auto">
<li>Pros: Good protection (1 disk failure)</li>
<li>Cons: Uses more space (almost double), slightly slower write performance</li>
</ul>
<hr />
<h3 dir="auto"><strong>One-Liner for Interview</strong></h3>
<blockquote dir="auto">
<p dir="auto">"In ASM, <strong>EXTERNAL</strong> redundancy means no mirroring by ASM (relies on storage RAID), while <strong>NORMAL</strong> provides two-way mirroring and can tolerate one disk failure. For Single Node or DR environments, EXTERNAL is commonly preferred for better performance and space utilization."</p>
</blockquote>
<p dir="auto"><strong>Oracle Restart Setup &ndash; Complete Discussion</strong></p>
<h3 dir="auto"><strong>What is Oracle Restart?</strong></h3>
<p dir="auto"><strong>Oracle Restart</strong> is a high-availability feature introduced in Oracle 11gR2 and enhanced in 19c. It automatically <strong>starts and restarts</strong> Oracle components (Database, Listener, ASM, etc.) in case of failure or server reboot.</p>
<p dir="auto">It is the <strong>single-node version</strong> of Oracle Clusterware (without full RAC).</p>
<hr />
<h3 dir="auto"><strong>Why Oracle Restart?</strong></h3>
<div>
<div>
<div dir="auto">
<table dir="auto">
<thead>
<tr>
<th data-col-size="md">Feature</th>
<th data-col-size="md">Oracle Restart</th>
<th data-col-size="lg">Full RAC Clusterware</th>
</tr>
</thead>
<tbody>
<tr>
<td data-col-size="md">Number of Nodes</td>
<td data-col-size="md">Single Node</td>
<td data-col-size="lg">Multiple Nodes</td>
</tr>
<tr>
<td data-col-size="md">High Availability</td>
<td data-col-size="md">Yes (Restart on failure)</td>
<td data-col-size="lg">Yes (Failover + Restart)</td>
</tr>
<tr>
<td data-col-size="md">ASM Management</td>
<td data-col-size="md">Yes</td>
<td data-col-size="lg">Yes</td>
</tr>
<tr>
<td data-col-size="md">SCAN Listener</td>
<td data-col-size="md">Yes</td>
<td data-col-size="lg">Yes</td>
</tr>
<tr>
<td data-col-size="md">Cost</td>
<td data-col-size="md">Included with Database license</td>
<td data-col-size="lg">Requires RAC License</td>
</tr>
<tr>
<td data-col-size="md">Use Case</td>
<td data-col-size="md">Standalone / DR Servers</td>
<td data-col-size="lg">Production Multi-Node RAC</td>
</tr>
</tbody>
</table>
</div>
</div>
<div>&nbsp;</div>
<div>&nbsp;</div>
</div>
<hr />
<h3 dir="auto"><strong>Key Components of Oracle Restart</strong></h3>
<ol dir="auto">
<li><strong>OHASD</strong> (Oracle High Availability Services Daemon) &ndash; Main daemon</li>
<li><strong>CRSCTL</strong> &ndash; Command line tool to manage resources</li>
<li><strong>SRVCTL</strong> &ndash; Server Control utility</li>
<li><strong>ASM</strong> &ndash; Automatic Storage Management</li>
<li><strong>Listener &amp; Database</strong> &ndash; Automatically managed</li>
</ol>
<hr />
<h3 dir="auto"><strong>Advantages of Oracle Restart</strong></h3>
<ul dir="auto">
<li>Automatic startup of ASM, Database, and Listener after server reboot.</li>
<li>Automatic restart of failed components.</li>
<li>Centralized management using srvctl and crsctl.</li>
<li>Easier maintenance than manual scripts.</li>
<li>Good for <strong>DR Servers</strong>, Test/Dev, and Single Node Production.</li>
</ul>
<hr />
<h3 dir="auto"><strong>Oracle Restart Setup &ndash; High Level Steps</strong></h3>
<ol dir="auto">
<li><strong>Install Grid Infrastructure for Standalone Cluster</strong> (Oracle Restart)</li>
<li><strong>Configure ASM Disk Groups</strong></li>
<li><strong>Install Database Software</strong></li>
<li><strong>Create Database using DBCA</strong> (with Oracle Restart enabled)</li>
<li><strong>Register Database with Oracle Restart</strong> using srvctl</li>
</ol>
<hr />
<h3 dir="auto"><strong>Important Interview Questions &amp; Answers</strong></h3>
<p dir="auto"><strong>Q1. What is the difference between Oracle Restart and Oracle RAC?</strong> <strong>Ans:</strong> Oracle Restart is for <strong>single node</strong> high availability. Oracle RAC is for <strong>multi-node</strong> clustering with failover and load balancing.</p>
<p dir="auto"><strong>Q2. How do you check if Oracle Restart is configured?</strong></p>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>crsctl check has
crsctl stat res -t
srvctl status database -d &lt;dbname&gt;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<p dir="auto"><strong>Q3. How do you enable/disable Oracle Restart for a database?</strong></p>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code># Enable
srvctl add database -db &lt;dbname&gt; -oraclehome $ORACLE_HOME

# Start
srvctl start database -db &lt;dbname&gt;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<p dir="auto"><strong>Q4. What is the main process of Oracle Restart?</strong> <strong>OHASD</strong> (Oracle High Availability Services Daemon)</p>
<p dir="auto"><strong>Q5. In which scenario did you use Oracle Restart?</strong> &rarr; In DR environment for <strong>INFRADB DR</strong> (Single Node), we installed Grid Infrastructure as Standalone Cluster (Oracle Restart) so that ASM, Listener, and Database automatically start after server reboot or failure.</p>
<hr />
<p dir="auto"><strong>Comprehensive Interview-Ready Note on ASM Redundancy</strong></p>
<h3 dir="auto"><strong>1. ASM Redundancy Types (Summary)</strong></h3>
<div>
<div>
<div dir="auto">
<table dir="auto">
<thead>
<tr>
<th data-col-size="lg">Redundancy Type</th>
<th data-col-size="xs">No. of Copies</th>
<th data-col-size="xs">Survives Disk Failure</th>
<th data-col-size="xs">Min. Disks Required</th>
<th data-col-size="md">Space Overhead</th>
<th data-col-size="xl">Recommended For</th>
</tr>
</thead>
<tbody>
<tr>
<td data-col-size="lg"><strong>EXTERNAL</strong></td>
<td data-col-size="xs">1</td>
<td data-col-size="xs">0</td>
<td data-col-size="xs">1</td>
<td data-col-size="md">None</td>
<td data-col-size="xl">DR, Test, Single Node</td>
</tr>
<tr>
<td data-col-size="lg"><strong>NORMAL</strong></td>
<td data-col-size="xs">2</td>
<td data-col-size="xs">1</td>
<td data-col-size="xs">2</td>
<td data-col-size="md">~100%</td>
<td data-col-size="xl">Most Production RAC</td>
</tr>
<tr>
<td data-col-size="lg"><strong>HIGH</strong></td>
<td data-col-size="xs"><strong>3</strong></td>
<td data-col-size="xs"><strong>2</strong></td>
<td data-col-size="xs"><strong>3</strong></td>
<td data-col-size="md">~200%</td>
<td data-col-size="xl">Mission-Critical Systems</td>
</tr>
</tbody>
</table>
</div>
</div>
<div>&nbsp;</div>
<div>&nbsp;</div>
</div>
<hr />
<h3 dir="auto"><strong>2. HIGH Redundancy (Detailed)</strong></h3>
<ul dir="auto">
<li><strong>HIGH</strong> redundancy maintains <strong>3 mirrored copies</strong> of every extent.</li>
<li>It can survive the failure of <strong>up to 2 disks</strong> (or 2 failure groups).</li>
<li>Requires at least <strong>3 failure groups</strong> (ideally 3+ disks).</li>
</ul>
<p dir="auto"><strong>Use Cases:</strong></p>
<ul dir="auto">
<li>Very critical production databases (Banking, Trading, Healthcare).</li>
<li>When maximum data protection is required.</li>
</ul>
<p dir="auto"><strong>Disadvantages:</strong></p>
<ul dir="auto">
<li>High storage overhead (almost 3x space).</li>
<li>Slightly slower write performance due to triple mirroring.</li>
</ul>
<hr />
<h3 dir="auto"><strong>3. Failure Groups in ASM</strong></h3>
<p dir="auto"><strong>Failure Group (FG)</strong> = A set of disks that can fail together (e.g., disks in same storage array, same shelf, or same controller).</p>
<p dir="auto"><strong>Key Points:</strong></p>
<ul dir="auto">
<li>ASM mirrors extents across <strong>different Failure Groups</strong>.</li>
<li>In <strong>HIGH</strong> redundancy, ASM stores 3 copies in <strong>3 different Failure Groups</strong>.</li>
<li>If you don&rsquo;t define Failure Groups, ASM puts each disk in its own Failure Group.</li>
</ul>
<p dir="auto"><strong>Best Practice:</strong></p>
<ul dir="auto">
<li>Always define <strong>Failure Groups</strong> manually in production for better protection.</li>
<li>Example: One Failure Group per storage controller or per rack.</li>
</ul>
<hr />
<h3 dir="auto"><strong>4. ASM Best Practices: Production vs DR</strong></h3>
<div>
<div>
<div dir="auto">
<table dir="auto">
<thead>
<tr>
<th data-col-size="md">Parameter</th>
<th data-col-size="lg">Production Environment</th>
<th data-col-size="lg">DR Environment</th>
</tr>
</thead>
<tbody>
<tr>
<td data-col-size="md">Redundancy</td>
<td data-col-size="lg">NORMAL (Preferred) or HIGH</td>
<td data-col-size="lg">EXTERNAL (Recommended)</td>
</tr>
<tr>
<td data-col-size="md">Failure Groups</td>
<td data-col-size="lg">Minimum 2 (NORMAL), 3 (HIGH)</td>
<td data-col-size="lg">Not mandatory</td>
</tr>
<tr>
<td data-col-size="md">Disk Group Size</td>
<td data-col-size="lg">Large (+DATA &amp; +FRA)</td>
<td data-col-size="lg">Moderate (depending on DB size)</td>
</tr>
<tr>
<td data-col-size="md">Performance</td>
<td data-col-size="lg">Balanced</td>
<td data-col-size="lg">Prioritize I/O performance</td>
</tr>
<tr>
<td data-col-size="md">Protection Level</td>
<td data-col-size="lg">High</td>
<td data-col-size="lg">Moderate (rely on storage RAID)</td>
</tr>
<tr>
<td data-col-size="md">ASM Disks</td>
<td data-col-size="lg">Many small disks</td>
<td data-col-size="lg">Fewer large disks</td>
</tr>
<tr>
<td data-col-size="md">Redundancy Level</td>
<td data-col-size="lg">Higher</td>
<td data-col-size="lg">Lower</td>
</tr>
</tbody>
</table>
</div>
</div>
<div>&nbsp;</div>
<div>&nbsp;</div>
</div>
<p dir="auto"><strong>General Best Practices:</strong></p>
<ul dir="auto">
<li>Use <strong>External</strong> redundancy in DR to save space and get better performance.</li>
<li>Always separate <strong>+DATA</strong> and <strong>+FRA</strong> disk groups.</li>
<li>Keep <strong>FRA</strong> size at least 2x of redo generation.</li>
<li>Use <strong>ASMLib</strong> or <strong>ASM Filter Driver (AFD)</strong> in 19c.</li>
</ul>
<hr />
<h3 dir="auto"><strong>5. Common Interview Questions &amp; Answers</strong></h3>
<p dir="auto"><strong>Q1. What is HIGH redundancy in ASM?</strong> <strong>Ans:</strong> HIGH redundancy maintains <strong>three mirrored copies</strong> of data across different failure groups. It can tolerate failure of up to <strong>two disks/failure groups</strong>.</p>
<p dir="auto"><strong>Q2. When do you choose HIGH redundancy?</strong> <strong>Ans:</strong> We choose HIGH redundancy for very critical production databases where data loss is not acceptable even if two disks fail simultaneously.</p>
<p dir="auto"><strong>Q3. What is a Failure Group?</strong> <strong>Ans:</strong> A Failure Group is a collection of disks that share a common point of failure (same storage shelf, controller, or array). ASM ensures mirrored copies are stored in different Failure Groups.</p>
<p dir="auto"><strong>Q4. Why do you use EXTERNAL redundancy in DR environment?</strong> <strong>Ans:</strong> In DR, we use EXTERNAL redundancy because:</p>
<ul dir="auto">
<li>It gives better performance.</li>
<li>Saves storage space (no mirroring overhead).</li>
<li>We rely on underlying storage RAID or SAN protection.</li>
<li>DR is usually for short-term switchover.</li>
</ul>
<p dir="auto"><strong>Q5. Can we change redundancy level after creating Disk Group?</strong> <strong>Ans:</strong> No. Redundancy level cannot be changed after Disk Group creation. We have to create a new Disk Group with desired redundancy and move data.</p>
<p dir="auto"><strong>Q6. What is the difference between NORMAL and HIGH redundancy?</strong> <strong>Ans:</strong> NORMAL provides 2-way mirroring (1 disk failure tolerance), while HIGH provides 3-way mirroring (2 disk failure tolerance).</p>
<p dir="auto"><strong>Comprehensive Explanation of RMAN (Recovery Manager)</strong></p>
<h3 dir="auto"><strong>What is RMAN?</strong></h3>
<p dir="auto"><strong>RMAN (Recovery Manager)</strong> is Oracle&rsquo;s <strong>built-in utility</strong> for <strong>backup, restore, and recovery</strong> of Oracle databases. It is the <strong>recommended and most powerful</strong> tool for Oracle database backup and recovery.</p>
<p dir="auto">It was introduced in Oracle 8 and has become the standard in all modern Oracle environments (11g, 12c, 19c, 21c).</p>
<hr />
<h3 dir="auto"><strong>Why RMAN? (Advantages)</strong></h3>
<div>
<div>
<div dir="auto">
<table dir="auto">
<thead>
<tr>
<th data-col-size="md">Feature</th>
<th data-col-size="lg">RMAN Advantage</th>
</tr>
</thead>
<tbody>
<tr>
<td data-col-size="md">Block-level backup</td>
<td data-col-size="lg">Backs up only used blocks</td>
</tr>
<tr>
<td data-col-size="md">Incremental Backup</td>
<td data-col-size="lg">Level 0 + Level 1 (very efficient)</td>
</tr>
<tr>
<td data-col-size="md">Block Media Recovery</td>
<td data-col-size="lg">Can recover single corrupted block</td>
</tr>
<tr>
<td data-col-size="md">Compression &amp; Encryption</td>
<td data-col-size="lg">Built-in support</td>
</tr>
<tr>
<td data-col-size="md">Parallelism</td>
<td data-col-size="lg">Multi-channel backup</td>
</tr>
<tr>
<td data-col-size="md">Catalog Support</td>
<td data-col-size="lg">Recovery Catalog for long-term retention</td>
</tr>
<tr>
<td data-col-size="md">Integration with Data Guard</td>
<td data-col-size="lg">Excellent</td>
</tr>
<tr>
<td data-col-size="md">Automatic Tape &amp; Disk Management</td>
<td data-col-size="lg">Yes</td>
</tr>
</tbody>
</table>
</div>
</div>
<div>&nbsp;</div>
<div>&nbsp;</div>
</div>
<hr />
<h3 dir="auto"><strong>Key Concepts in RMAN</strong></h3>
<div>
<div>
<div dir="auto">
<table dir="auto">
<thead>
<tr>
<th data-col-size="md">Term</th>
<th data-col-size="lg">Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td data-col-size="md"><strong>Backup Set</strong></td>
<td data-col-size="lg">Logical backup (default)</td>
</tr>
<tr>
<td data-col-size="md"><strong>Image Copy</strong></td>
<td data-col-size="lg">Physical copy of datafile</td>
</tr>
<tr>
<td data-col-size="md"><strong>Level 0 Backup</strong></td>
<td data-col-size="lg">Full backup (Base for incremental)</td>
</tr>
<tr>
<td data-col-size="md"><strong>Level 1 Backup</strong></td>
<td data-col-size="lg">Incremental (only changed blocks)</td>
</tr>
<tr>
<td data-col-size="md"><strong>Recovery Catalog</strong></td>
<td data-col-size="lg">Central repository of backup metadata</td>
</tr>
<tr>
<td data-col-size="md"><strong>Control File</strong></td>
<td data-col-size="lg">Stores backup information (default)</td>
</tr>
<tr>
<td data-col-size="md"><strong>Flash Recovery Area (FRA)</strong></td>
<td data-col-size="lg">Default location for backups</td>
</tr>
</tbody>
</table>
</div>
</div>
<div>&nbsp;</div>
<div>&nbsp;</div>
</div>
<hr />
<h3 dir="auto"><strong>Common RMAN Commands (Most Asked in Interviews)</strong></h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code># Connect to RMAN
rman target /

# Full Database Backup
BACKUP DATABASE PLUS ARCHIVELOG;

# Incremental Backup
BACKUP INCREMENTAL LEVEL 1 DATABASE;

# Backup Archive Logs
BACKUP ARCHIVELOG ALL;

# Restore &amp; Recover
RESTORE DATABASE;
RECOVER DATABASE;

# Block Media Recovery
BLOCKRECOVER DATAFILE 5 BLOCK 20;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>RMAN Interview Questions &amp; Answers</strong></h3>
<p dir="auto"><strong>Q1. What is the difference between User-Managed Backup and RMAN Backup?</strong> <strong>Ans:</strong> User-managed uses OS commands (cp, tar). RMAN is intelligent &mdash; it knows datafile headers, can do incremental, block-level recovery, and validates backups.</p>
<p dir="auto"><strong>Q2. What is the difference between Level 0 and Level 1 Incremental Backup?</strong> <strong>Ans:</strong></p>
<ul dir="auto">
<li>Level 0 = Full Backup (Base)</li>
<li>Level 1 = Incremental (only changed blocks since Level 0)</li>
</ul>
<p dir="auto"><strong>Q3. What is Recovery Catalog? Why do we use it?</strong> <strong>Ans:</strong> Recovery Catalog is a separate database that stores RMAN metadata for longer periods. It is more reliable than control file.</p>
<p dir="auto"><strong>Q4. What is the difference between DELETE and CROSSCHECK in RMAN?</strong> <strong>Ans:</strong></p>
<ul dir="auto">
<li>CROSSCHECK &rarr; Checks whether backup pieces exist on disk/tape.</li>
<li>DELETE &rarr; Removes backup metadata and physical files.</li>
</ul>
<p dir="auto"><strong>Q5. How do you take consistent backup in RMAN?</strong> <strong>Ans:</strong> Put database in BEGIN BACKUP / END BACKUP mode or take backup in MOUNT state.</p>
<hr />
<h3 dir="auto"><strong>RMAN Best Practices (Production vs DR)</strong></h3>
<ul dir="auto">
<li>Take <strong>Level 0</strong> weekly + <strong>Level 1</strong> daily.</li>
<li>Always backup <strong>Archive Logs</strong>.</li>
<li>Configure <strong>Retention Policy</strong> (e.g., 30 days).</li>
<li>Use <strong>FRA</strong> (Flash Recovery Area).</li>
<li>Regularly validate backups (VALIDATE DATABASE).</li>
<li>In <strong>DR</strong>, you can take lighter backups (more frequent Level 1).</li>
</ul>
<p dir="auto"><strong>Here are the most useful and commonly used RMAN Script Examples</strong> (ready for your resume &amp; interview preparation)</p>
<hr />
<h3 dir="auto"><strong>1. Basic RMAN Configuration Script</strong></h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code># Connect to RMAN
rman target /

CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 30 DAYS;
CONFIGURE BACKUP OPTIMIZATION ON;
CONFIGURE DEFAULT DEVICE TYPE TO DISK;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '/oracle/backup/%F';
CONFIGURE CHANNEL DEVICE TYPE DISK FORMAT '/oracle/backup/%U';
SHOW ALL;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>2. Full Database Backup Script (Most Common)</strong></h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code># Full Database + Archive Log Backup
RUN {
    ALLOCATE CHANNEL c1 DEVICE TYPE DISK;
    ALLOCATE CHANNEL c2 DEVICE TYPE DISK;
    BACKUP AS COMPRESSED BACKUPSET 
        DATABASE PLUS ARCHIVELOG 
        DELETE INPUT;
    RELEASE CHANNEL c1;
    RELEASE CHANNEL c2;
}</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>3. Incremental Backup Strategy (Level 0 + Level 1)</strong></h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code># Level 0 (Weekly Full)
BACKUP INCREMENTAL LEVEL 0 DATABASE PLUS ARCHIVELOG;

# Level 1 (Daily Incremental)
BACKUP INCREMENTAL LEVEL 1 CUMULATIVE DATABASE PLUS ARCHIVELOG;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>4. Archive Log Backup Only</strong></h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>RUN {
    BACKUP ARCHIVELOG ALL NOT BACKED UP DELETE INPUT;
}</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>5. Restore &amp; Recover Database Script</strong></h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>RUN {
    SET UNTIL TIME "TO_DATE('2026-06-26 10:00:00', 'YYYY-MM-DD HH24:MI:SS')";
    RESTORE DATABASE;
    RECOVER DATABASE;
}</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>6. Block Media Recovery (Very Important Interview Question)</strong></h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code># Recover single corrupted block
BLOCKRECOVER DATAFILE 5 BLOCK 120;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>7. Validate Backup (Without Actual Restore)</strong></h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code># Check if backup is valid
VALIDATE DATABASE;

# Validate specific backupset
VALIDATE BACKUPSET 1234;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>8. Complete Production Backup Script (Recommended)</strong></h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>#!/bin/bash
# prod_backup.sh
rman target / &lt;&lt; EOF
RUN {
    CROSSCHECK BACKUP;
    CROSSCHECK ARCHIVELOG ALL;
    
    BACKUP AS COMPRESSED BACKUPSET 
        INCREMENTAL LEVEL 1 CUMULATIVE 
        DATABASE PLUS ARCHIVELOG DELETE INPUT;
        
    DELETE NOPROMPT OBSOLETE;
    DELETE NOPROMPT ARCHIVELOG ALL COMPLETED BEFORE SYSDATE-7;
}
EXIT;
EOF</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>9. RMAN Report Commands</strong></h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>REPORT NEED BACKUP;
REPORT UNRECOVERABLE;
LIST BACKUP SUMMARY;
LIST BACKUP OF DATABASE;</code></pre>
</div>
</div>
</div>
</div>
<p dir="auto"><strong>RMAN Configuration Best Practices for Oracle 12c &amp; 19c</strong></p>
<p dir="auto">Here&rsquo;s a clean, professional, and interview-ready summary:</p>
<h3 dir="auto"><strong>RMAN Best Practices (12c &amp; 19c)</strong></h3>
<div>
<div>
<div dir="auto">
<table dir="auto">
<thead>
<tr>
<th data-col-size="md">Configuration</th>
<th data-col-size="md">Recommended Setting</th>
<th data-col-size="lg">Reason</th>
</tr>
</thead>
<tbody>
<tr>
<td data-col-size="md"><strong>Retention Policy</strong></td>
<td data-col-size="md">RECOVERY WINDOW OF 30 DAYS</td>
<td data-col-size="lg">Balances recoverability and space</td>
</tr>
<tr>
<td data-col-size="md"><strong>Backup Optimization</strong></td>
<td data-col-size="md">ON</td>
<td data-col-size="lg">Skips unchanged files</td>
</tr>
<tr>
<td data-col-size="md"><strong>Controlfile Autobackup</strong></td>
<td data-col-size="md">ON</td>
<td data-col-size="lg">Critical for recovery</td>
</tr>
<tr>
<td data-col-size="md"><strong>Compression</strong></td>
<td data-col-size="md">HIGH or BASIC</td>
<td data-col-size="lg">Saves space (HIGH is better in 19c)</td>
</tr>
<tr>
<td data-col-size="md"><strong>Parallelism</strong></td>
<td data-col-size="md">4 to 16 channels</td>
<td data-col-size="lg">Depends on CPU &amp; Storage</td>
</tr>
<tr>
<td data-col-size="md"><strong>Encryption</strong></td>
<td data-col-size="md">TDE + RMAN Encryption (if required)</td>
<td data-col-size="lg">Security</td>
</tr>
<tr>
<td data-col-size="md"><strong>FRA Usage</strong></td>
<td data-col-size="md">Mandatory</td>
<td data-col-size="lg">For Archive logs + Backups</td>
</tr>
<tr>
<td data-col-size="md"><strong>Catalog</strong></td>
<td data-col-size="md">Use Recovery Catalog in Production</td>
<td data-col-size="lg">Better long-term management</td>
</tr>
</tbody>
</table>
</div>
</div>
<div>&nbsp;</div>
<div>&nbsp;</div>
</div>
<hr />
<h3 dir="auto"><strong>Recommended RMAN Configuration Script (19c)</strong></h3>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>SQL
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 30 DAYS;
CONFIGURE BACKUP OPTIMIZATION ON;
CONFIGURE DEFAULT DEVICE TYPE TO DISK;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '/oracle/backup/%F';
CONFIGURE MAXSETSIZE TO UNLIMITED;
CONFIGURE COMPRESSION ALGORITHM 'HIGH' AS OF RELEASE 'DEFAULT';
CONFIGURE DEVICE TYPE DISK PARALLELISM 8 BACKUP TYPE TO COMPRESSED BACKUPSET;
CONFIGURE SNAPSHOT CONTROLFILE NAME TO '/oracle/backup/snapcf_%F';
SHOW ALL;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>Production Backup Strategy (Best Practice)</strong></h3>
<ul dir="auto">
<li><strong>Weekly</strong> &rarr; Level 0 (Full Backup)</li>
<li><strong>Daily</strong> &rarr; Level 1 Cumulative Incremental</li>
<li><strong>Archive Logs</strong> &rarr; Backup every 4&ndash;6 hours or when 70% full</li>
<li><strong>Delete Obsolete</strong> &rarr; Run daily</li>
</ul>
<hr />
<h3 dir="auto"><strong>RMAN Best Practices Summary</strong></h3>
<p dir="auto"><strong>Do&rsquo;s:</strong></p>
<ul dir="auto">
<li>Always enable CONTROLFILE AUTOBACKUP</li>
<li>Use <strong>Compressed Backupset</strong></li>
<li>Separate +DATA and +FRA disk groups</li>
<li>Regularly run DELETE OBSOLETE and VALIDATE DATABASE</li>
<li>Use <strong>Recovery Catalog</strong> in Production</li>
<li>Monitor FRA space usage daily</li>
</ul>
<p dir="auto"><strong>Don&rsquo;ts:</strong></p>
<ul dir="auto">
<li>Don&rsquo;t keep very long retention without enough space</li>
<li>Don&rsquo;t run high parallelism during peak hours</li>
<li>Don&rsquo;t store backups on the same disk as database files</li>
</ul>
<hr />
<h3 dir="auto"><strong>Common Interview Questions</strong></h3>
<p dir="auto"><strong>Q1. What are the first 5 RMAN configurations you do after database creation?</strong> <strong>Ans:</strong> Retention Policy, Controlfile Autobackup, Backup Optimization, Compression, and Parallelism.</p>
<p dir="auto"><strong>Q2. Why do you enable Controlfile Autobackup?</strong> <strong>Ans:</strong> It automatically backs up the control file and SPFILE after every backup or structural change. Very important during disaster recovery when control file is lost.</p>
<p dir="auto"><strong>Q3. What is the difference between DELETE OBSOLETE and CROSSCHECK?</strong> <strong>Ans:</strong></p>
<ul dir="auto">
<li>CROSSCHECK &rarr; Checks physical existence of backup pieces.</li>
<li>DELETE OBSOLETE &rarr; Deletes backups according to retention policy.</li>
</ul>
<p dir="auto"><strong>Q4. Which compression algorithm do you prefer in 19c?</strong> <strong>Ans:</strong> HIGH &mdash; better compression ratio with acceptable CPU overhead.</p>
<hr />
<p dir="auto">Would you like me to also prepare:</p>
<ul dir="auto">
<li>RMAN Incremental Strategy (Level 0 + Level 1)</li>
<li>RMAN with Recovery Catalog Best Practices</li>
<li>Active Database Duplication Script (for DR)</li>
</ul>
<p dir="auto">Let me know!</p>
<div>&nbsp;</div>
<div>
<div>
<p dir="auto"><strong>✅ Oracle Data Guard Configuration &ndash; Complete Guide</strong></p>
<h3 dir="auto"><strong>What is Oracle Data Guard?</strong></h3>
<p dir="auto"><strong>Oracle Data Guard</strong> is Oracle&rsquo;s <strong>disaster recovery (DR)</strong> solution. It provides <strong>high availability, data protection, and disaster recovery</strong> by maintaining one or more <strong>standby databases</strong> that are transactionally consistent with the <strong>primary database</strong>.</p>
<hr />
<h3 dir="auto"><strong>Types of Standby Databases</strong></h3>
<div>
<div>
<div dir="auto">
<table dir="auto">
<thead>
<tr>
<th data-col-size="md">Type</th>
<th data-col-size="lg">Open Mode</th>
<th data-col-size="md">Redo Apply Method</th>
<th data-col-size="xl">Use Case</th>
</tr>
</thead>
<tbody>
<tr>
<td data-col-size="md"><strong>Physical Standby</strong></td>
<td data-col-size="lg">Mounted / Read Only</td>
<td data-col-size="md">Redo Apply (MRP)</td>
<td data-col-size="xl"><strong>Most Common</strong> (Exact copy)</td>
</tr>
<tr>
<td data-col-size="md"><strong>Logical Standby</strong></td>
<td data-col-size="lg">Read Only / Read Write</td>
<td data-col-size="md">SQL Apply</td>
<td data-col-size="xl">Reporting, Rolling Upgrades</td>
</tr>
<tr>
<td data-col-size="md"><strong>Snapshot Standby</strong></td>
<td data-col-size="lg">Read Write</td>
<td data-col-size="md">Flashback + Redo</td>
<td data-col-size="xl">Testing &amp; Development</td>
</tr>
</tbody>
</table>
</div>
</div>
<div>&nbsp;</div>
<div>&nbsp;</div>
</div>
<hr />
<h3 dir="auto"><strong>Data Guard Architecture Components</strong></h3>
<ul dir="auto">
<li><strong>Primary Database</strong> &mdash; Production database</li>
<li><strong>Standby Database</strong> &mdash; DR copy</li>
<li><strong>Redo Transport</strong> &mdash; Ships redo from Primary to Standby</li>
<li><strong>Redo Apply</strong> &mdash; Applies redo on Standby (MRP process)</li>
<li><strong>Data Guard Broker</strong> &mdash; Management tool (dgmgrl)</li>
</ul>
<hr />
<h3 dir="auto"><strong>Data Guard Configuration Modes</strong></h3>
<div>
<div>
<div dir="auto">
<table dir="auto">
<thead>
<tr>
<th data-col-size="lg">Protection Mode</th>
<th data-col-size="md">Data Loss Possible?</th>
<th data-col-size="sm">Performance Impact</th>
<th data-col-size="md">Recommended For</th>
</tr>
</thead>
<tbody>
<tr>
<td data-col-size="lg"><strong>Maximum Protection</strong></td>
<td data-col-size="md">Zero Data Loss</td>
<td data-col-size="sm">High</td>
<td data-col-size="md">Very Critical</td>
</tr>
<tr>
<td data-col-size="lg"><strong>Maximum Availability</strong></td>
<td data-col-size="md">Minimal</td>
<td data-col-size="sm">Medium</td>
<td data-col-size="md"><strong>Most Production</strong></td>
</tr>
<tr>
<td data-col-size="lg"><strong>Maximum Performance</strong></td>
<td data-col-size="md">Possible</td>
<td data-col-size="sm">Low</td>
<td data-col-size="md"><strong>DR Environments</strong> (Recommended)</td>
</tr>
</tbody>
</table>
</div>
</div>
<div>&nbsp;</div>
<div>&nbsp;</div>
</div>
<hr />
<h3 dir="auto"><strong>Step-by-Step Data Guard Configuration (19c)</strong></h3>
<h4 dir="auto"><strong>Phase 1: Primary Database Preparation</strong></h4>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>SQL
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>-- Enable Force Logging
ALTER DATABASE FORCE LOGGING;

-- Check Redo Log size (should be minimum 1 GB)
SELECT group#, bytes/1024/1024 "Size MB" FROM v$log;

-- Create Standby Redo Logs (SRL)
ALTER DATABASE ADD STANDBY LOGFILE THREAD 1 GROUP 11 ('+DATA') SIZE 1G;
-- Repeat for more groups</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<h4 dir="auto"><strong>Phase 2: Init Parameters on Primary</strong></h4>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>SQL
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>ALTER SYSTEM SET log_archive_config='DG_CONFIG=(PROD,PROD_DR)' SCOPE=SPFILE;
ALTER SYSTEM SET log_archive_dest_2='SERVICE=PROD_DR ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=PROD_DR' SCOPE=SPFILE;
ALTER SYSTEM SET fal_server='PROD_DR' SCOPE=SPFILE;
ALTER SYSTEM SET db_file_name_convert='/PROD_DR/','/PROD/' SCOPE=SPFILE;
ALTER SYSTEM SET log_file_name_convert='/PROD_DR/','/PROD/' SCOPE=SPFILE;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<h4 dir="auto"><strong>Phase 3: Standby Database Creation</strong></h4>
<p dir="auto">Common methods:</p>
<ol dir="auto">
<li><strong>RMAN Active Duplication</strong> (Recommended)</li>
<li><strong>RMAN Backup + Restore</strong></li>
<li><strong>Snapshot Standby</strong></li>
</ol>
<hr />
<h3 dir="auto"><strong>Most Important Interview Questions</strong></h3>
<p dir="auto"><strong>Q1. What is the difference between Physical and Logical Standby?</strong> <strong>Ans:</strong> Physical Standby is block-for-block identical (Redo Apply), while Logical Standby is logically identical (SQL Apply) and can be opened in Read-Write mode.</p>
<p dir="auto"><strong>Q2. What are the Data Guard Protection Modes?</strong> <strong>Ans:</strong> Maximum Protection, Maximum Availability, and Maximum Performance.</p>
<p dir="auto"><strong>Q3. What is the role of MRP process?</strong> <strong>Ans:</strong> Managed Recovery Process (MRP) applies redo logs on the standby database.</p>
<p dir="auto"><strong>Q4. What is Data Guard Broker?</strong> <strong>Ans:</strong> A centralized management tool that simplifies configuration, monitoring, and switchover using dgmgrl.</p>
<p dir="auto"><strong>Q5. How do you check Data Guard status?</strong></p>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>SQL
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>-- On Standby
SELECT name, value FROM v$dataguard_stats;
SELECT process, status, thread#, sequence# FROM v$managed_standby;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<p dir="auto">Would you like me to also provide:</p>
<ul dir="auto">
<li><strong>Full RMAN Active Duplication Script</strong> for Physical Standby</li>
<li><strong>Data Guard Broker Configuration Steps</strong></li>
<li><strong>Switchover &amp; Failover Commands</strong></li>
<li><strong>Best Practices for Data Guard in 19c</strong></li>
</ul>
<p dir="auto">Just tell me which section you need next.</p>
</div>
</div>
<section></section>
<p dir="auto"><strong>Here is the Complete Guide for all four requested topics:</strong></p>
<hr />
<h3 dir="auto"><strong>1. Full RMAN Active Duplication Script for Physical Standby (19c)</strong></h3>
<p dir="auto"><strong>Best Method for Creating Physical Standby</strong></p>
<h4 dir="auto"><strong>On Primary Database (Run as sysdba)</strong></h4>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>SQL
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>-- Enable Force Logging
ALTER DATABASE FORCE LOGGING;

-- Create Standby Redo Logs
ALTER DATABASE ADD STANDBY LOGFILE THREAD 1 GROUP 11 SIZE 1G;
ALTER DATABASE ADD STANDBY LOGFILE THREAD 1 GROUP 12 SIZE 1G;
ALTER DATABASE ADD STANDBY LOGFILE THREAD 1 GROUP 13 SIZE 1G;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<h4 dir="auto"><strong>On Standby Server &ndash; RMAN Active Duplication Script</strong></h4>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code># Connect to RMAN on Standby
rman target sys/password@PRODCDB_DR auxiliary sys/password@PRODCDB

# Run this script
DUPLICATE TARGET DATABASE
  FOR STANDBY
  FROM ACTIVE DATABASE
  SPFILE
  PARAMETER_VALUE_CONVERT 'PRODCDB','PRODCDB_DR'
  SET DB_UNIQUE_NAME='PRODCDB_DR'
  SET DB_FILE_NAME_CONVERT '/PRODCDB/','/PRODCDB_DR/'
  SET LOG_FILE_NAME_CONVERT '/PRODCDB/','/PRODCDB_DR/'
  SET STANDBY_FILE_MANAGEMENT='AUTO'
  NOFILENAMECHECK;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>2. Data Guard Broker Configuration Steps</strong></h3>
<h4 dir="auto"><strong>Step 1: On Both Primary and Standby</strong></h4>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>SQL
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>ALTER SYSTEM SET dg_broker_config_file1='+DATA/dr1PRODCDB_DR.dat' SCOPE=SPFILE;
ALTER SYSTEM SET dg_broker_config_file2='+DATA/dr2PRODCDB_DR.dat' SCOPE=SPFILE;
ALTER SYSTEM SET dg_broker_start=TRUE SCOPE=SPFILE;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<h4 dir="auto"><strong>Step 2: Start DGMGRL and Create Configuration</strong></h4>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code>dgmgrl sys/password@PRODCDB

CREATE CONFIGURATION 'ERP_DG' AS PRIMARY DATABASE IS 'PRODCDB' CONNECT IDENTIFIER IS 'PRODCDB';
ADD DATABASE 'PRODCDB_DR' AS CONNECT IDENTIFIER IS 'PRODCDB_DR' MAINTAINED AS PHYSICAL;
ENABLE CONFIGURATION;
SHOW CONFIGURATION;</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>3. Switchover &amp; Failover Commands</strong></h3>
<h4 dir="auto"><strong>Switchover (Planned Role Transition)</strong></h4>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code># On Primary
dgmgrl
SWITCHOVER TO 'PRODCDB_DR';</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<h4 dir="auto"><strong>Failover (Emergency &ndash; When Primary is down)</strong></h4>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code># On Standby
dgmgrl
FAILOVER TO 'PRODCDB_DR';</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<h4 dir="auto"><strong>After Failover (Reinstate Old Primary as New Standby)</strong></h4>
<div dir="auto">
<div data-testid="code-block">
<div>
<div>Bash
<div>&nbsp;</div>
</div>
<div>
<pre tabindex="0"><code># On old Primary (after fixing it)
dgmgrl
REINSTATE DATABASE 'PRODCDB';</code></pre>
</div>
<div>&nbsp;</div>
</div>
<div>&nbsp;</div>
</div>
</div>
<hr />
<h3 dir="auto"><strong>4. Data Guard Best Practices in 19c</strong></h3>
<p dir="auto"><strong>General Best Practices:</strong></p>
<ul dir="auto">
<li>Use <strong>Maximum Availability</strong> mode in Production.</li>
<li>Use <strong>Maximum Performance</strong> mode in DR environments.</li>
<li>Always create sufficient <strong>Standby Redo Logs</strong> (SRL).</li>
<li>Enable <strong>Real-Time Apply</strong> (USING CURRENT LOGFILE).</li>
<li>Use <strong>Data Guard Broker</strong> for management.</li>
<li>Monitor using v$dataguard_stats and v$managed_standby.</li>
<li>Regularly test <strong>Switchover</strong> and <strong>Failover</strong>.</li>
<li>Keep <strong>Flash Recovery Area</strong> well-sized on both sides.</li>
<li>Use <strong>RMAN Active Duplication</strong> for new standbys.</li>
</ul>
<p dir="auto"><strong>DR Specific Recommendations:</strong></p>
<ul dir="auto">
<li>Use <strong>EXTERNAL</strong> ASM redundancy on DR.</li>
<li>Schedule regular <strong>DR Drill</strong> every 6 months.</li>
<li>Use <strong>Active Data Guard</strong> (if licensed) for reporting on Standby.</li>
<li>Monitor <strong>Transport Lag</strong> and <strong>Apply Lag</strong> closely.</li>
</ul>
<hr />
<p dir="auto">Would you like me to expand any of these sections further (with more scripts or examples)?</p>
<p dir="auto">You now have a complete set for:</p>
<ul dir="auto">
<li>RMAN</li>
<li>Data Guard</li>
<li>Oracle Restart</li>
<li>ASM Redundancy</li>
</ul>
<hr />
<p dir="auto">&nbsp;</p>
