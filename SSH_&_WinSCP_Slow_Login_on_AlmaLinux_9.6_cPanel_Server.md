<h1 data-start="111" data-end="195">✅ <strong data-start="115" data-end="195">Root Cause Analysis &ndash; SSH &amp; WinSCP Slow Login on AlmaLinux 9.6 cPanel Server</strong></h1>
<h3 data-start="197" data-end="218"><strong data-start="201" data-end="218">Issue Summary</strong></h3>
<p data-start="219" data-end="237">Users experienced:</p>
<ul data-start="238" data-end="406">
<li data-start="238" data-end="272">
<p data-start="240" data-end="272">Very slow SSH login from PuTTY</p>
</li>
<li data-start="273" data-end="346">
<p data-start="275" data-end="346">WinSCP showing <em data-start="290" data-end="344">&ldquo;host is not communicating for more than 15 seconds&rdquo;</em></p>
</li>
<li data-start="347" data-end="406">
<p data-start="349" data-end="406">SSH sessions taking unusually long before password prompt</p>
</li>
</ul>
<h3 data-start="408" data-end="433"><strong data-start="412" data-end="433">Symptoms Observed</strong></h3>
<p data-start="434" data-end="469">System logs showed repeated errors:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!">pam_systemd(sshd:<span class="hljs-keyword">session</span>): Failed <span class="hljs-keyword">to</span> <span class="hljs-keyword">create</span> <span class="hljs-keyword">session</span>: <span class="hljs-keyword">Connection</span> timed <span class="hljs-keyword">out</span> </code></div>
</div>
<p data-start="554" data-end="617">SSH logins were delayed until this PAM/systemd timeout expired.</p>
<p data-start="619" data-end="670">I/O, CPU, RAM were normal based on <code data-start="654" data-end="662">iostat</code> output.</p>
<p data-start="672" data-end="736">SSH became normal immediately after restarting <code data-start="719" data-end="735">systemd-logind</code>.</p>
<h3 data-start="738" data-end="756"><strong data-start="742" data-end="756">Root Cause</strong></h3>
<p data-start="757" data-end="886">The underlying cause was a <strong data-start="784" data-end="816">hang/stall in systemd-logind</strong> &mdash; the component responsible for creating user login sessions via PAM.</p>
<p data-start="888" data-end="931">When <code data-start="893" data-end="909">systemd-logind</code> becomes unresponsive:</p>
<ul data-start="932" data-end="1062">
<li data-start="932" data-end="976">
<p data-start="934" data-end="976">SSH waits for session creation through PAM</p>
</li>
<li data-start="977" data-end="1024">
<p data-start="979" data-end="1024">This causes up to <strong data-start="997" data-end="1012">~30 seconds</strong> login delay</p>
</li>
<li data-start="1025" data-end="1062">
<p data-start="1027" data-end="1062">WinSCP connection attempts time out</p>
</li>
</ul>
<p data-start="1064" data-end="1096">This issue is known to occur on:</p>
<ul data-start="1097" data-end="1209">
<li data-start="1097" data-end="1127">
<p data-start="1099" data-end="1127">AlmaLinux/RHEL 9.x systems</p>
</li>
<li data-start="1128" data-end="1160">
<p data-start="1130" data-end="1160">cPanel servers after updates</p>
</li>
<li data-start="1161" data-end="1209">
<p data-start="1163" data-end="1209">When the D-Bus or systemd-logind daemon stalls</p>
</li>
</ul>
<h3 data-start="1211" data-end="1240"><strong data-start="1215" data-end="1240">Technical Explanation</strong></h3>
<p data-start="1241" data-end="1281">SSH session creation follows this chain:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!"><span class="hljs-variable">sshd</span> &rarr; <span class="hljs-variable">PAM</span> &rarr; <span class="hljs-type">pam_systemd</span><span class="hljs-operator">.</span><span class="hljs-variable">so</span> &rarr; <span class="hljs-variable">systemd</span><span class="hljs-operator">-</span><span class="hljs-variable">logind</span> &rarr; <span class="hljs-built_in">D</span><span class="hljs-operator">-</span><span class="hljs-variable">Bus</span> </code></div>
</div>
<p data-start="1345" data-end="1362">In this incident:</p>
<ul data-start="1363" data-end="1497">
<li data-start="1363" data-end="1446">
<p data-start="1365" data-end="1446"><code data-start="1365" data-end="1381">systemd-logind</code> was running but <strong data-start="1398" data-end="1421">inside a hung state</strong>, not processing requests</p>
</li>
<li data-start="1447" data-end="1497">
<p data-start="1449" data-end="1497">PAM waited for a response &rarr; timeout &rarr; slow login</p>
</li>
</ul>
<p data-start="1499" data-end="1546">Restarting the service cleared the stuck state:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!"><span class="hljs-attribute">systemctl</span> restart systemd-logind </code></div>
</div>
<p data-start="1590" data-end="1638">Immediately restoring normal SSH responsiveness.</p>
<h3 data-start="1640" data-end="1669"><strong data-start="1644" data-end="1669">Immediate Fix Applied</strong></h3>
<ul data-start="1670" data-end="1801">
<li data-start="1670" data-end="1700">
<p data-start="1672" data-end="1700">Restarted <code data-start="1682" data-end="1698">systemd-logind</code></p>
</li>
<li data-start="1701" data-end="1749">
<p data-start="1703" data-end="1749">Verified successful session creation in logs</p>
</li>
<li data-start="1750" data-end="1801">
<p data-start="1752" data-end="1801">SSH &amp; WinSCP login performance returned to normal</p>
</li>
</ul>
<h3 data-start="1803" data-end="1828"><strong data-start="1807" data-end="1828">Corrective Action</strong></h3>
<ul data-start="1829" data-end="1969">
<li data-start="1829" data-end="1906">
<p data-start="1831" data-end="1906">Monitored <code data-start="1841" data-end="1857">systemd-logind</code> post-restart (<code data-start="1872" data-end="1905">systemctl status systemd-logind</code>)</p>
</li>
<li data-start="1907" data-end="1969">
<p data-start="1909" data-end="1969">Confirmed &ldquo;Processing requests&hellip;&rdquo; and successful new sessions</p>
</li>
</ul>
<h3 data-start="1971" data-end="1997"><strong data-start="1975" data-end="1997">Preventive Actions</strong></h3>
<p data-start="1998" data-end="2018">To avoid recurrence:</p>
<ol data-start="2019" data-end="2433">
<li data-start="2019" data-end="2084">
<p data-start="2022" data-end="2053">Ensure system is fully updated:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!">dnf <span class="hljs-keyword">update</span> <span class="hljs-operator">-</span>y </code></div>
</div>
</li>
<li data-start="2085" data-end="2131">
<p data-start="2088" data-end="2131">Avoid forced reboots during cPanel updates.</p>
</li>
<li data-start="2132" data-end="2217">
<p data-start="2135" data-end="2217">Monitor <code data-start="2143" data-end="2162">/var/log/messages</code> &amp; <code data-start="2165" data-end="2195">journalctl -u systemd-logind</code> for recurring stalls.</p>
</li>
<li data-start="2218" data-end="2360">
<p data-start="2221" data-end="2310">(Optional Mitigation) Disable systemd session creation in SSH if this happens frequently:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!"><span class="hljs-meta">#session required pam_systemd.so</span> </code></div>
</div>
</li>
<li data-start="2361" data-end="2433">
<p data-start="2364" data-end="2433">Ensure no external software or security module interferes with D-Bus.</p>
</li>
</ol>
<hr data-start="2435" data-end="2438" />
<h1 data-start="2440" data-end="2466">✔️ <strong data-start="2445" data-end="2466">Final RCA Summary</strong></h1>
<p data-start="2468" data-end="2658"><strong data-start="2468" data-end="2658">The root cause of the slow SSH/WinSCP login was a hung systemd-logind service, which caused PAM session creation to time out. Restarting systemd-logind restored normal login performance.</strong></p>
<!-- Comments are visible in the HTML source only -->
