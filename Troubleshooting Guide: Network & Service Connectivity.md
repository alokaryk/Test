<h2 data-path-to-node="3">1. Executive Summary &amp; RCA</h2>
<ul data-path-to-node="4">
<li>
<p data-path-to-node="4,0,0"><strong data-path-to-node="4,0,0" data-index-in-node="0">Incident:</strong> Services (MongoDB/Postgres) were accessible locally but timed out from remote locations.</p>
</li>
<li>
<p data-path-to-node="4,1,0"><strong data-path-to-node="4,1,0" data-index-in-node="0">Root Cause:</strong> <strong data-path-to-node="4,1,0" data-index-in-node="12">Implicit Deny in <code data-path-to-node="4,1,0" data-index-in-node="29">nftables</code>.</strong> While <code data-path-to-node="4,1,0" data-index-in-node="45">iptables</code> and <code data-path-to-node="4,1,0" data-index-in-node="58">UFW</code> appeared clear or inactive, the modern Ubuntu 24.04 backend (<code data-path-to-node="4,1,0" data-index-in-node="123">nftables</code>) had a whitelist configuration that dropped all traffic not explicitly defined.</p>
</li>
<li>
<p data-path-to-node="4,2,0"><strong data-path-to-node="4,2,0" data-index-in-node="0">Resolution:</strong> Inserted <code data-path-to-node="4,2,0" data-index-in-node="21">accept</code> rules for ports 2234 and 5534 at the top of the <code data-path-to-node="4,2,0" data-index-in-node="76">inet filter input</code> chain in <code data-path-to-node="4,2,0" data-index-in-node="103">nftables</code>.</p>
</li>
</ul>
<hr data-path-to-node="5" />
<h2 data-path-to-node="6">2. The 4-Step Troubleshooting Framework</h2>
<h3 data-path-to-node="7">Step 1: Verify Service Binding</h3>
<p data-path-to-node="8">Check if the application is actually listening on the network interface, not just <code data-path-to-node="8" data-index-in-node="82">localhost</code>.</p>
<ul data-path-to-node="9">
<li>
<p data-path-to-node="9,0,0"><strong data-path-to-node="9,0,0" data-index-in-node="0">Command:</strong> <code data-path-to-node="9,0,0" data-index-in-node="9">ss -tulpn | grep [PORT]</code></p>
</li>
<li>
<p data-path-to-node="9,1,0"><strong data-path-to-node="9,1,0" data-index-in-node="0">Success Criteria:</strong> Should show <code data-path-to-node="9,1,0" data-index-in-node="30">0.0.0.0:[PORT]</code> or <code data-path-to-node="9,1,0" data-index-in-node="48">*:[PORT]</code>.</p>
</li>
<li>
<p data-path-to-node="9,2,0"><strong data-path-to-node="9,2,0" data-index-in-node="0">Failure:</strong> If it shows <code data-path-to-node="9,2,0" data-index-in-node="21">127.0.0.1:[PORT]</code>, update the app config (e.g., <code data-path-to-node="9,2,0" data-index-in-node="68">bindIp</code> in Mongo or <code data-path-to-node="9,2,0" data-index-in-node="87">listen_addresses</code> in Postgres).</p>
</li>
</ul>
<h3 data-path-to-node="10">Step 2: Live Traffic Analysis (The "Truth" Test)</h3>
<p data-path-to-node="11">Use <code data-path-to-node="11" data-index-in-node="4">tcpdump</code> to see if packets are even reaching the server. This determines if the block is <strong data-path-to-node="11" data-index-in-node="92">External</strong> (Cloud Firewall) or <strong data-path-to-node="11" data-index-in-node="121">Internal</strong> (OS Firewall).</p>
<ul data-path-to-node="12">
<li>
<p data-path-to-node="12,0,0"><strong data-path-to-node="12,0,0" data-index-in-node="0">Command:</strong> <code data-path-to-node="12,0,0" data-index-in-node="9">tcpdump -ni any port [PORT]</code></p>
</li>
<li>
<p data-path-to-node="12,1,0"><strong data-path-to-node="12,1,0" data-index-in-node="0">Observation:</strong></p>
<ul data-path-to-node="12,1,1">
<li>
<p data-path-to-node="12,1,1,0,0"><strong data-path-to-node="12,1,1,0,0" data-index-in-node="0">No output:</strong> Blocked by Cloud Provider (AWS SG, DigitalOcean Firewall, etc.).</p>
</li>
<li>
<p data-path-to-node="12,1,1,1,0"><strong data-path-to-node="12,1,1,1,0" data-index-in-node="0">"In [S]" only:</strong> Packet arrives, but OS is not replying. <strong data-path-to-node="12,1,1,1,0" data-index-in-node="55">(OS Firewall Issue)</strong>.</p>
</li>
<li>
<p data-path-to-node="12,1,1,2,0"><strong data-path-to-node="12,1,1,2,0" data-index-in-node="0">"In [S]" followed by "Out [S.]":</strong> Handshake is working; issue is likely App-level authentication.</p>
</li>
</ul>
</li>
</ul>
<h3 data-path-to-node="13">Step 3: Layered Firewall Audit</h3>
<p data-path-to-node="14">Ubuntu 24.04 has multiple layers. Check them in this order:</p>
<ol start="1" data-path-to-node="15">
<li>
<p data-path-to-node="15,0,0"><strong data-path-to-node="15,0,0" data-index-in-node="0">UFW:</strong> <code data-path-to-node="15,0,0" data-index-in-node="5">ufw status</code></p>
</li>
<li>
<p data-path-to-node="15,1,0"><strong data-path-to-node="15,1,0" data-index-in-node="0">IPTables:</strong> <code data-path-to-node="15,1,0" data-index-in-node="10">iptables -nvL</code> (Check packet counters).</p>
</li>
<li>
<p data-path-to-node="15,2,0"><strong data-path-to-node="15,2,0" data-index-in-node="0">NFTables (Modern Backend):</strong> <code data-path-to-node="15,2,0" data-index-in-node="27">nft list ruleset</code></p>
<ul data-path-to-node="15,2,1">
<li>
<p data-path-to-node="15,2,1,0,0"><em data-path-to-node="15,2,1,0,0" data-index-in-node="0">Note:</em> Look for <code data-path-to-node="15,2,1,0,0" data-index-in-node="15">drop</code> or <code data-path-to-node="15,2,1,0,0" data-index-in-node="23">reject</code> rules at the end of the <code data-path-to-node="15,2,1,0,0" data-index-in-node="54">input</code> chain. Rules added via <code data-path-to-node="15,2,1,0,0" data-index-in-node="83">iptables</code> may not show the full <code data-path-to-node="15,2,1,0,0" data-index-in-node="114">nftables</code> logic.</p>
</li>
</ul>
</li>
</ol>
<h3 data-path-to-node="16">Step 4: Routing &amp; NAT Check</h3>
<p data-path-to-node="17">If traffic comes in on a Private IP (NAT), ensure the kernel isn't dropping the response.</p>
<ul data-path-to-node="18">
<li>
<p data-path-to-node="18,0,0"><strong data-path-to-node="18,0,0" data-index-in-node="0">Check Reverse Path Filtering:</strong> <code data-path-to-node="18,0,0" data-index-in-node="30">sysctl net.ipv4.conf.all.rp_filter</code></p>
</li>
<li>
<p data-path-to-node="18,1,0"><strong data-path-to-node="18,1,0" data-index-in-node="0">Fix:</strong> Set to <code data-path-to-node="18,1,0" data-index-in-node="12">0</code> if the server is confused about which interface to use for the return path.</p>
</li>
</ul>
<hr data-path-to-node="19" />
<h2 data-path-to-node="20">3. Practical Reference Commands</h2>
<table data-path-to-node="21">
<thead>
<tr>
<td><strong>Goal</strong></td>
<td><strong>Command</strong></td>
</tr>
</thead>
<tbody>
<tr>
<td><span data-path-to-node="21,1,0,0"><strong data-path-to-node="21,1,0,0" data-index-in-node="0">Open Port (Temporary)</strong></span></td>
<td><span data-path-to-node="21,1,1,0"><code data-path-to-node="21,1,1,0" data-index-in-node="0">nft insert rule inet filter input tcp dport [PORT] accept</code></span></td>
</tr>
<tr>
<td><span data-path-to-node="21,2,0,0"><strong data-path-to-node="21,2,0,0" data-index-in-node="0">Check Hidden Rules</strong></span></td>
<td><span data-path-to-node="21,2,1,0"><code data-path-to-node="21,2,1,0" data-index-in-node="0">nft -a list ruleset</code> (The <code data-path-to-node="21,2,1,0" data-index-in-node="25">-a</code> shows handles/IDs)</span></td>
</tr>
<tr>
<td><span data-path-to-node="21,3,0,0"><strong data-path-to-node="21,3,0,0" data-index-in-node="0">Delete Block Rule</strong></span></td>
<td><span data-path-to-node="21,3,1,0"><code data-path-to-node="21,3,1,0" data-index-in-node="0">nft delete rule inet filter input handle [ID]</code></span></td>
</tr>
<tr>
<td><span data-path-to-node="21,4,0,0"><strong data-path-to-node="21,4,0,0" data-index-in-node="0">Trace Packet Drop</strong></span></td>
<td><span data-path-to-node="21,4,1,0">`dmesg</span></td>
</tr>
<tr>
<td><span data-path-to-node="21,5,0,0"><strong data-path-to-node="21,5,0,0" data-index-in-node="0">Check App Logs</strong></span></td>
<td><span data-path-to-node="21,5,1,0"><code data-path-to-node="21,5,1,0" data-index-in-node="0">tail -f /var/log/mongodb/mongod.log</code></span></td>
</tr>
</tbody>
</table>
<hr data-path-to-node="22" />
<h2 data-path-to-node="23">4. Prevention &amp; Hardening Checklist</h2>
<p data-path-to-node="24">To avoid future "Black Box" connectivity issues:</p>
<ul data-path-to-node="25">
<li>
<p data-path-to-node="25,0,0">[ ] <strong data-path-to-node="25,0,0" data-index-in-node="4">Centralize Firewall Management:</strong> Use <em data-path-to-node="25,0,0" data-index-in-node="40">either</em> <code data-path-to-node="25,0,0" data-index-in-node="47">UFW</code> or <code data-path-to-node="25,0,0" data-index-in-node="54">nftables</code> directly. Mixing them causes "invisible" blocks.</p>
</li>
<li>
<p data-path-to-node="25,1,0">[ ] <strong data-path-to-node="25,1,0" data-index-in-node="4">Persistent Config:</strong> Ensure rules are added to <code data-path-to-node="25,1,0" data-index-in-node="49">/etc/nftables.conf</code> or <code data-path-to-node="25,1,0" data-index-in-node="71">/etc/network/iptables.rules</code>.</p>
</li>
<li>
<p data-path-to-node="25,2,0">[ ] <strong data-path-to-node="25,2,0" data-index-in-node="4">Security Groups:</strong> Always double-check the Hosting Provider&rsquo;s external dashboard before troubleshooting the OS.</p>
</li>
<li>
<p data-path-to-node="25,3,0">[ ] <strong data-path-to-node="25,3,0" data-index-in-node="4">Principle of Least Privilege:</strong> Never use <code data-path-to-node="25,3,0" data-index-in-node="44">0.0.0.0/0</code> for databases. Use a VPN or specific IP whitelisting:</p>
<ul data-path-to-node="25,3,1">
<li>
<p data-path-to-node="25,3,1,0,0"><em data-path-to-node="25,3,1,0,0" data-index-in-node="0">Example:</em> <code data-path-to-node="25,3,1,0,0" data-index-in-node="9">nft insert rule inet filter input ip saddr [ADMIN_IP] tcp dport 2234 accept</code></p>
</li>
</ul>
</li>
<li>
<p>root@mail:~# iptables -F<br />root@mail:~# nft list ruleset<br />table inet filter {<br /> chain input {<br /> type filter hook input priority filter; policy accept;<br /> iif "lo" accept<br /> ip protocol icmp icmp type echo-request limit rate over 10/second burst 4 packets drop<br /> ip6 nexthdr ipv6-icmp icmpv6 type echo-request limit rate over 10/second burst 4 packets drop<br /> ct state established,related accept<br /> ip6 nexthdr ipv6-icmp icmpv6 type { destination-unreachable, packet-too-big, time-exceeded, parameter-problem, echo-request, mld-listener-query, mld-listener-report, mld-listener-done, nd-router-solicit, nd-router-advert, nd-neighbor-solicit, nd-neighbor-advert, ind-neighbor-solicit, ind-neighbor-advert, mld2-listener-report } accept<br /> ip protocol icmp icmp type { destination-unreachable, echo-request, router-advertisement, router-solicitation, time-exceeded, parameter-problem } accept<br /> ip protocol igmp accept<br /> tcp dport 2232 accept<br /> tcp dport 80 accept<br /> tcp dport 443 accept<br /> tcp dport 25 accept<br /> tcp dport 587 accept<br /> tcp dport 465 accept<br /> tcp dport 110 accept<br /> tcp dport 995 accept<br /> tcp dport 143 accept<br /> tcp dport 993 accept<br /> counter packets 9940 bytes 699540 drop<br /> }</p>
<p>chain output {<br /> type filter hook output priority filter; policy accept;<br /> }</p>
<p>chain forward {<br /> type filter hook forward priority filter; policy drop;<br /> }<br />}<br />table inet f2b-table {<br /> set addr-set-sshd {<br /> type ipv4_addr<br /> }</p>
<p>set addr-set-pregreet {<br /> type ipv4_addr<br /> }</p>
<p>set addr-set-postfix {<br /> type ipv4_addr<br /> elements = { 45.144.212.169, 158.94.211.170 }<br /> }</p>
<p>chain f2b-chain {<br /> type filter hook input priority filter - 1; policy accept;<br /> tcp dport 2232 ip saddr @addr-set-sshd reject with icmp port-unreachable<br /> tcp dport { 25, 80, 110, 143, 443, 465, 587, 993, 995, 4190 } ip saddr @addr-set-pregreet reject with icmp port-unreachable<br /> tcp dport { 25, 80, 110, 143, 443, 465, 587, 993, 995, 4190 } ip saddr @addr-set-postfix reject with icmp port-unreachable<br /> }<br />}<br />table ip filter {<br /> chain INPUT {<br /> type filter hook input priority filter; policy accept;<br /> }</p>
<p>chain OUTPUT {<br /> type filter hook output priority filter; policy accept;<br /> }<br />}<br />table ip6 filter {<br />}<br />root@mail:~# nft add rule inet filter input tcp dport 2234 accept<br />root@mail:~# nft add rule inet filter input tcp dport 5534 accept<br />root@mail:~# nft list ruleset | grep dport<br /> tcp dport 2232 accept<br /> tcp dport 80 accept<br /> tcp dport 443 accept<br /> tcp dport 25 accept<br /> tcp dport 587 accept<br /> tcp dport 465 accept<br /> tcp dport 110 accept<br /> tcp dport 995 accept<br /> tcp dport 143 accept<br /> tcp dport 993 accept<br /> tcp dport 2234 accept<br /> tcp dport 5534 accept<br /> tcp dport 2232 ip saddr @addr-set-sshd reject with icmp port-unreachable<br /> tcp dport { 25, 80, 110, 143, 443, 465, 587, 993, 995, 4190 } ip saddr @addr-set-pregreet reject with icmp port-unreachable<br /> tcp dport { 25, 80, 110, 143, 443, 465, 587, 993, 995, 4190 } ip saddr @addr-set-postfix reject with icmp port-unreachable<br />root@mail:~# nft insert rule inet filter input tcp dport 2234 accept<br />root@mail:~# nft insert rule inet filter input tcp dport 5534 accept<br />root@mail:~# nft -a list ruleset inet<br />table inet filter { # handle 5<br /> chain input { # handle 1<br /> type filter hook input priority filter; policy accept;<br /> tcp dport 5534 accept # handle 27<br /> tcp dport 2234 accept # handle 26<br /> iif "lo" accept # handle 4<br /> ip protocol icmp icmp type echo-request limit rate over 10/second burst 4 packets drop # handle 5<br /> ip6 nexthdr ipv6-icmp icmpv6 type echo-request limit rate over 10/second burst 4 packets drop # handle 6<br /> ct state established,related accept # handle 7<br /> ip6 nexthdr ipv6-icmp icmpv6 type { destination-unreachable, packet-too-big, time-exceeded, parameter-problem, echo-request, mld-listener-query, mld-listener-report, mld-listener-done, nd-router-solicit, nd-router-advert, nd-neighbor-solicit, nd-neighbor-advert, ind-neighbor-solicit, ind-neighbor-advert, mld2-listener-report } accept # handle 9<br /> ip protocol icmp icmp type { destination-unreachable, echo-request, router-advertisement, router-solicitation, time-exceeded, parameter-problem } accept # handle 11<br /> ip protocol igmp accept # handle 12<br /> tcp dport 2232 accept # handle 13<br /> tcp dport 80 accept # handle 14<br /> tcp dport 443 accept # handle 15<br /> tcp dport 25 accept # handle 16<br /> tcp dport 587 accept # handle 17<br /> tcp dport 465 accept # handle 18<br /> tcp dport 110 accept # handle 19<br /> tcp dport 995 accept # handle 20<br /> tcp dport 143 accept # handle 21<br /> tcp dport 993 accept # handle 22<br /> counter packets 9942 bytes 699656 drop # handle 23<br /> tcp dport 2234 accept # handle 24<br /> tcp dport 5534 accept # handle 25<br /> }</p>
<p>chain output { # handle 2<br /> type filter hook output priority filter; policy accept;<br /> }</p>
<p>chain forward { # handle 3<br /> type filter hook forward priority filter; policy drop;<br /> }<br />}<br />table inet f2b-table { # handle 6<br /> set addr-set-sshd { # handle 2<br /> type ipv4_addr<br /> }</p>
<p>set addr-set-pregreet { # handle 5<br /> type ipv4_addr<br /> }</p>
<p>set addr-set-postfix { # handle 8<br /> type ipv4_addr<br /> elements = { 45.144.212.169, 158.94.211.170 }<br /> }</p>
<p>chain f2b-chain { # handle 1<br /> type filter hook input priority filter - 1; policy accept;<br /> tcp dport 2232 ip saddr @addr-set-sshd reject with icmp port-unreachable # handle 4<br /> tcp dport { 25, 80, 110, 143, 443, 465, 587, 993, 995, 4190 } ip saddr @addr-set-pregreet reject with icmp port-unreachable # handle 7<br /> tcp dport { 25, 80, 110, 143, 443, 465, 587, 993, 995, 4190 } ip saddr @addr-set-postfix reject with icmp port-unreachable # handle 10<br /> }<br />}<br />root@mail:~#</p>
</li>
</ul>
