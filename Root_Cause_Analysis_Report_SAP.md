<p data-start="169" data-end="364"><strong data-start="169" data-end="188">Incident Title:</strong> SAP ASCS/ERS Cluster Failover Failure &ndash; NFS Share Outage<br data-start="245" data-end="248" /> <strong data-start="248" data-end="261">Customer:</strong> HPGCL<br data-start="267" data-end="270" /> <strong data-start="270" data-end="286">Environment:</strong> SAP ASCS/ERS HA Cluster &ndash; Pacemaker + Corosync &ndash; SLES<br data-start="340" data-end="343" /> <strong data-start="343" data-end="352">Date:</strong> 26-Oct-2025</p>
<hr data-start="366" data-end="369" />
<h3 data-start="371" data-end="394">📌 Incident Summary</h3>
<p data-start="395" data-end="677">The SAP ASCS (00) and ERS (10) resources became unavailable due to <strong data-start="462" data-end="496">loss of NFS share connectivity</strong> from the shared storage backend. Pacemaker attempted to stop services but stop actions timed out, resulting in <strong data-start="608" data-end="633">fail-count = INFINITY</strong> and resources entering a <strong data-start="659" data-end="670">BLOCKED</strong> state.</p>
<p data-start="679" data-end="780">Services were restored by performing cluster cleanup and restarting resources in a controlled manner.</p>
<hr data-start="782" data-end="785" />
<h3 data-start="787" data-end="811">🕒 Incident Timeline</h3>
<div class="_tableContainer_1rjym_1">
<div class="group _tableWrapper_1rjym_13 flex w-fit flex-col-reverse" tabindex="-1">
<table class="w-fit min-w-(--thread-content-width)" data-start="813" data-end="1269">
<thead data-start="813" data-end="835">
<tr data-start="813" data-end="835">
<th data-start="813" data-end="826" data-col-size="sm">Time (IST)</th>
<th data-start="826" data-end="835" data-col-size="md">Event</th>
</tr>
</thead>
<tbody data-start="858" data-end="1269">
<tr data-start="858" data-end="935">
<td data-start="858" data-end="870" data-col-size="sm"><strong data-start="860" data-end="869">16:00</strong></td>
<td data-col-size="md" data-start="870" data-end="935">Issue started: NFS became unreachable from node HPGCLECCPRDCI</td>
</tr>
<tr data-start="936" data-end="1016">
<td data-start="936" data-end="950" data-col-size="sm">16:01&ndash;16:05</td>
<td data-col-size="md" data-start="950" data-end="1016">Pacemaker monitors failed &rarr; SAPInstance stop actions triggered</td>
</tr>
<tr data-start="1017" data-end="1097">
<td data-start="1017" data-end="1031" data-col-size="sm">16:05&ndash;20:15</td>
<td data-col-size="md" data-start="1031" data-end="1097">Stop actions timed out &rarr; Resources set to <strong data-start="1075" data-end="1095">FAILED + BLOCKED</strong></td>
</tr>
<tr data-start="1098" data-end="1143">
<td data-start="1098" data-end="1106" data-col-size="sm">20:00</td>
<td data-start="1106" data-end="1143" data-col-size="md">Storage/NFS connectivity restored</td>
</tr>
<tr data-start="1144" data-end="1207">
<td data-start="1144" data-end="1152" data-col-size="sm">22:48</td>
<td data-col-size="md" data-start="1152" data-end="1207">Fail-count cleared + resources recovered and online</td>
</tr>
<tr data-start="1208" data-end="1269">
<td data-start="1208" data-end="1220" data-col-size="sm"><strong data-start="1210" data-end="1219">22:30</strong></td>
<td data-col-size="md" data-start="1220" data-end="1269">Issue fully resolved and cluster stabilized ✅</td>
</tr>
</tbody>
</table>
</div>
</div>
<p data-start="1271" data-end="1317"><strong data-start="1271" data-end="1297">Total Impact Duration:</strong> ~6 hours 30 minutes</p>
<hr data-start="1319" data-end="1322" />
<h3 data-start="1324" data-end="1348">🚨 Impact Assessment</h3>
<div class="_tableContainer_1rjym_1">
<div class="group _tableWrapper_1rjym_13 flex w-fit flex-col-reverse" tabindex="-1">
<table class="w-fit min-w-(--thread-content-width)" data-start="1349" data-end="1663">
<thead data-start="1349" data-end="1387">
<tr data-start="1349" data-end="1387">
<th data-start="1349" data-end="1361" data-col-size="sm">Component</th>
<th data-start="1361" data-end="1387" data-col-size="sm">Status During Incident</th>
</tr>
</thead>
<tbody data-start="1425" data-end="1663">
<tr data-start="1425" data-end="1453">
<td data-start="1425" data-end="1445" data-col-size="sm">SAP ASCS Instance</td>
<td data-col-size="sm" data-start="1445" data-end="1453">Down</td>
</tr>
<tr data-start="1454" data-end="1481">
<td data-start="1454" data-end="1473" data-col-size="sm">SAP ERS Instance</td>
<td data-col-size="sm" data-start="1473" data-end="1481">Down</td>
</tr>
<tr data-start="1482" data-end="1541">
<td data-start="1482" data-end="1523" data-col-size="sm">SAP Central Services High Availability</td>
<td data-col-size="sm" data-start="1523" data-end="1541">Not functional</td>
</tr>
<tr data-start="1542" data-end="1602">
<td data-start="1542" data-end="1566" data-col-size="sm">Business Transactions</td>
<td data-col-size="sm" data-start="1566" data-end="1602">Impacted (SAP logins could fail)</td>
</tr>
<tr data-start="1603" data-end="1663">
<td data-start="1603" data-end="1622" data-col-size="sm">Cluster Behavior</td>
<td data-col-size="sm" data-start="1622" data-end="1663">Resources blocked due to fail-timeout</td>
</tr>
</tbody>
</table>
</div>
</div>
<p data-start="1665" data-end="1743">No data loss reported ✅<br data-start="1688" data-end="1691" /> No node reboot occurred ✅<br data-start="1716" data-end="1719" /> Quorum remained intact ✅</p>
<hr data-start="1745" data-end="1748" />
<h3 data-start="1750" data-end="1766">✅ Root Cause</h3>
<p data-start="1767" data-end="1957"><strong data-start="1767" data-end="1807">Root Cause Type A: NFS Server Outage</strong><br data-start="1807" data-end="1810" /> The shared <code data-start="1821" data-end="1831">/usr/sap</code> filesystem became unreachable from CI node due to backend NFS storage failure, making SAPInstance monitor/status checks fail.</p>
<p data-start="1959" data-end="2010">Pacemaker marked SAPInstance as permanently failed:</p>
<blockquote data-start="2012" data-end="2062">
<p data-start="2014" data-end="2062">"Resource agent did not complete within timeout"</p>
</blockquote>
<p data-start="2064" data-end="2122">Fail-count reached <strong data-start="2083" data-end="2095">INFINITY</strong>, leading to BLOCKED state.</p>
<hr data-start="2124" data-end="2127" />
<h3 data-start="2129" data-end="2185">🔧 Resolution &amp; Recovery Actions &mdash; Performed Steps ✅</h3>
<p data-start="2187" data-end="2264">1️⃣ Restored NFS connectivity from storage team<br data-start="2234" data-end="2237" /> 2️⃣ Verified cluster state:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-bash">crm status </code></div>
</div>
<p data-start="2290" data-end="2340">3️⃣ Cleared failure state and unblocked resources:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-bash">crm resource cleanup HPE_ASCS_Group crm resource cleanup HPE_ERS_Group </code></div>
</div>
<p data-start="2426" data-end="2462">4️⃣ Verified resources auto-started:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-bash">crm resource status </code></div>
</div>
<p data-start="2497" data-end="2525">5️⃣ Validated SAP instances:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-bash">sapcontrol -nr 00 -<span class="hljs-keyword">function</span> GetProcessList sapcontrol -nr 10 -<span class="hljs-keyword">function</span> GetProcessList </code></div>
</div>
<p data-start="2626" data-end="2667">6️⃣ Confirmed cluster stability via logs:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-bash"><span class="hljs-built_in">tail</span> -f /var/log/pacemaker/pacemaker.log </code></div>
</div>
<p data-start="2723" data-end="2860">✅ SAP ASCS + ERS fully online<br data-start="2752" data-end="2755" /> ✅ No pending fail-counts<br data-start="2779" data-end="2782" /> ✅ All resources running on node HPGCLECCPRDCI<br data-start="2827" data-end="2830" /> ✅ Failover readiness confirmed</p>
<hr data-start="2862" data-end="2865" />
<h3 data-start="2867" data-end="2910">🛡 Preventive Actions &amp; Recommendations</h3>
<div class="_tableContainer_1rjym_1">
<div class="group _tableWrapper_1rjym_13 flex w-fit flex-col-reverse" tabindex="-1">
<table class="w-fit min-w-(--thread-content-width)" data-start="2912" data-end="3383">
<thead data-start="2912" data-end="2940">
<tr data-start="2912" data-end="2940">
<th data-start="2912" data-end="2929" data-col-size="md">Recommendation</th>
<th data-start="2929" data-end="2940" data-col-size="sm">Purpose</th>
</tr>
</thead>
<tbody data-start="2969" data-end="3383">
<tr data-start="2969" data-end="3058">
<td data-start="2969" data-end="3018" data-col-size="md">Implement NFS mount liveness monitoring script</td>
<td data-col-size="sm" data-start="3018" data-end="3058">Detect freeze conditions proactively</td>
</tr>
<tr data-start="3059" data-end="3140">
<td data-start="3059" data-end="3112" data-col-size="md">Reduce SAPInstance stop timeout to avoid long hang</td>
<td data-col-size="sm" data-start="3112" data-end="3140">Faster failover response</td>
</tr>
<tr data-start="3141" data-end="3214">
<td data-start="3141" data-end="3185" data-col-size="md">Enable storage-side redundancy validation</td>
<td data-col-size="sm" data-start="3185" data-end="3214">Avoid complete share drop</td>
</tr>
<tr data-start="3215" data-end="3308">
<td data-start="3215" data-end="3269" data-col-size="md">Configure Pacemaker fencing STONITH (if not active)</td>
<td data-col-size="sm" data-start="3269" data-end="3308">Prevent split-brain &amp; blocked state</td>
</tr>
<tr data-start="3309" data-end="3383">
<td data-start="3309" data-end="3353" data-col-size="md">Set proper resource dependencies ordering</td>
<td data-col-size="sm" data-start="3353" data-end="3383">Clean failover transitions</td>
</tr>
</tbody>
</table>
</div>
</div>
<p data-start="3385" data-end="3442">Optional monitoring script snippet available on request ✅</p>
<hr data-start="3444" data-end="3447" />
<h3 data-start="3449" data-end="3466">📌 Conclusion</h3>
<p data-start="3467" data-end="3657">The issue was triggered by an <strong data-start="3497" data-end="3527">external NFS storage fault</strong>, causing SAP cluster services to become blocked. Controlled recovery ensured business continuity and restored full HA capability.</p>
<!-- Comments are visible in the HTML source only -->
