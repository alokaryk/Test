<h1>Network Throttling &mdash; RCA (Root Cause Analysis) Template</h1>
<blockquote>
<p>Purpose: A reusable, step-by-step RCA-style note to investigate, document, and resolve suspected network throttling (bandwidth limits, traffic shaping, or abnormal latency). Use this as a checklist and as the basis for an incident report.</p>
</blockquote>
<hr />
<h2>1. Title &amp; Metadata</h2>
<ul>
<li>
<p><strong>Incident title:</strong> Network performance degradation / suspected throttling</p>
</li>
<li>
<p><strong>Server/Host/VM:</strong></p>
</li>
<li>
<p><strong>IP / Interface(s) observed:</strong></p>
</li>
<li>
<p><strong>Date / Time (start):</strong></p>
</li>
<li>
<p><strong>Date / Time (investigated):</strong></p>
</li>
<li>
<p><strong>Author / Investigator:</strong></p>
</li>
<li>
<p><strong>Severity:</strong> (P1 / P2 / P3)</p>
</li>
<li>
<p><strong>Stakeholders / Escalation contacts:</strong></p>
</li>
</ul>
<hr />
<h2>2. Executive Summary (1&ndash;2 sentences)</h2>
<p>Summarise what was observed, the impact, and the final conclusion (e.g., "Observed slow outbound transfers; investigation found no OS-level shaping; upstream datacenter/ISP asymmetric policy suspected"). Keep this short for management.</p>
<hr />
<h2>3. Scope &amp; Impact</h2>
<ul>
<li>
<p><strong>Services affected:</strong> (web, mail, backups, FTP, app API, etc.)</p>
</li>
<li>
<p><strong>Users impacted / Regions:</strong></p>
</li>
<li>
<p><strong>Observable symptoms:</strong> (slow downloads, slow uploads, intermittent timeouts, increased latency, retransmissions)</p>
</li>
<li>
<p><strong>Business impact:</strong> (tickets, SLA breach, revenue/time loss)</p>
</li>
</ul>
<hr />
<h2>4. Timeline of Events (fill as you go)</h2>
<table>
<thead>
<tr>
<th align="right">Time (UTC+?)</th>
<th>Action / Event</th>
<th>Command / Evidence Reference</th>
<th>Result</th>
</tr>
</thead>
<tbody>
<tr>
<td align="right">2025-10-08 12:00</td>
<td>User reports slow outbound uploads</td>
<td>Ticket #12345</td>
<td>Uploads failing &gt; 2 Mbps</td>
</tr>
</tbody>
</table>
<p><strong>Notes:</strong> Record exact timestamps and every command you ran with output references (logs, screenshots, pcap files).</p>
<hr />
<h2>5. Evidence Collected (how to collect)</h2>
<p>Collect and attach (or paste references to) outputs and logs. Keep originals.</p>
<p><strong>Suggested commands (run as root or with sudo):</strong></p>
<h3>a) Inspect traffic control (kernel shaping)</h3>
<pre><code class="language-bash"># show qdisc and classes
sudo tc qdisc show
sudo tc -s qdisc ls dev &lt;iface&gt;
sudo tc class show dev &lt;iface&gt;
sudo tc filter show dev &lt;iface&gt;
</code></pre>
<p><strong>What to look for:</strong> <code>tbf</code>, <code>htb</code>, <code>netem</code>, <code>rate</code>, or explicit classes/filters that limit bandwidth.</p>
<h3>b) Check NIC link / speed / duplex</h3>
<pre><code class="language-bash">sudo ethtool &lt;iface&gt;
# quick grep
sudo ethtool &lt;iface&gt; | grep -E "Speed|Duplex|Link"
</code></pre>
<p><strong>What to look for:</strong> <code>Speed</code> lower than expected, <code>Duplex: Half</code>, <code>Link detected: no</code>.</p>
<h3>c) Interface stats (errors/drops)</h3>
<pre><code class="language-bash">ip -s link show &lt;iface&gt;
netstat -i
ifconfig &lt;iface&gt;
</code></pre>
<p><strong>What to look for:</strong> non-zero <code>errors</code>, <code>dropped</code>, <code>overrun</code> values.</p>
<h3>d) Real-time bandwidth and active flows</h3>
<pre><code class="language-bash"># interactive
sudo iftop -i &lt;iface&gt;
sudo nload &lt;iface&gt;
sudo bmon -p &lt;iface&gt;
</code></pre>
<p><strong>What to look for:</strong> single flow pegged at a fixed value, many retransmissions, or sudden identical rate caps.</p>
<h3>e) Historical metrics</h3>
<pre><code class="language-bash"># sysstat/sar
sar -n DEV 1 10
sar -n DEV -f /var/log/sysstat/sa&lt;day&gt;
</code></pre>
<p><strong>What to look for:</strong> interface util spikes, long-term saturation.</p>
<h3>f) External throughput tests</h3>
<pre><code class="language-bash"># speedtest (may be blocked) or curl wrapper
speedtest-cli
curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python3 -
# iperf3 to a known server
iperf3 -c &lt;server&gt;
# or host your own server
# on remote: iperf3 -s
# on local: iperf3 -c &lt;remote-ip&gt;
</code></pre>
<p><strong>What to look for:</strong> consistent low upload vs download -&gt; indicates upstream shaping or asymmetric link.</p>
<h3>g) Path and latency analysis</h3>
<pre><code class="language-bash">mtr -r -c 100 &lt;destination&gt;
traceroute -n &lt;destination&gt;
ping -c 50 &lt;destination&gt;
</code></pre>
<p><strong>What to look for:</strong> packet loss at a certain hop, high and variable latency beyond the first couple hops.</p>
<h3>h) Packet capture (when needed)</h3>
<pre><code class="language-bash">sudo tcpdump -i &lt;iface&gt; -w /tmp/capture.pcap
# if storage limited, capture with filters (port 80 or host &lt;ip&gt;)
</code></pre>
<p>Use Wireshark/tshark to analyze retransmissions, dup ACKs, RSTs, CWND issues.</p>
<h3>i) Application &amp; OS checks</h3>
<pre><code class="language-bash"># sockets and connections
ss -tunap | head -n 40
# process network usage
sudo ss -p -o state established
# system I/O
iostat -x 1 3
vmstat 1 5
# check CPU and IOWAIT
sar -u 1 5
</code></pre>
<p><strong>What to look for:</strong> CPU/IO wait causing perceived network slowness; processes hitting limits.</p>
<h3>j) Firewall / NAT / Rate Limiters</h3>
<pre><code class="language-bash"># iptables / nftables
sudo iptables -L -n -v
sudo nft list ruleset
# cloud/dc ACL or firewall rules (check provider UI)
</code></pre>
<p><strong>What to look for:</strong> packet/byte counters showing policing; explicit rate-limiting rules.</p>
<h2>6. Interpretation Guide (how to read outputs)</h2>
<ul>
<li>
<p><strong><code>tc</code> shows <code>tbf</code>, <code>htb</code>, <code>netem</code></strong> &rarr; <em>local shaping</em> present. Locate class/filter and remove/adjust.</p>
</li>
<li>
<p><strong><code>fq_codel</code> only</strong> &rarr; default latency control; <em>not</em> throttling.</p>
</li>
<li>
<p><strong><code>ethtool</code> speed less than expected (e.g., 100Mb vs 1Gb)</strong> &rarr; check switch port, VM settings, or host vSwitch.</p>
</li>
<li>
<p><strong>High <code>RX/TX errors</code> or <code>dropped</code></strong> &rarr; potential NIC/sfp/switch issue or driver bug.</p>
</li>
<li>
<p><strong><code>iftop</code> shows many flows but each limited to same small rate</strong> &rarr; could be per-flow shaping (router/ISP) or congestion control behavior.</p>
</li>
<li>
<p><strong>Speedtest shows asymmetry (download &gt;&gt; upload)</strong> &rarr; upstream shaping or asymmetric plan.</p>
</li>
<li>
<p><strong><code>mtr</code> shows loss starting at provider hop</strong> &rarr; escalate to ISP/Datacenter with hop evidence.</p>
</li>
<li>
<p><strong><code>tcpdump</code> shows lots of retransmits &amp; dup acks</strong> &rarr; packet loss on path &rarr; network device or upstream issue.</p>
</li>
</ul>
<hr />
<h2>7. Common Root Causes (with examples)</h2>
<ol>
<li>
<p><strong>Local OS/Kernel shaping</strong> &mdash; <code>tc</code> rules, <code>netem</code>, or systemd-networkd config.</p>
</li>
<li>
<p><strong>Hypervisor / vSwitch limits</strong> &mdash; cloud provider applied QoS, virtual NIC bandwidth limits.</p>
</li>
<li>
<p><strong>Datacenter / ISP shaping</strong> &mdash; upstream policy enforcing download/upload limits.</p>
</li>
<li>
<p><strong>Physical link negotiation</strong> &mdash; switch port at 100Mb; NIC at full but upstream at lower speed.</p>
</li>
<li>
<p><strong>Firewall / NAT device rate-limiting</strong> &mdash; provider or edge device.</p>
</li>
<li>
<p><strong>Application-layer congestion</strong> &mdash; web server threads/process limits, upload processing slow.</p>
</li>
<li>
<p><strong>DDoS / noisy neighbors</strong> &mdash; bandwidth hogging by other tenants.</p>
</li>
<li>
<p><strong>Hardware faults</strong> &mdash; failing SFP, cable, or port errors.</p>
</li>
</ol>
<hr />
<h2>8. Corrective Actions (by root cause)</h2>
<ul>
<li>
<p><strong>If <code>tc</code> rules found:</strong></p>
<ul>
<li>
<p>Inspect with <code>tc -s qdisc ls dev &lt;iface&gt;</code> and remove/mask rule after assessing impact:</p>
<pre><code class="language-bash">sudo tc qdisc del dev &lt;iface&gt; root
</code></pre>
</li>
<li>
<p><em>Caution:</em> removing qdiscs can affect QoS; document and plan rollback.</p>
</li>
</ul>
</li>
<li>
<p><strong>If hypervisor limit discovered:</strong></p>
<ul>
<li>
<p>Open a ticket with provider; request removal/increase of vNIC bandwidth.</p>
</li>
</ul>
</li>
<li>
<p><strong>If ISP/datacenter shaping suspected:</strong></p>
<ul>
<li>
<p>Provide <code>mtr</code>/<code>traceroute</code>/speedtest evidence to provider; request clarification.</p>
</li>
</ul>
</li>
<li>
<p><strong>If link negotiation issue:</strong></p>
<ul>
<li>
<p>Force <code>ethtool -s &lt;iface&gt; speed 1000 duplex full autoneg off</code> <em>only after confirming partner switch supports it</em>.</p>
</li>
</ul>
</li>
<li>
<p><strong>If firewall/NAT rate-limiting:</strong></p>
<ul>
<li>
<p>Adjust or remove rate-limit rules in provider UI or edge device.</p>
</li>
</ul>
</li>
<li>
<p><strong>If application bottleneck:</strong></p>
<ul>
<li>
<p>Profile app (threads, DB queries), scale horizontally, or tune worker counts.</p>
</li>
</ul>
</li>
<li>
<p><strong>If packet loss / hardware error:</strong></p>
<ul>
<li>
<p>Replace cable/SFP, move VM to different host, or update NIC driver.</p>
</li>
</ul>
</li>
</ul>
<hr />
<h2>9. Validation &amp; Post-fix Checks</h2>
<p>After each remediation, validate with:</p>
<ul>
<li>
<p><code>iperf3</code> (controlled remote server)</p>
</li>
<li>
<p><code>speedtest</code> (if permitted)</p>
</li>
<li>
<p><code>iftop</code>, <code>nload</code> to watch real-time behavior</p>
</li>
<li>
<p><code>sar -n DEV</code> for historical confirmation</p>
</li>
<li>
<p>Re-run <code>tc</code>, <code>ethtool</code>, and <code>ip -s link</code> to confirm changes</p>
</li>
</ul>
<p>Document before/after results with timestamps.</p>
<hr />
<h2>10. Preventive Actions &amp; Monitoring</h2>
<ul>
<li>
<p>Add interface utilization alerts (thresholds: 70%, 85%, 95%).</p>
</li>
<li>
<p>Monitor packet drops/errors and set alerts on non-zero counts.</p>
</li>
<li>
<p>Keep a network diagram and inventory of expected link speeds.</p>
</li>
<li>
<p>Document standard <code>tc</code> rules used by the platform and keep change history (git).</p>
</li>
<li>
<p>Periodically run synthetic tests (iperf/speedtest) and log results.</p>
</li>
</ul>
<hr />
<h2>11. Communication &amp; Postmortem Template</h2>
<ul>
<li>
<p><strong>Summary of incident</strong></p>
</li>
<li>
<p><strong>Root cause</strong></p>
</li>
<li>
<p><strong>Corrective action taken</strong></p>
</li>
<li>
<p><strong>Mitigation plan</strong></p>
</li>
<li>
<p><strong>Next steps &amp; owners</strong></p>
</li>
<li>
<p><strong>ETA for permanent fix</strong></p>
</li>
</ul>
<hr />
<h2>12. Appendix &mdash; Quick Command Cheat Sheet</h2>
<pre><code class="language-bash"># traffic control
sudo tc qdisc show
sudo tc -s qdisc ls dev &lt;iface&gt;
# link
sudo ethtool &lt;iface&gt;
ip -s link show &lt;iface&gt;
# real-time
sudo iftop -i &lt;iface&gt;
nload &lt;iface&gt;
# historical
sar -n DEV 1 10
# external test
iperf3 -c &lt;server&gt;
# traceroute/mtr
mtr -r -c 100 &lt;ip&gt;
# capture
sudo tcpdump -i &lt;iface&gt; -w /tmp/cap.pcap
# sockets
ss -tunap
# firewall
sudo iptables -L -n -v
sudo nft list ruleset
</code></pre>
<hr />
<h2>13. Example Findings (fill in for your incident)</h2>
<ul>
<li>
<p><strong>Observed:</strong> Outbound upload rates ~3 Mbps vs expected &gt;100 Mbps</p>
</li>
<li>
<p><strong>Evidence:</strong> <code>speedtest</code> output (attach), <code>iperf3</code> (attach), <code>tc qdisc show</code> returned only <code>fq_codel</code>, <code>ethtool</code> 10Gb/s</p>
</li>
<li>
<p><strong>Root cause:</strong> Upstream provider asymmetric policy limiting upload</p>
</li>
<li>
<p><strong>Resolution:</strong> Escalated to provider, temporary workaround implemented (off-peak bulk transfer), planned change: purchase symmetric uplink or dedicated uplink</p>
</li>
</ul>
<hr />
<p><em>End of template &mdash; modify to fit your environment and team process.</em></p>
<!-- Comments are visible in the HTML source only -->
