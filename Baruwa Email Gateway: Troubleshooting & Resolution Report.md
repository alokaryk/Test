<h3 data-path-to-node="4">1. Incident Symptom: "Process Saturation"</h3>
<ul data-path-to-node="5">
<li>
<p data-path-to-node="5,0,0"><strong data-path-to-node="5,0,0" data-index-in-node="0">Observation:</strong> The server showed a high number of <strong data-path-to-node="5,0,0" data-index-in-node="48">Zombie (Z) processes</strong> and a total task count exceeding <strong data-path-to-node="5,0,0" data-index-in-node="102">600+</strong>.</p>
</li>
<li>
<p data-path-to-node="5,1,0"><strong data-path-to-node="5,1,0" data-index-in-node="0">Impact:</strong> Mail processing slowed down or stopped entirely as the process table was exhausted.</p>
</li>
<li>
<p data-path-to-node="5,2,0"><strong data-path-to-node="5,2,0" data-index-in-node="0">Primary Culprit:</strong> Exim was "hanging" on RBL (Real-time Blacklist) DNS lookups, specifically to <code data-path-to-node="5,2,0" data-index-in-node="94">hk.rbl.baruwa.net</code>.</p>
</li>
</ul>
<h3 data-path-to-node="6">2. Root Cause Analysis</h3>
<p data-path-to-node="7">The network outage caused the local DNS resolver (<strong data-path-to-node="7" data-index-in-node="50">Unbound</strong>) to become saturated with timed-out requests.</p>
<ul data-path-to-node="8">
<li>
<p data-path-to-node="8,0,0"><strong data-path-to-node="8,0,0" data-index-in-node="0">The Timeout Loop:</strong> Exim was configured with the <code data-path-to-node="8,0,0" data-index-in-node="47">+defer_unknown</code> flag. This meant that if the DNS server didn't respond immediately (due to the network glitch), Exim would "hold" the process open and try again, rather than simply skipping the check.</p>
</li>
<li>
<p data-path-to-node="8,1,0"><strong data-path-to-node="8,1,0" data-index-in-node="0">Resource Stagnation:</strong> Because the processes were waiting for DNS replies that never came, they occupied "slots" in the process table but used 0% CPU, leading to the "low load but high task count" paradox.</p>
</li>
</ul>
<hr data-path-to-node="9" />
<h3 data-path-to-node="10">3. Resolution Steps Taken (The Fix)</h3>
<h4 data-path-to-node="11">Step A: DNS Resolver Recovery</h4>
<p data-path-to-node="12">We restarted the local caching resolver to flush the "toxic" or stalled cache:</p>
<div class="code-block ng-tns-c3098535048-1053 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwiLhq7dhPSRAxUAAAAAHQAAAAAQtQ0">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c3098535048-1053 ng-star-inserted"><span class="ng-tns-c3098535048-1053">Bash</span>
<div class="buttons ng-tns-c3098535048-1053 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c3098535048-1053">
<div class="animated-opacity ng-tns-c3098535048-1053">
<pre class="ng-tns-c3098535048-1053"><code class="code-container formatted ng-tns-c3098535048-1053" data-test-id="code-content">service unbound restart
</code></pre>
</div>
</div>
</div>
<p data-path-to-node="14"><em data-path-to-node="14" data-index-in-node="0">Verification:</em> Tested with <code data-path-to-node="14" data-index-in-node="26">host -t mx microsoft.com</code> to ensure sub-millisecond resolution was restored.</p>
<h4 data-path-to-node="15">Step B: Configuration Optimization (The "Saturation Breaker")</h4>
<p data-path-to-node="16">We edited the Exim configuration (typically <code data-path-to-node="16" data-index-in-node="44">/etc/exim/exim.conf</code>) to remove the <code data-path-to-node="16" data-index-in-node="79">+defer_unknown</code> flag from the RBL check blocks.</p>
<ul data-path-to-node="17">
<li>
<p data-path-to-node="17,0,0"><strong data-path-to-node="17,0,0" data-index-in-node="0">Before:</strong> <code data-path-to-node="17,0,0" data-index-in-node="8">dnslists = +defer_unknown : hk.rbl.baruwa.net=127.0.0.2</code></p>
</li>
<li>
<p data-path-to-node="17,1,0"><strong data-path-to-node="17,1,0" data-index-in-node="0">After:</strong> <code data-path-to-node="17,1,0" data-index-in-node="7">dnslists = hk.rbl.baruwa.net=127.0.0.2</code></p>
</li>
<li>
<p data-path-to-node="17,2,0"><strong data-path-to-node="17,2,0" data-index-in-node="0">Result:</strong> This ensures that if the DNS RBL is slow or down, Exim ignores the result and delivers the mail anyway, rather than creating a zombie process.</p>
</li>
</ul>
<h4 data-path-to-node="18">Step C: Process Table Purge</h4>
<p data-path-to-node="19">We cleared the stale/hanging processes to allow the system to start fresh:</p>
<div class="code-block ng-tns-c3098535048-1054 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwiLhq7dhPSRAxUAAAAAHQAAAAAQtg0">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c3098535048-1054 ng-star-inserted"><span class="ng-tns-c3098535048-1054">Bash</span>
<div class="buttons ng-tns-c3098535048-1054 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c3098535048-1054">
<div class="animated-opacity ng-tns-c3098535048-1054">
<pre class="ng-tns-c3098535048-1054"><code class="code-container formatted ng-tns-c3098535048-1054" data-test-id="code-content">killall exim
service baruwascanner restart
</code></pre>
</div>
</div>
</div>
<hr data-path-to-node="21" />
<h3 data-path-to-node="22">4. Current System Health (Verified)</h3>
<table data-path-to-node="23">
<thead>
<tr>
<td><strong>Metric</strong></td>
<td><strong>Status</strong></td>
<td><strong>Value</strong></td>
</tr>
</thead>
<tbody>
<tr>
<td><span data-path-to-node="23,1,0,0"><strong data-path-to-node="23,1,0,0" data-index-in-node="0">Zombies</strong></span></td>
<td><span data-path-to-node="23,1,1,0"><strong data-path-to-node="23,1,1,0" data-index-in-node="0">Healthy</strong></span></td>
<td><span data-path-to-node="23,1,2,0">0</span></td>
</tr>
<tr>
<td><span data-path-to-node="23,2,0,0"><strong data-path-to-node="23,2,0,0" data-index-in-node="0">Scanner Processes</strong></span></td>
<td><span data-path-to-node="23,2,1,0"><strong data-path-to-node="23,2,1,0" data-index-in-node="0">Normal</strong></span></td>
<td><span data-path-to-node="23,2,2,0">7</span></td>
</tr>
<tr>
<td><span data-path-to-node="23,3,0,0"><strong data-path-to-node="23,3,0,0" data-index-in-node="0">DNS Resolution</strong></span></td>
<td><span data-path-to-node="23,3,1,0"><strong data-path-to-node="23,3,1,0" data-index-in-node="0">Fast</strong></span></td>
<td><span data-path-to-node="23,3,2,0">Sub-50ms</span></td>
</tr>
<tr>
<td><span data-path-to-node="23,4,0,0"><strong data-path-to-node="23,4,0,0" data-index-in-node="0">Mail Queue</strong></span></td>
<td><span data-path-to-node="23,4,1,0"><strong data-path-to-node="23,4,1,0" data-index-in-node="0">Clear</strong></span></td>
<td><span data-path-to-node="23,4,2,0">No backlog</span></td>
</tr>
</tbody>
</table>
<hr data-path-to-node="24" />
<h3 data-path-to-node="25">5. Future Prevention (Best Practices)</h3>
<ol start="1" data-path-to-node="26">
<li>
<p data-path-to-node="26,0,0"><strong data-path-to-node="26,0,0" data-index-in-node="0">Avoid <code data-path-to-node="26,0,0" data-index-in-node="6">+defer_unknown</code>:</strong> In high-volume environments, it is better to allow a few spams through during a DNS outage than to have the entire server crash due to process saturation.</p>
</li>
<li>
<p data-path-to-node="26,1,0"><strong data-path-to-node="26,1,0" data-index-in-node="0">Monitor Unbound:</strong> Periodically check the status of the local resolver. If <code data-path-to-node="26,1,0" data-index-in-node="73">unbound</code> stops, Baruwa will immediately begin creating zombies.</p>
</li>
<li>
<p data-path-to-node="26,2,0"><strong data-path-to-node="26,2,0" data-index-in-node="0">Log Monitoring:</strong> Regularly run <code data-path-to-node="26,2,0" data-index-in-node="30">grep "DNS list lookup defer" /var/log/maillog</code>. If you see this appearing frequently, it means the RBL provider is throttling you or your DNS is lagging again.</p>
</li>
</ol>
<!-- Comments are visible in the HTML source only -->
