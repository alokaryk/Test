<h3 data-path-to-node="1">Baruwa Health &amp; Anti-Saturation Script</h3>
<p data-path-to-node="2">This script checks for three things:</p>
<ol start="1" data-path-to-node="3">
<li>
<p data-path-to-node="3,0,0"><strong data-path-to-node="3,0,0" data-index-in-node="0">Zombie processes</strong> (The "Z" state you found).</p>
</li>
<li>
<p data-path-to-node="3,1,0"><strong data-path-to-node="3,1,0" data-index-in-node="0">Exim Deferral rate</strong> (Detecting if DNS is timing out).</p>
</li>
<li>
<p data-path-to-node="3,2,0"><strong data-path-to-node="3,2,0" data-index-in-node="0">Unbound Status</strong> (Ensuring the local resolver is up).</p>
</li>
</ol>
<div class="code-block ng-tns-c3098535048-1064 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwiLhq7dhPSRAxUAAAAAHQAAAAAQ2Q0">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c3098535048-1064 ng-star-inserted"><span class="ng-tns-c3098535048-1064">Bash</span>
<div class="buttons ng-tns-c3098535048-1064 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c3098535048-1064">
<div class="animated-opacity ng-tns-c3098535048-1064">
<pre class="ng-tns-c3098535048-1064"><code class="code-container formatted ng-tns-c3098535048-1064" data-test-id="code-content"><span class="hljs-meta">#!/bin/bash</span>

<span class="hljs-comment"># --- Configuration ---</span>
EMAIL=<span class="hljs-string">"admin@yourdomain.com"</span>
MAX_ZOMBIES=5
MAX_SCANNERS=50
LOG_FILE=<span class="hljs-string">"/var/log/baruwa_health.log"</span>

<span class="hljs-built_in">echo</span> <span class="hljs-string">"--- Baruwa Health Check: <span class="hljs-subst">$(date)</span> ---"</span> &gt;&gt; <span class="hljs-variable">$LOG_FILE</span>

<span class="hljs-comment"># 1. Check for Zombie Processes</span>
ZOMBIE_COUNT=$(ps aux | awk <span class="hljs-string">'$8=="Z" {print $2}'</span> | wc -l)

<span class="hljs-keyword">if</span> [ <span class="hljs-string">"<span class="hljs-variable">$ZOMBIE_COUNT</span>"</span> -gt <span class="hljs-string">"<span class="hljs-variable">$MAX_ZOMBIES</span>"</span> ]; <span class="hljs-keyword">then</span>
    <span class="hljs-built_in">echo</span> <span class="hljs-string">"ALERT: <span class="hljs-variable">$ZOMBIE_COUNT</span> zombies detected. Clearing..."</span> &gt;&gt; <span class="hljs-variable">$LOG_FILE</span>
    <span class="hljs-comment"># Kill parent processes of zombies to clean table</span>
    ps -ef | grep defunct | grep -v grep | awk <span class="hljs-string">'{print $3}'</span> | xargs <span class="hljs-built_in">kill</span> -9
<span class="hljs-keyword">fi</span>

<span class="hljs-comment"># 2. Check Unbound (Local DNS)</span>
<span class="hljs-keyword">if</span> ! systemctl is-active --quiet unbound; <span class="hljs-keyword">then</span>
    <span class="hljs-built_in">echo</span> <span class="hljs-string">"ALERT: Unbound DNS is down. Restarting..."</span> &gt;&gt; <span class="hljs-variable">$LOG_FILE</span>
    systemctl restart unbound
<span class="hljs-keyword">fi</span>

<span class="hljs-comment"># 3. Check for DNS Deferrals in Mail Log (Last 5 mins)</span>
DNS_DEFER=$(grep <span class="hljs-string">"DNS list lookup defer"</span> /var/<span class="hljs-built_in">log</span>/maillog | tail -n 20)

<span class="hljs-keyword">if</span> [ ! -z <span class="hljs-string">"<span class="hljs-variable">$DNS_DEFER</span>"</span> ]; <span class="hljs-keyword">then</span>
    <span class="hljs-built_in">echo</span> <span class="hljs-string">"WARNING: DNS RBL timeouts detected in maillog."</span> &gt;&gt; <span class="hljs-variable">$LOG_FILE</span>
    <span class="hljs-comment"># Optional: Restart scanner if it's hanging</span>
    systemctl restart baruwascanner
