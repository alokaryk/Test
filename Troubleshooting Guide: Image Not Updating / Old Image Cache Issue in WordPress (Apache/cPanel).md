<p class="demoTitle"><strong data-start="247" data-end="266">Applicable for:</strong> Websites hosted on WHM/cPanel with Apache, no LiteSpeed installed</p>
<h2 data-start="435" data-end="458"><strong data-start="438" data-end="458">1. Issue Summary</strong></h2>
<p data-start="459" data-end="653">A new image or updated file uploaded to WordPress does <strong data-start="514" data-end="545">not reflect on the frontend</strong>, even after clearing the browser cache.<br data-start="585" data-end="588" /> The URL continues showing an <strong data-start="617" data-end="639">old cached version</strong> of the image.</p>
<h3 data-start="655" data-end="671"><strong data-start="659" data-end="671">Symptoms</strong></h3>
<ul data-start="672" data-end="996">
<li data-start="672" data-end="710">
<p data-start="674" data-end="710">Direct file access shows old image</p>
</li>
<li data-start="711" data-end="754">
<p data-start="713" data-end="754">WordPress media library shows new image</p>
</li>
<li data-start="755" data-end="824">
<p data-start="757" data-end="824"><code data-start="757" data-end="794">x-litespeed-cache-control: no-cache</code> header unexpectedly appears</p>
</li>
<li data-start="825" data-end="874">
<p data-start="827" data-end="874">No LiteSpeed Web Server installed&mdash;only Apache</p>
</li>
<li data-start="875" data-end="925">
<p data-start="877" data-end="925">No active caching or WebP plugins in WordPress</p>
</li>
<li data-start="926" data-end="996">
<p data-start="928" data-end="996">After some actions by the application team, updates start reflecting</p>
</li>
</ul>
<hr data-start="998" data-end="1001" />
<h2 data-start="1003" data-end="1033"><strong data-start="1006" data-end="1033">2. Root Causes (Common)</strong></h2>
<p data-start="1034" data-end="1098">This issue generally occurs due to one or more of the following:</p>
<h3 data-start="1100" data-end="1133"><strong data-start="1104" data-end="1133">2.1 WordPress-Level Cache</strong></h3>
<p data-start="1134" data-end="1201">Even if caching plugins are removed, their cached files may remain:</p>
<ul data-start="1202" data-end="1369">
<li data-start="1202" data-end="1225">
<p data-start="1204" data-end="1225"><code data-start="1204" data-end="1223">wp-content/cache/</code></p>
</li>
<li data-start="1226" data-end="1261">
<p data-start="1228" data-end="1261"><code data-start="1228" data-end="1259">wp-content/uploads/litespeed/</code></p>
</li>
<li data-start="1262" data-end="1287">
<p data-start="1264" data-end="1287">Object cache remnants</p>
</li>
<li data-start="1288" data-end="1369">
<p data-start="1290" data-end="1369">Page cache stored by plugin previously installed (e.g., LiteSpeed Cache plugin)</p>
</li>
</ul>
<h3 data-start="1371" data-end="1419"><strong data-start="1375" data-end="1419">2.2 Server-Level Cache Misinterpretation</strong></h3>
<p data-start="1420" data-end="1509">If LiteSpeed Cache plugin <strong data-start="1446" data-end="1471">was installed earlier</strong>, Apache may generate LSCache headers:</p>
<ul data-start="1510" data-end="1569">
<li data-start="1510" data-end="1549">
<p data-start="1512" data-end="1549"><code data-start="1512" data-end="1549">x-litespeed-cache-control: no-cache</code></p>
</li>
<li data-start="1550" data-end="1569">
<p data-start="1552" data-end="1569"><code data-start="1552" data-end="1569">x-litespeed-tag</code></p>
</li>
</ul>
<p data-start="1571" data-end="1630">These are plugin headers, <strong data-start="1597" data-end="1629">not LiteSpeed server headers</strong>.</p>
<h3 data-start="1632" data-end="1667"><strong data-start="1636" data-end="1667">2.3 Browser/Proxy/CDN Cache</strong></h3>
<p data-start="1668" data-end="1694">Long TTL headers returned:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!"><span class="hljs-section">cache-control: max-age=31536000</span> <span class="hljs-section">expires: +1 year</span> </code></div>
</div>
<p data-start="1752" data-end="1804">Any proxy or device may continue serving stale file.</p>
<h3 data-start="1806" data-end="1880"><strong data-start="1810" data-end="1880">2.4 WordPress Image Optimization Plugins (even previously removed)</strong></h3>
<p data-start="1881" data-end="1905">Folders may still exist:</p>
<ul data-start="1906" data-end="2002">
<li data-start="1906" data-end="1940">
<p data-start="1908" data-end="1940"><code data-start="1908" data-end="1938">/wp-content/uploads/2024/12/</code></p>
</li>
<li data-start="1941" data-end="1981">
<p data-start="1943" data-end="1981"><code data-start="1943" data-end="1954">.htaccess</code> rules created by plugins</p>
</li>
<li data-start="1982" data-end="2002">
<p data-start="1984" data-end="2002">WebP cache folders</p>
</li>
</ul>
<h3 data-start="2004" data-end="2045"><strong data-start="2008" data-end="2045">2.5 Wrong File Replacement Method</strong></h3>
<p data-start="2046" data-end="2080">Developers may replace images via:</p>
<ul data-start="2081" data-end="2112">
<li data-start="2081" data-end="2104">
<p data-start="2083" data-end="2104">cPanel File Manager</p>
</li>
<li data-start="2105" data-end="2112">
<p data-start="2107" data-end="2112">FTP</p>
</li>
</ul>
<p data-start="2114" data-end="2199">If permissions differ or the file exists in multiple versions, old one may be served.</p>
<hr data-start="2201" data-end="2204" />
<h2 data-start="2206" data-end="2254"><strong data-start="2209" data-end="2254">3. Step-by-Step Troubleshooting Procedure</strong></h2>
<hr data-start="2256" data-end="2259" />
<h3 data-start="2261" data-end="2306"><strong data-start="2265" data-end="2306">Step 1 &mdash; Verify Actual File on Server</strong></h3>
<p data-start="2307" data-end="2327">SSH or File Manager:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!"><span class="hljs-built_in">ls</span> -l /home/USERNAME/public_html/wp-content/uploads/2024/12/jaipuriamba-hdr.jpg </code></div>
</div>
<p data-start="2418" data-end="2424">Check:</p>
<ul data-start="2425" data-end="2490">
<li data-start="2425" data-end="2447">
<p data-start="2427" data-end="2447">Last modified date</p>
</li>
<li data-start="2448" data-end="2461">
<p data-start="2450" data-end="2461">File size</p>
</li>
<li data-start="2462" data-end="2490">
<p data-start="2464" data-end="2490">Re-upload file and compare</p>
</li>
</ul>
<hr data-start="2492" data-end="2495" />
<h3 data-start="2497" data-end="2545"><strong data-start="2501" data-end="2545">Step 2 &mdash; Check for Duplicate Image Names</strong></h3>
<p data-start="2546" data-end="2573">Sometimes WordPress stores:</p>
<ul data-start="2574" data-end="2661">
<li data-start="2574" data-end="2597">
<p data-start="2576" data-end="2597"><code data-start="2576" data-end="2597">jaipuriamba-hdr.jpg</code></p>
</li>
<li data-start="2598" data-end="2628">
<p data-start="2600" data-end="2628"><code data-start="2600" data-end="2628">jaipuriamba-hdr-scaled.jpg</code></p>
</li>
<li data-start="2629" data-end="2661">
<p data-start="2631" data-end="2661"><code data-start="2631" data-end="2661">jaipuriamba-hdr-1536x864.jpg</code></p>
</li>
</ul>
<p data-start="2663" data-end="2701">Verify actual file being used in HTML.</p>
<hr data-start="2703" data-end="2706" />
<h3 data-start="2708" data-end="2758"><strong data-start="2712" data-end="2758">Step 3 &mdash; Test With Cache-Busting Parameter</strong></h3>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!">https://domain.com/wp-content/uploads/2024/12/jaipuriamba-hdr.jpg?<span class="hljs-built_in">test</span>=123 </code></div>
</div>
<p data-start="2842" data-end="2947">If new image loads here &rarr; <strong data-start="2868" data-end="2884">server cache</strong> is not the issue; caching is at <strong data-start="2917" data-end="2932">client side</strong> or via plugin.</p>
<hr data-start="2949" data-end="2952" />
<h3 data-start="2954" data-end="2993"><strong data-start="2958" data-end="2993">Step 4 &mdash; Check Response Headers</strong></h3>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!">curl -I https://domain.com/wp-content/uploads/2024/12/jaipuriamba-hdr.jpg </code></div>
</div>
<p data-start="3077" data-end="3087">Check for:</p>
<ul data-start="3089" data-end="3238">
<li data-start="3089" data-end="3121">
<p data-start="3091" data-end="3121"><code data-start="3091" data-end="3106">cache-control</code> long max-age</p>
</li>
<li data-start="3122" data-end="3179">
<p data-start="3124" data-end="3179">Unexpected <code data-start="3135" data-end="3162">x-litespeed-cache-control</code> (plugin-related)</p>
</li>
<li data-start="3180" data-end="3206">
<p data-start="3182" data-end="3206"><code data-start="3182" data-end="3197">last-modified</code> header</p>
</li>
<li data-start="3207" data-end="3238">
<p data-start="3209" data-end="3238">CDN headers (Cloudflare etc.)</p>
</li>
</ul>
<hr data-start="3240" data-end="3243" />
<h3 data-start="3245" data-end="3309"><strong data-start="3249" data-end="3309">Step 5 &mdash; Remove LiteSpeed Cache Plugin Folder Completely</strong></h3>
<p data-start="3310" data-end="3376">Even if plugin is removed from WordPress, folder may still remain:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!"><span class="hljs-built_in">rm</span> -rf wp-content/litespeed/ <span class="hljs-built_in">rm</span> -rf wp-content/plugins/litespeed-cache/ <span class="hljs-built_in">rm</span> -rf wp-content/uploads/litespeed/ </code></div>
</div>
<hr data-start="3496" data-end="3499" />
<h3 data-start="3501" data-end="3546"><strong data-start="3505" data-end="3546">Step 6 &mdash; Clear WordPress Object Cache</strong></h3>
<p data-start="3547" data-end="3577">If Redis or Memcached is used:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!">redis-cli FLUSHALL </code></div>
</div>
<p data-start="3606" data-end="3632">(or through hosting panel)</p>
<hr data-start="3634" data-end="3637" />
<h3 data-start="3639" data-end="3671"><strong data-start="3643" data-end="3671">Step 7 &mdash; Reset .htaccess</strong></h3>
<p data-start="3672" data-end="3708">Sometimes plugins add caching rules.</p>
<p data-start="3710" data-end="3729">Rename <code data-start="3717" data-end="3728">.htaccess</code>:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!"><span class="hljs-built_in">mv</span> .htaccess .htaccess_old </code></div>
</div>
<p data-start="3767" data-end="3830">Regenerate from WordPress admin &rarr; Settings &rarr; Permalinks &rarr; Save.</p>
<hr data-start="3832" data-end="3835" />
<h3 data-start="3837" data-end="3880"><strong data-start="3841" data-end="3880">Step 8 &mdash; Check Apache Cache Modules</strong></h3>
<p data-start="3881" data-end="3925">Verify if mod_cache is enabled accidentally:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!">apachectl -M | <span class="hljs-keyword">grep</span> cache </code></div>
</div>
<p data-start="3962" data-end="3969">If yes:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!"><span class="hljs-attribute">a2dismod</span> cache a2dismod cache_disk systemctl restart httpd </code></div>
</div>
<hr data-start="4039" data-end="4042" />
<h3 data-start="4044" data-end="4095"><strong data-start="4048" data-end="4095">Step 9 &mdash; Regenerate New Image via WordPress</strong></h3>
<p data-start="4096" data-end="4113">Ask developer to:</p>
<ul data-start="4114" data-end="4241">
<li data-start="4114" data-end="4153">
<p data-start="4116" data-end="4153">Delete old image from Media Library</p>
</li>
<li data-start="4154" data-end="4189">
<p data-start="4156" data-end="4189">Upload new image with SAME name</p>
</li>
<li data-start="4190" data-end="4210">
<p data-start="4192" data-end="4210">Clear all caches</p>
</li>
<li data-start="4211" data-end="4241">
<p data-start="4213" data-end="4241">Validate on incognito mode</p>
</li>
</ul>
<p data-start="4243" data-end="4278">This usually resolves the mismatch.</p>
<hr data-start="4280" data-end="4283" />
<h2 data-start="4285" data-end="4339"><strong data-start="4288" data-end="4339">4. Resolution in This Case (jaipuriamba.edu.in)</strong></h2>
<p data-start="4341" data-end="4389">The Application Team performed corrective steps:</p>
<p data-start="4391" data-end="4529">✔ Replaced the image<br data-start="4411" data-end="4414" /> ✔ Updated WordPress media reference<br data-start="4449" data-end="4452" /> ✔ Cleared WordPress cache from backend<br data-start="4490" data-end="4493" /> ✔ Refreshed uploads folder content</p>
<p data-start="4531" data-end="4552">As a result, the URL:</p>
<p data-start="4554" data-end="4629"><code data-start="4554" data-end="4629">https://jaipuriamba.edu.in/wp-content/uploads/2024/12/jaipuriamba-hdr.jpg</code></p>
<p data-start="4631" data-end="4669"><strong data-start="4631" data-end="4669">started showing the updated image.</strong></p>
<hr data-start="4671" data-end="4674" />
<h2 data-start="4676" data-end="4705"><strong data-start="4679" data-end="4705">5. Preventive Measures</strong></h2>
<h3 data-start="4707" data-end="4752"><strong data-start="4711" data-end="4750">✔ Delete caching plugins completely</strong></h3>
<p data-start="4753" data-end="4807">Remove files, delete folders, clean cache directories.</p>
<h3 data-start="4809" data-end="4850"><strong data-start="4813" data-end="4848">✔ Avoid long-term cache headers</strong></h3>
<p data-start="4851" data-end="4870">Add to <code data-start="4858" data-end="4869">.htaccess</code>:</p>
<div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary">
<div class="sticky top-9">&nbsp;</div>
<div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!">&lt;FilesMatch "\.(jpg|jpeg|png|gif|webp)$"&gt; <span class="hljs-keyword">Header</span> <span class="hljs-keyword">set</span> <span class="hljs-keyword">Cache</span>-Control "max-age=<span class="hljs-number">86400</span>, <span class="hljs-built_in">public</span>" &lt;/FilesMatch&gt; </code></div>
</div>
<h3 data-start="4990" data-end="5040"><strong data-start="4994" data-end="5038">✔ Ensure only one caching system is used</strong></h3>
<p data-start="5041" data-end="5052">Do not mix:</p>
<ul data-start="5053" data-end="5147">
<li data-start="5053" data-end="5071">
<p data-start="5055" data-end="5071">WP Super Cache</p>
</li>
<li data-start="5072" data-end="5091">
<p data-start="5074" data-end="5091">LiteSpeed Cache</p>
</li>
<li data-start="5092" data-end="5106">
<p data-start="5094" data-end="5106">Cloudflare</p>
</li>
<li data-start="5107" data-end="5122">
<p data-start="5109" data-end="5122">WP-Optimize</p>
</li>
<li data-start="5123" data-end="5133">
<p data-start="5125" data-end="5133">Breeze</p>
</li>
<li data-start="5134" data-end="5147">
<p data-start="5136" data-end="5147">Nitropack</p>
</li>
</ul>
<h3 data-start="5149" data-end="5185"><strong data-start="5153" data-end="5183">✔ Use proper update method</strong></h3>
<p data-start="5186" data-end="5237">Update images only through WordPress Media Library.</p>
<h3 data-start="5239" data-end="5294"><strong data-start="5243" data-end="5292">✔ Disable server-side caching if using Apache</strong></h3>
<p data-start="5295" data-end="5339">Ensure LiteSpeed is NOT installed or active.</p>
<hr data-start="5341" data-end="5344" />
<h2 data-start="5346" data-end="5372"><strong data-start="5349" data-end="5372">6. Document Version</strong></h2>
<p data-start="5374" data-end="5564"><strong data-start="5374" data-end="5384">Title:</strong> Troubleshooting Image Cache Issues on WordPress (Apache/cPanel)<br data-start="5448" data-end="5451" /> <strong data-start="5451" data-end="5463">Version:</strong> 1.0<br data-start="5467" data-end="5470" /> <strong data-start="5470" data-end="5479">Date:</strong> 26-Nov-2025<br data-start="5491" data-end="5494" /> <strong data-start="5494" data-end="5511">Prepared for:</strong> jaipuriamba.edu.in<br data-start="5530" data-end="5533" /> <strong data-start="5533" data-end="5549">Prepared by:</strong> Support Team</p>
<!-- Comments are visible in the HTML source only -->
