<h1 data-start="201" data-end="265">Incident Report: Website Access Issue on rpmcollegepatna.ac.in</h1>
<p data-start="267" data-end="509"><strong data-start="267" data-end="276">Date:</strong> 07-Oct-2025<br data-start="288" data-end="291" /> <strong data-start="291" data-end="307">Reported by:</strong> Server Monitoring / Client Feedback<br data-start="343" data-end="346" /> <strong data-start="346" data-end="367">Affected Service:</strong> Websites hosted on the server (Nginx &rarr; Apache &rarr; PHP-FPM)<br data-start="424" data-end="427" /> <strong data-start="427" data-end="438">Server:</strong> <code data-start="439" data-end="465">mail.localhost.localhost</code> (Ubuntu 22.04 / Apache 2.4 / PHP 8.3-FPM)</p>
<hr data-start="511" data-end="514" />
<h2 data-start="516" data-end="538">1. Incident Summary</h2>
<p data-start="540" data-end="644">Users reported <strong data-start="555" data-end="585">intermittent access issues</strong> on the website <code data-start="601" data-end="624">rpmcollegepatna.ac.in</code>. Symptoms included:</p>
<ul data-start="646" data-end="813">
<li data-start="646" data-end="679">
<p data-start="648" data-end="679"><strong data-start="648" data-end="670">Connection refused</strong> errors</p>
</li>
<li data-start="680" data-end="735">
<p data-start="682" data-end="735"><strong data-start="682" data-end="711">Broken pipe / empty stdin</strong> errors in Apache logs</p>
</li>
<li data-start="736" data-end="813">
<p data-start="738" data-end="813"><strong data-start="738" data-end="759">Upstream timeouts</strong> when accessing the site through Nginx reverse proxy</p>
</li>
</ul>
<hr data-start="815" data-end="818" />
<h2 data-start="820" data-end="839">2. Investigation</h2>
<h3 data-start="841" data-end="862">2.1 Logs Observed</h3>
<p data-start="864" data-end="945"><strong data-start="864" data-end="883">Nginx error log</strong> (<code data-start="885" data-end="943">/var/log/apache2/domains/rpmcollegepatna.ac.in.error.log</code>):</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!">upstream timed <span class="hljs-keyword">out</span> (<span class="hljs-number">110</span>: <span class="hljs-keyword">Connection</span> timed <span class="hljs-keyword">out</span>) <span class="hljs-keyword">while</span> connecting <span class="hljs-keyword">to</span> upstream peer closed <span class="hljs-keyword">connection</span> <span class="hljs-keyword">in</span> SSL handshake (<span class="hljs-number">104</span>: <span class="hljs-keyword">Connection</span> <span class="hljs-keyword">reset</span> <span class="hljs-keyword">by</span> peer) <span class="hljs-keyword">connect</span>() failed (<span class="hljs-number">111</span>: <span class="hljs-keyword">Connection</span> refused) <span class="hljs-keyword">while</span> connecting <span class="hljs-keyword">to</span> upstream </code></div>
</div>
<p data-start="1176" data-end="1213"><strong data-start="1176" data-end="1191">Apache logs</strong> (<code data-start="1193" data-end="1211">proxy_fcgi:error</code>):</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!">Broken pipe: [client x.x.x.x] AH01075: <span class="hljs-keyword">Error</span> dispatching request <span class="hljs-keyword">to</span> : (sending empty stdin) </code></div>
</div>
<p data-start="1316" data-end="1340"><strong data-start="1316" data-end="1339">PHP-FPM pool status</strong>:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!">/run/php/php8<span class="hljs-number">.3</span>-fpm-rpmcollegepatna.ac.in.sock exists php8<span class="hljs-number">.3</span>-fpm service: <span class="hljs-built_in">active</span> (running) </code></div>
</div>
<p data-start="1442" data-end="1521"><strong data-start="1442" data-end="1470">Apache MPM configuration</strong> showed low <code data-start="1482" data-end="1501">MaxRequestWorkers</code> and <code data-start="1506" data-end="1520">StartServers</code>:</p>
<ul data-start="1523" data-end="1602">
<li data-start="1523" data-end="1543">
<p data-start="1525" data-end="1543">StartServers = 2</p>
</li>
<li data-start="1544" data-end="1571">
<p data-start="1546" data-end="1571">MaxRequestWorkers = 300</p>
</li>
<li data-start="1572" data-end="1602">
<p data-start="1574" data-end="1602">MaxConnectionsPerChild = 0</p>
</li>
</ul>
<hr data-start="1604" data-end="1607" />
<h3 data-start="1609" data-end="1627">2.2 Root Cause</h3>
<ol data-start="1629" data-end="1930">
<li data-start="1629" data-end="1723">
<p data-start="1632" data-end="1723">Apache <strong data-start="1639" data-end="1660">MPM worker limits</strong> were too low, causing requests to queue and fail under load.</p>
</li>
<li data-start="1724" data-end="1851">
<p data-start="1727" data-end="1851">PHP-FPM pool was running <code data-start="1752" data-end="1762">ondemand</code> with <strong data-start="1768" data-end="1791">pm.max_children = 8</strong>, which occasionally delayed handling concurrent requests.</p>
</li>
<li data-start="1852" data-end="1930">
<p data-start="1855" data-end="1930">Some PHP-FPM socket connections were stale or blocked, requiring a restart.</p>
</li>
</ol>
<hr data-start="1932" data-end="1935" />
<h2 data-start="1937" data-end="1959">3. Resolution Steps</h2>
<h3 data-start="1961" data-end="2002">3.1 Increase Apache MPM Worker Limits</h3>
<p data-start="2004" data-end="2054">File: <code data-start="2010" data-end="2054">/etc/apache2/mods-available/mpm_event.conf</code></p>
<p data-start="2056" data-end="2073"><strong data-start="2056" data-end="2073">Changes made:</strong></p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-ini">StartServers 20 ServerLimit 20 MinSpareThreads 25 MaxSpareThreads 75 ThreadLimit 64 ThreadsPerChild 25 MaxRequestWorkers 500 MaxConnectionsPerChild 1000 </code></div>
</div>
<p data-start="2306" data-end="2392">✅ Effect: Apache can now handle more concurrent connections without dropping requests.</p>
<hr data-start="2394" data-end="2397" />
<h3 data-start="2399" data-end="2422">3.2 Restart PHP-FPM</h3>
<p data-start="2424" data-end="2441">Command executed:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-bash">systemctl restart php8.3-fpm </code></div>
</div>
<p data-start="2485" data-end="2595">✅ Effect: Cleared stale PHP-FPM processes and sockets, allowing Apache to successfully connect to the backend.</p>
<hr data-start="2597" data-end="2600" />
<h3 data-start="2602" data-end="2640">3.3 Verify PHP-FPM Socket and Pool</h3>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-bash"><span class="hljs-built_in">ls</span> -l /run/php/php8.3-fpm-rpmcollegepatna.ac.in.sock </code></div>
</div>
<p data-start="2708" data-end="2759">Output confirmed correct ownership and permissions:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!">srw-rw---- <span class="hljs-number">1</span> rpm1coll www-data <span class="hljs-number">0</span> Oct <span class="hljs-number">4</span> <span class="hljs-number">17</span>:<span class="hljs-number">13</span> /run/php/php8<span class="hljs-number">.3</span>-fpm-rpmcollegepatna.ac.in.sock </code></div>
</div>
<hr data-start="2863" data-end="2866" />
<h3 data-start="2868" data-end="2902">3.4 Confirm Site Accessibility</h3>
<p data-start="2904" data-end="2917">Tested using:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-bash">curl -vk https://rpmcollegepatna.ac.in:8443/ --resolve rpmcollegepatna.ac.in:8443:192.168.0.6 </code></div>
</div>
<p data-start="3026" data-end="3083">Result: HTTP 200 OK returned, SSL handshake successful.</p>
<p data-start="3085" data-end="3131">All hosted websites were <strong data-start="3110" data-end="3130">fully accessible</strong>.</p>
<hr data-start="3133" data-end="3136" />
<h2 data-start="3138" data-end="3159">4. Recommendations</h2>
<ol data-start="3161" data-end="3195">
<li data-start="3161" data-end="3195">
<p data-start="3164" data-end="3195"><strong data-start="3164" data-end="3194">Monitor PHP-FPM pool usage</strong>:</p>
</li>
</ol>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-bash">systemctl status php8.3-fpm <span class="hljs-built_in">tail</span> -f /var/log/php8.3-fpm.log </code></div>
</div>
<ol start="2" data-start="3270" data-end="3332">
<li data-start="3270" data-end="3332">
<p data-start="3273" data-end="3332"><strong data-start="3273" data-end="3306">Consider dynamic PHP-FPM pool</strong> for high-traffic domains:</p>
</li>
</ol>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-ini"><span class="hljs-attr">pm</span> = dynamic <span class="hljs-attr">pm.max_children</span> = <span class="hljs-number">16</span> <span class="hljs-attr">pm.start_servers</span> = <span class="hljs-number">4</span> <span class="hljs-attr">pm.min_spare_servers</span> = <span class="hljs-number">2</span> <span class="hljs-attr">pm.max_spare_servers</span> = <span class="hljs-number">8</span> </code></div>
</div>
<ol start="3" data-start="3451" data-end="3619">
<li data-start="3451" data-end="3525">
<p data-start="3454" data-end="3525"><strong data-start="3454" data-end="3494">Regularly review Apache MPM settings</strong> and adjust based on traffic.</p>
</li>
<li data-start="3527" data-end="3619">
<p data-start="3530" data-end="3619">Implement <strong data-start="3540" data-end="3569">systemd timers or scripts</strong> to restart PHP-FPM if socket issues are detected.</p>
</li>
</ol>
<hr data-start="3621" data-end="3624" />
<h2 data-start="3626" data-end="3642">5. Conclusion</h2>
<p data-start="3644" data-end="3791">The incident was caused by <strong data-start="3671" data-end="3734">Apache worker limits combined with PHP-FPM pool constraints</strong>, leading to connection refusals and upstream timeouts.</p>
<p data-start="3793" data-end="3856">After updating Apache MPM configuration and restarting PHP-FPM:</p>
<ul data-start="3858" data-end="3958">
<li data-start="3858" data-end="3899">
<p data-start="3860" data-end="3899">All websites became fully accessible.</p>
</li>
<li data-start="3900" data-end="3958">
<p data-start="3902" data-end="3958">No further upstream or broken pipe errors were observed.</p>
</li>
</ul>