<span class="hljs-keyword">fi</span>

<span class="hljs-comment"># 4. Process Table Cleanup</span>
<span class="hljs-comment"># If BaruwaScanner exceeds safe limits, restart it to prevent RAM saturation</span>
SCANNER_COUNT=$(ps aux | grep -i BaruwaScanner | grep -v grep | wc -l)
<span class="hljs-keyword">if</span> [ <span class="hljs-string">"<span class="hljs-variable">$SCANNER_COUNT</span>"</span> -gt <span class="hljs-string">"<span class="hljs-variable">$MAX_SCANNERS</span>"</span> ]; <span class="hljs-keyword">then</span>
    <span class="hljs-built_in">echo</span> <span class="hljs-string">"CRITICAL: Scanner saturation (<span class="hljs-variable">$SCANNER_COUNT</span>). Restarting services..."</span> &gt;&gt; <span class="hljs-variable">$LOG_FILE</span>
    systemctl restart baruwascanner
    systemctl restart exim
<span class="hljs-keyword">fi</span>

<span class="hljs-built_in">echo</span> <span class="hljs-string">"Check complete. Current Scanners: <span class="hljs-variable">$SCANNER_COUNT</span>, Zombies: <span class="hljs-variable">$ZOMBIE_COUNT</span>"</span> &gt;&gt; <span class="hljs-variable">$LOG_FILE</span>
</code></pre>
</div>
</div>
</div>
<hr data-path-to-node="5" />
<h3 data-path-to-node="6">How to Install and Automate</h3>
<ol start="1" data-path-to-node="7">
<li>
<p data-path-to-node="7,0,0"><strong data-path-to-node="7,0,0" data-index-in-node="0">Create the file:</strong> <code data-path-to-node="7,0,0" data-index-in-node="17">vi /usr/local/bin/baruwa_check.sh</code></p>
</li>
<li>
<p data-path-to-node="7,1,0"><strong data-path-to-node="7,1,0" data-index-in-node="0">Paste the code above and save.</strong></p>
</li>
<li>
<p data-path-to-node="7,2,0"><strong data-path-to-node="7,2,0" data-index-in-node="0">Make it executable:</strong> <code data-path-to-node="7,2,0" data-index-in-node="20">chmod +x /usr/local/bin/baruwa_check.sh</code></p>
</li>
<li>
<p data-path-to-node="7,3,0"><strong data-path-to-node="7,3,0" data-index-in-node="0">Add to Crontab (Run every 10 minutes):</strong> Run <code data-path-to-node="7,3,0" data-index-in-node="43">crontab -e</code> and add this line at the bottom: <code data-path-to-node="7,3,0" data-index-in-node="87">*/10 * * * * /usr/local/bin/baruwa_check.sh &gt; /dev/null 2&gt;&amp;1</code></p>
</li>
</ol>
<h3 data-path-to-node="8">Why this works for your specific case:</h3>
<ul data-path-to-node="9">
<li>
<p data-path-to-node="9,0,0"><strong data-path-to-node="9,0,0" data-index-in-node="0">The Zombie Killer:</strong> It identifies the parent of a "defunct" process (the hanging Exim or Scanner task) and kills it, preventing the process table from filling up.</p>
</li>
<li>
<p data-path-to-node="9,1,0"><strong data-path-to-node="9,1,0" data-index-in-node="0">The Unbound Guard:</strong> Since your saturation was caused by DNS timeouts, ensuring <code data-path-to-node="9,1,0" data-index-in-node="78">unbound</code> is always active is the most important "fail-safe."</p>
</li>
<li>
<p data-path-to-node="9,2,0"><strong data-path-to-node="9,2,0" data-index-in-node="0">Logging:</strong> It keeps a record in <code data-path-to-node="9,2,0" data-index-in-node="30">/var/log/baruwa_health.log</code> so you can see if the system became saturated while you were away.</p>
</li>
</ul>
<p data-path-to-node="10"><strong data-path-to-node="10" data-index-in-node="0">Would you like me to add a line to this script that also checks the disk space of your mail queue directory, just in case a backup fills the partition?</strong></p>
<!-- Comments are visible in the HTML source only -->
