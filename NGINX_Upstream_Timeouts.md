<h3 data-start="615" data-end="655">2️⃣ Nginx &rarr; Apache &rarr; PHP-FPM problem</h3>
<ul data-start="657" data-end="794">
<li data-start="657" data-end="737">
<p data-start="659" data-end="737">Nginx is <strong data-start="668" data-end="695">proxying HTTPS requests</strong> to Apache (<code data-start="707" data-end="733">https://192.168.0.6:8443</code>).</p>
</li>
<li data-start="738" data-end="779">
<p data-start="740" data-end="779">Apache serves PHP via <strong data-start="762" data-end="776">FPM socket</strong>.</p>
</li>
<li data-start="780" data-end="794">
<p data-start="782" data-end="794">Errors show:</p>
</li>
</ul>
<pre class="overflow-visible!" data-start="796" data-end="980">&nbsp;upstream timed out (110: Connection timed out) while connecting to upstream<br />peer closed connection in SSL handshake (104: Connection reset by peer)<br />proxy_fcgi:error Broken pipe<br /><br /></pre>
<p data-start="982" data-end="1112">This means <strong data-start="993" data-end="1050">Apache isn&rsquo;t returning the response properly to Nginx</strong>, likely because PHP-FPM is either too slow or hitting limits.</p>
<p data-start="982" data-end="1112">&nbsp;</p>
<p data-start="982" data-end="1112">systemctl restart php8.3-fpm</p>
<p data-start="982" data-end="1112">&nbsp;</p>
<h3 data-start="115" data-end="141">Apache MPM Changes</h3>
<p data-start="142" data-end="244">Your <strong data-start="147" data-end="167"><code data-start="149" data-end="165">mpm_event.conf</code></strong> changes increased the number of worker threads and maximum request handling:</p>
<table style="width: 378px;">
<thead>
<tr>
<th style="width: 134px;">Directive</th>
<th style="width: 107.896px;">Before</th>
<th style="width: 115.104px;">After</th>
</tr>
</thead>
<tbody>
<tr>
<td style="width: 134px;">StartServers</td>
<td style="width: 107.896px;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;20</td>
<td style="width: 115.104px;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;20</td>
</tr>
<tr>
<td style="width: 134px;">MaxRequestWorkers</td>
<td style="width: 107.896px;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;300</td>
<td style="width: 115.104px;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;500</td>
</tr>
<tr>
<td style="width: 134px;">MaxConnectionsPerChild</td>
<td style="width: 107.896px;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0</td>
<td style="width: 115.104px;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;1000</td>
</tr>
</tbody>
</table>
<p>&nbsp;</p>
<p data-start="406" data-end="577">✅ Effect: Apache can now handle more concurrent connections without dropping requests, reducing <strong data-start="502" data-end="524">connection refused</strong> and <strong data-start="529" data-end="544">broken pipe</strong> errors when proxying to PHP-FPM.</p>
<hr data-start="579" data-end="582" />
<h3 data-start="584" data-end="604">2️⃣ PHP-FPM Pool</h3>
<p data-start="605" data-end="657">Your <strong data-start="610" data-end="642"><code data-start="612" data-end="640">rpmcollegepatna.ac.in.conf</code></strong> pool is still:</p>
<p data-start="605" data-end="657">pm = ondemand<br />pm.max_children = 8<br />pm.max_requests = 4000<br />pm.process_idle_timeout = 10s</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-ini"><span class="hljs-attr">pm</span> = <span class="hljs-literal">on</span>demand <span class="hljs-attr">pm.max_children</span> = <span class="hljs-number">8</span> <span class="hljs-attr">pm.max_requests</span> = <span class="hljs-number">4000</span> <span class="hljs-attr">pm.process_idle_timeout</span> = <span class="hljs-number">10</span>s </code></div>
</div>
<ul data-start="758" data-end="990">
<li data-start="758" data-end="922">
<p data-start="760" data-end="922">PHP-FPM was running fine, but sometimes under load the <code data-start="815" data-end="825">ondemand</code> children limit (8) can cause temporary request delays if too many concurrent requests come in.</p>
</li>
<li data-start="923" data-end="990">
<p data-start="925" data-end="990">Restarting PHP-FPM cleared any stuck processes and freed sockets.</p>
</li>
</ul>
<p data-start="992" data-end="1109">✅ Effect: PHP-FPM is now accepting new connections properly, so Apache can serve the requests through the FPM socket.</p>
<hr data-start="1111" data-end="1114" />
<h3 data-start="1116" data-end="1136">3️⃣ Observations</h3>
<ul data-start="1138" data-end="1312">
<li data-start="1138" data-end="1171">
<p data-start="1140" data-end="1171">All websites are now working.</p>
</li>
<li data-start="1172" data-end="1242">
<p data-start="1174" data-end="1242">Nginx &rarr; Apache &rarr; PHP-FPM proxy no longer shows broken pipe errors.</p>
</li>
<li data-start="1243" data-end="1312">
<p data-start="1245" data-end="1312">Apache can handle higher load due to increased MPM worker settings.</p>
</li>
</ul>
<hr data-start="1314" data-end="1317" />
<h3 data-start="1319" data-end="1356">4️⃣ Recommendations for Stability</h3>
<ol data-start="1358" data-end="1941">
<li data-start="1358" data-end="1586">
<p data-start="1361" data-end="1387"><strong data-start="1361" data-end="1386">Monitor PHP-FPM usage</strong>:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-bash">systemctl status php8.3-fpm <span class="hljs-built_in">tail</span> -f /var/log/php8.3-fpm.log </code></div>
</div>
<p data-start="1475" data-end="1586">If you see <code data-start="1486" data-end="1518">server reached pm.max_children</code>, consider increasing <code data-start="1540" data-end="1557">pm.max_children</code> to <strong data-start="1561" data-end="1570">12&ndash;16</strong> for busy sites.</p>
</li>
<li data-start="1588" data-end="1724">
<p data-start="1591" data-end="1620"><strong data-start="1591" data-end="1619">Check Apache concurrency</strong>:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-bash">apachectl fullstatus </code></div>
</div>
<p data-start="1666" data-end="1724">Ensure <code data-start="1673" data-end="1692">MaxRequestWorkers</code> is sufficient for peak traffic.</p>
</li>
<li data-start="1726" data-end="1817">
<p data-start="1729" data-end="1817"><strong data-start="1729" data-end="1758">Regularly restart PHP-FPM</strong> (or use <code data-start="1767" data-end="1776">systemd</code> timers) if you notice socket timeouts.</p>
</li>
<li data-start="1819" data-end="1941">
<p data-start="1822" data-end="1941"><strong data-start="1822" data-end="1834">Optional</strong>: Use <strong data-start="1840" data-end="1858"><code data-start="1842" data-end="1856">pm = dynamic</code></strong> for high-traffic sites instead of <code data-start="1893" data-end="1903">ondemand</code> for better responsiveness under load.</p>
</li>
</ol>
