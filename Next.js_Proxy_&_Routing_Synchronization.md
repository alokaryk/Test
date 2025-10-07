<h2>Action Plan Documentation: Next.js Proxy &amp; Routing Synchronization</h2>
<p>&nbsp;</p>
<p>This document outlines the Root Cause Analysis (RCA) and resolution steps taken to fix the routing, asset loading (CSS/JS 404s), and redirection failures when deploying the Next.js application under the <code>/CRM/</code> subdirectory behind the cPanel Apache proxy.</p>
<hr />
<p>&nbsp;</p>
<h3>RCA: Root Cause Analysis</h3>
<p>&nbsp;</p>
<p>The intermittent failures (stuck UI, incomplete redirect, missing CSS) were caused by a mismatch in routing context between the front-end application and the server's proxy:</p>
<ol start="1">
<li>
<p><strong>Failure to Serve Static Assets (404 Not Found):</strong> The Next.js application was running under the subdirectory <code>/CRM/</code> on port <span class="math-inline"><span class="katex"><span class="katex-html"><span class="base"><span class="mord">3000</span></span></span></span></span>. However, without explicit configuration, the application built its asset paths starting from the root (e.g., <code>/_next/static/css/...</code>). When the user accessed the site via <code>https://crm.rashmimetaliks.com/CRM/</code>, the browser tried to load assets from the wrong location, resulting in HTTP <strong>404 Not Found</strong> errors for all CSS and JavaScript files.</p>
</li>
<li>
<p><strong>Failed Redirection and Stuck UI:</strong> When processing the successful login, the Node.js application performed a redirect. Because the Apache proxy was not passing essential HTTPS headers (like <code>X-Forwarded-Proto</code>), the Node.js server often generated an incorrect absolute internal redirect URL, causing the browser to loop or fail the navigation, resulting in the stuck UI.</p>
</li>
</ol>
<hr />
<p>&nbsp;</p>
<h3>Resolution Phase 1: Application (Next.js) Code Fix</h3>
<p>&nbsp;</p>
<p>The application must be configured to correctly prefix all internal links, routes, and static asset references with the <code>/CRM</code> subdirectory.</p>
<div class="horizontal-scroll-wrapper">
<div class="table-block-component">
<div class="table-block has-export-button has-scrollbar">
<div class="table-content not-end-of-paragraph" data-hveid="0" data-ved="0CAAQ3ecQahgKEwjm45fokpKQAxUAAAAAHQAAAAAQtgY">
<table>
<thead>
<tr>
<td>Step</td>
<td>Action</td>
<td>Command/File</td>
<td>Purpose</td>
</tr>
</thead>
<tbody>
<tr>
<td><strong>1. Set <code>basePath</code></strong></td>
<td>Edited the main configuration file to set the base path.</td>
<td><code>/home/crmrashmimetalik/CRM/next.config.ts</code></td>
<td><strong>CRITICAL:</strong> Injects <code>/CRM</code> into all internal URLs and asset paths during the build.</td>
</tr>
<tr>
<td><strong>2. Define Environment Variable</strong></td>
<td>Ensured the variable required by <code>next.config.ts</code> is explicitly set during the build.</td>
<td>`basePath: process.env.NEXT_PUBLIC_BASE_PATH</td>
<td>&nbsp;</td>
</tr>
<tr>
<td><strong>3. Rebuild Application</strong></td>
<td>Ran the build command with the environment variable.</td>
<td><code>NEXT_PUBLIC_BASE_PATH='/CRM' npm run build</code></td>
<td>Generates the new, correctly prefixed static assets (<code>/CRM/_next/static/...</code>).</td>
</tr>
</tbody>
</table>
</div>
<div class="table-footer hide-from-message-actions ng-star-inserted"><span class="mdc-button__label"><span class="export-sheets-button">&nbsp;</span></span></div>
</div>
</div>
</div>
<hr />
<p>&nbsp;</p>
<h3>Resolution Phase 2: Server (Apache/cPanel) Proxy Header Fix</h3>
<p>&nbsp;</p>
<p>The custom proxy file was deployed to the cPanel include directory to ensure Apache proxies the requests correctly and passes the necessary HTTPS context.</p>
<div class="horizontal-scroll-wrapper">
<div class="table-block-component">
<div class="table-block has-export-button has-scrollbar">
<div class="table-content not-end-of-paragraph" data-hveid="0" data-ved="0CAAQ3ecQahgKEwjm45fokpKQAxUAAAAAHQAAAAAQuAY">
<table>
<thead>
<tr>
<td>Step</td>
<td>Action</td>
<td>Configuration File</td>
<td>Purpose</td>
</tr>
</thead>
<tbody>
<tr>
<td><strong>1. Create Config File</strong></td>
<td>Created the custom configuration file.</td>
<td><code>/etc/apache2/conf.d/userdata/ssl/2_4/crmrashmimetalik/crm.rashmimetaliks.com/nodeapp_proxy.conf</code></td>
<td>Provides a clean, custom include for the domain's VHost.</td>
</tr>
<tr>
<td><strong>2. Configure Proxy Pass</strong></td>
<td>Added the primary proxy rule using the confirmed port and path.</td>
<td><code>ProxyPass /CRM/ http://127.0.0.1:3000/ disablereuse=on nocanon</code></td>
<td>Forwards all <code>/CRM/</code> traffic to the Node.js port.</td>
</tr>
<tr>
<td><strong>3. Forward Headers</strong></td>
<td>Added critical header directives.</td>
<td><code>RequestHeader set X-Forwarded-Proto "https"</code> <code>RequestHeader set X-Forwarded-SSL "on"</code></td>
<td><strong>CRITICAL FIX:</strong> Tells the Node.js app that the user connected via <strong>HTTPS</strong>, resolving failed redirect loops.</td>
</tr>
<tr>
<td><strong>4. Configure Static Assets</strong></td>
<td>Added a dedicated pass for static files (to resolve lingering <span class="math-inline"><span class="katex"><span class="katex-html"><span class="base"><span class="mord">404</span></span></span></span></span> issues).</td>
<td><code>ProxyPass /CRM/_next/static/ http://127.0.0.1:3000/_next/static/ disablereuse=on nocanon</code></td>
<td>Ensures the Next.js server can locate and serve its static assets correctly under the <code>/CRM/</code> context.</td>
</tr>
</tbody>
</table>
</div>
<div class="table-footer hide-from-message-actions ng-star-inserted"><span class="mdc-button__label"><span class="export-sheets-button">&nbsp;</span></span></div>
</div>
</div>
</div>
<hr />
<p>&nbsp;</p>
<h3>Final Service Deployment and Verification</h3>
<p>&nbsp;</p>
<ol start="1">
<li>
<p><strong>Integrate Configuration:</strong> Run the cPanel script to automatically activate the custom configuration file.</p>
<div class="code-block ng-tns-c1485810703-1308 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjm45fokpKQAxUAAAAAHQAAAAAQugY">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-1308 ng-star-inserted"><span class="ng-tns-c1485810703-1308">Bash</span>
<div class="buttons ng-tns-c1485810703-1308 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-1308">
<div class="animated-opacity ng-tns-c1485810703-1308">
<pre class="ng-tns-c1485810703-1308"><code class="code-container formatted ng-tns-c1485810703-1308" data-test-id="code-content">/scripts/rebuildhttpdconf
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p><strong>Restart Web Server:</strong> Apply the new proxy rules.</p>
<div class="code-block ng-tns-c1485810703-1309 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjm45fokpKQAxUAAAAAHQAAAAAQuwY">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-1309 ng-star-inserted"><span class="ng-tns-c1485810703-1309">Bash</span>
<div class="buttons ng-tns-c1485810703-1309 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-1309">
<div class="animated-opacity ng-tns-c1485810703-1309">
<pre class="ng-tns-c1485810703-1309"><code class="code-container formatted ng-tns-c1485810703-1309" data-test-id="code-content">/scripts/restartsrv httpd
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p><strong>Restart Application:</strong> Load the newly built code (with the <code>basePath</code> fix) and the correct environment variables.</p>
<div class="code-block ng-tns-c1485810703-1310 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjm45fokpKQAxUAAAAAHQAAAAAQvAY">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-1310 ng-star-inserted"><span class="ng-tns-c1485810703-1310">Bash</span>
<div class="buttons ng-tns-c1485810703-1310 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-1310">
<div class="animated-opacity ng-tns-c1485810703-1310">
<pre class="ng-tns-c1485810703-1310"><code class="code-container formatted ng-tns-c1485810703-1310" data-test-id="code-content">NEXT_PUBLIC_BASE_PATH=<span class="hljs-string">'/CRM'</span> pm2 restart all
</code></pre>
</div>
</div>
</div>
</li>
</ol>
<p><strong><em>Note:</em></strong> <em>Following these steps resolved the proxy and routing issues. The subsequent <strong>HTTP 500 error</strong> was isolated to a <strong>PostgreSQL Authentication Failure</strong> caused by incorrect database credentials in the application's runtime environment (<code>.env</code> file), which must be resolved by verifying and resetting the database password.</em></p>
