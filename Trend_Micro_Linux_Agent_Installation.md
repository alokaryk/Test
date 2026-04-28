<p class="demoTitle">This Standard Operating Procedure (SOP) outlines the installation of the <strong data-path-to-node="0" data-index-in-node="73">Trend Micro Endpoint Basecamp (XBC)</strong> agent on various Linux distributions. This agent is the foundation for Trend Micro Vision One and Server &amp; Workload Protection.</p>
<h2 data-path-to-node="2"><strong data-path-to-node="2" data-index-in-node="0">Trend Micro Linux Agent Installation</strong></h2>
<p data-path-to-node="3"><strong data-path-to-node="3" data-index-in-node="0">Version:</strong> 1.1 | <strong data-path-to-node="3" data-index-in-node="15">Target:</strong> Linux x86_64 | <strong data-path-to-node="3" data-index-in-node="38">Agent Type:</strong> Endpoint Basecamp</p>
<h3 data-path-to-node="4"><strong data-path-to-node="4" data-index-in-node="0">1. Prerequisites</strong></h3>
<p data-path-to-node="5">Before beginning, ensure the following requirements are met:</p>
<ul data-path-to-node="6">
<li>
<p data-path-to-node="6,0,0"><strong data-path-to-node="6,0,0" data-index-in-node="0">Root Access:</strong> You must have <code data-path-to-node="6,0,0" data-index-in-node="27">root</code> or <code data-path-to-node="6,0,0" data-index-in-node="35">sudo</code> privileges.</p>
</li>
<li>
<p data-path-to-node="6,1,0"><strong data-path-to-node="6,1,0" data-index-in-node="0">Internet Connectivity:</strong> The server must reach Trend Micro backends (Port 443).</p>
</li>
<li>
<p data-path-to-node="6,2,0"><strong data-path-to-node="6,2,0" data-index-in-node="0">Time Sync:</strong> NTP must be active and time must be accurate.</p>
</li>
<li>
<p data-path-to-node="6,3,0"><strong data-path-to-node="6,3,0" data-index-in-node="0">Supported OS:</strong> * <strong data-path-to-node="6,3,0" data-index-in-node="16">RHEL-based:</strong> AlmaLinux 8/9, CentOS 7, Rocky Linux.</p>
<ul data-path-to-node="6,3,1">
<li>
<p data-path-to-node="6,3,1,0,0"><strong data-path-to-node="6,3,1,0,0" data-index-in-node="0">Debian-based:</strong> Ubuntu 20.04/22.04/24.04, Debian 11/12.</p>
</li>
<li>
<p data-path-to-node="6,3,1,1,0"><strong data-path-to-node="6,3,1,1,0" data-index-in-node="0">Others:</strong> Amazon Linux 2/2023, Oracle Linux.</p>
</li>
</ul>
</li>
</ul>
<hr data-path-to-node="7" />
<h3 data-path-to-node="8"><strong data-path-to-node="8" data-index-in-node="0">2. Preparation</strong></h3>
<ol start="1" data-path-to-node="9">
<li>
<p data-path-to-node="9,0,0"><strong data-path-to-node="9,0,0" data-index-in-node="0">Locate the Installer:</strong> Ensure <code data-path-to-node="9,0,0" data-index-in-node="29">TMServerAgent_Linux_x86_64_Server_Protection.tar</code> is in a dedicated directory (e.g., <code data-path-to-node="9,0,0" data-index-in-node="113">/home/admin1/</code>).</p>
</li>
<li>
<p data-path-to-node="9,1,0"><strong data-path-to-node="9,1,0" data-index-in-node="0">Extract the Archive:</strong></p>
<div class="code-block ng-tns-c541353407-463 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwili57E34-UAxUAAAAAHQAAAAAQ8gU">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c541353407-463 ng-star-inserted"><span class="ng-tns-c541353407-463">Bash</span>
<div class="buttons ng-tns-c541353407-463 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c541353407-463">
<div class="animated-opacity ng-tns-c541353407-463">
<pre class="ng-tns-c541353407-463"><code class="code-container formatted ng-tns-c541353407-463" data-test-id="code-content">tar -xvf TMServerAgent_Linux_x86_64_Server_Protection.tar
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p data-path-to-node="9,2,0"><strong data-path-to-node="9,2,0" data-index-in-node="0">Identify Key Files:</strong></p>
<ul data-path-to-node="9,2,1">
<li>
<p data-path-to-node="9,2,1,0,0"><code data-path-to-node="9,2,1,0,0" data-index-in-node="0">tmxbc</code>: The main installer binary.</p>
</li>
<li>
<p data-path-to-node="9,2,1,1,0"><code data-path-to-node="9,2,1,1,0" data-index-in-node="0">config.json</code>: Deployment configuration.</p>
</li>
<li>
<p data-path-to-node="9,2,1,2,0"><code data-path-to-node="9,2,1,2,0" data-index-in-node="0">.property</code>: <strong data-path-to-node="9,2,1,2,0" data-index-in-node="11">(Hidden)</strong> Contains tenant/identity metadata.</p>
</li>
</ul>
</li>
</ol>
<hr data-path-to-node="10" />
<h3 data-path-to-node="11"><strong data-path-to-node="11" data-index-in-node="0">3. Pre-Installation Check</strong></h3>
<p data-path-to-node="12">To ensure the environment is ready and won't fail during the process, run the early check tool:</p>
<div class="code-block ng-tns-c541353407-464 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwili57E34-UAxUAAAAAHQAAAAAQ8wU">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c541353407-464 ng-star-inserted"><span class="ng-tns-c541353407-464">Bash</span>
<div class="buttons ng-tns-c541353407-464 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c541353407-464">
<div class="animated-opacity ng-tns-c541353407-464">
<pre class="ng-tns-c541353407-464"><code class="code-container formatted ng-tns-c541353407-464" data-test-id="code-content">./tmxbc xbc-early-check
</code></pre>
</div>
</div>
</div>
<blockquote data-path-to-node="14">
<p data-path-to-node="14,0"><strong data-path-to-node="14,0" data-index-in-node="0">Note:</strong> All critical tests (Connectivity, Disk Space, OS Support) must show a green <strong data-path-to-node="14,0" data-index-in-node="82">✅ PASSED</strong>.</p>
</blockquote>
<hr data-path-to-node="15" />
<h3 data-path-to-node="16"><strong data-path-to-node="16" data-index-in-node="0">4. Installation Steps</strong></h3>
<h4 data-path-to-node="17"><strong data-path-to-node="17" data-index-in-node="0">Step 4.1: Execute the Installer</strong></h4>
<p data-path-to-node="18">Run the installation command. The binary automatically detects your Linux flavor (AlmaLinux, Ubuntu, etc.) and chooses the correct internal package.</p>
<div class="code-block ng-tns-c541353407-465 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwili57E34-UAxUAAAAAHQAAAAAQ9AU">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c541353407-465 ng-star-inserted"><span class="ng-tns-c541353407-465">Bash</span>
<div class="buttons ng-tns-c541353407-465 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c541353407-465">
<div class="animated-opacity ng-tns-c541353407-465">
<pre class="ng-tns-c541353407-465"><code class="code-container formatted ng-tns-c541353407-465" data-test-id="code-content">./tmxbc install
</code></pre>
</div>
</div>
</div>
<h4 data-path-to-node="20"><strong data-path-to-node="20" data-index-in-node="0">Step 4.2: Verify Identity Files</strong></h4>
<p data-path-to-node="21">In some environments, the hidden identity file <code data-path-to-node="21" data-index-in-node="47">.property</code> must be manually placed in the installation path to ensure the agent reports to the correct console.</p>
<div class="code-block ng-tns-c541353407-466 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwili57E34-UAxUAAAAAHQAAAAAQ9QU">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c541353407-466 ng-star-inserted"><span class="ng-tns-c541353407-466">Bash</span>
<div class="buttons ng-tns-c541353407-466 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c541353407-466">
<div class="animated-opacity ng-tns-c541353407-466">
<pre class="ng-tns-c541353407-466"><code class="code-container formatted ng-tns-c541353407-466" data-test-id="code-content">cp .property /opt/TrendMicro/EndpointBasecamp/bin/
chmod 600 /opt/TrendMicro/EndpointBasecamp/bin/.property
</code></pre>
</div>
</div>
</div>
<h4 data-path-to-node="23"><strong data-path-to-node="23" data-index-in-node="0">Step 4.3: Start and Enable Service</strong></h4>
<p data-path-to-node="24">The installer usually handles this, but manual verification is recommended:</p>
<div class="code-block ng-tns-c541353407-467 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwili57E34-UAxUAAAAAHQAAAAAQ9gU">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c541353407-467 ng-star-inserted"><span class="ng-tns-c541353407-467">Bash</span>
<div class="buttons ng-tns-c541353407-467 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c541353407-467">
<div class="animated-opacity ng-tns-c541353407-467">
<pre class="ng-tns-c541353407-467"><code class="code-container formatted ng-tns-c541353407-467" data-test-id="code-content"><span class="hljs-comment"># Reload systemd to recognize the new service</span>
systemctl daemon-reload

<span class="hljs-comment"># Enable and start the service</span>
systemctl <span class="hljs-built_in">enable</span> tmxbc
systemctl restart tmxbc
</code></pre>
</div>
</div>
</div>
<hr data-path-to-node="26" />
<h3 data-path-to-node="27"><strong data-path-to-node="27" data-index-in-node="0">5. Post-Installation Verification</strong></h3>
<table data-path-to-node="28">
<thead>
<tr>
<td><strong>Check Item</strong></td>
<td><strong>Command</strong></td>
<td><strong>Expected Result</strong></td>
</tr>
</thead>
<tbody>
<tr>
<td><span data-path-to-node="28,1,0,0"><strong data-path-to-node="28,1,0,0" data-index-in-node="0">Process Status</strong></span></td>
<td><span data-path-to-node="28,1,1,0"><code data-path-to-node="28,1,1,0" data-index-in-node="0">ps aux | grep tmxbc</code></span></td>
<td><span data-path-to-node="28,1,2,0">Service path: <code data-path-to-node="28,1,2,0" data-index-in-node="14">/opt/TrendMicro/.../tmxbc service run</code></span></td>
</tr>
<tr>
<td><span data-path-to-node="28,2,0,0"><strong data-path-to-node="28,2,0,0" data-index-in-node="0">Service Status</strong></span></td>
<td><span data-path-to-node="28,2,1,0"><code data-path-to-node="28,2,1,0" data-index-in-node="0">systemctl status tmxbc</code></span></td>
<td><span data-path-to-node="28,2,2,0"><code data-path-to-node="28,2,2,0" data-index-in-node="0">Active: active (running)</code></span></td>
</tr>
<tr>
<td><span data-path-to-node="28,3,0,0"><strong data-path-to-node="28,3,0,0" data-index-in-node="0">Module Download</strong></span></td>
<td><span data-path-to-node="28,3,1,0"><code data-path-to-node="28,3,1,0" data-index-in-node="0">ls -l /opt/TrendMicro/</code></span></td>
<td><span data-path-to-node="28,3,2,0">New folders like <code data-path-to-node="28,3,2,0" data-index-in-node="17">CloudOne</code> or <code data-path-to-node="28,3,2,0" data-index-in-node="29">tmagent</code> appear.</span></td>
</tr>
<tr>
<td><span data-path-to-node="28,4,0,0"><strong data-path-to-node="28,4,0,0" data-index-in-node="0">Cloud Connectivity</strong></span></td>
<td><span data-path-to-node="28,4,1,0"><code data-path-to-node="28,4,1,0" data-index-in-node="0">tail -f /opt/TrendMicro/EndpointBasecamp/log/tmxbc.log</code></span></td>
<td><span data-path-to-node="28,4,2,0">No "Connection Refused" or "Auth Failed" errors.</span></td>
</tr>
</tbody>
</table>
<hr data-path-to-node="29" />
<h3 data-path-to-node="30"><strong data-path-to-node="30" data-index-in-node="0">6. Troubleshooting</strong></h3>
<ul data-path-to-node="31">
<li>
<p data-path-to-node="31,0,0"><strong data-path-to-node="31,0,0" data-index-in-node="0"><span class="citation-1">Error: </span><code data-path-to-node="31,0,0" data-index-in-node="7"><span class="citation-1">Failed to create Linux group tm_xes</span></code></strong><code data-path-to-node="31,0,0" data-index-in-node="7"></code></p>
<ul data-path-to-node="31,0,1">
<li>
<p data-path-to-node="31,0,1,0,0"><strong data-path-to-node="31,0,1,0,0" data-index-in-node="0">Cause:</strong> Manual group creation is restricted.</p>
</li>
<li>
<p data-path-to-node="31,0,1,1,0"><strong data-path-to-node="31,0,1,1,0" data-index-in-node="0">Fix:</strong> Manually run <code data-path-to-node="31,0,1,1,0" data-index-in-node="18">groupadd -f tm_xes</code> as root before installing.</p>
</li>
</ul>
</li>
<li>
<p data-path-to-node="31,1,0"><strong data-path-to-node="31,1,0" data-index-in-node="0">Error: <code data-path-to-node="31,1,0" data-index-in-node="7">Unit tmxbc.service not found</code></strong></p>
<ul data-path-to-node="31,1,1">
<li>
<p data-path-to-node="31,1,1,0,0"><strong data-path-to-node="31,1,1,0,0" data-index-in-node="0">Fix:</strong> Ensure you are in the <code data-path-to-node="31,1,1,0,0" data-index-in-node="27">/opt/TrendMicro/EndpointBasecamp/bin</code> directory and run <code data-path-to-node="31,1,1,0,0" data-index-in-node="82">./tmxbc install</code> again, then <code data-path-to-node="31,1,1,0,0" data-index-in-node="110">systemctl daemon-reload</code>.</p>
</li>
</ul>
</li>
<li>
<p data-path-to-node="31,2,0"><strong data-path-to-node="31,2,0" data-index-in-node="0">Missing Dependencies (Older OS):</strong></p>
<ul data-path-to-node="31,2,1">
<li>
<p data-path-to-node="31,2,1,0,0"><strong data-path-to-node="31,2,1,0,0" data-index-in-node="0">Fix:</strong> Install <code data-path-to-node="31,2,1,0,0" data-index-in-node="13">libcap</code> or <code data-path-to-node="31,2,1,0,0" data-index-in-node="23">glibc.i686</code> if prompted (common on minimal installs).</p>
</li>
</ul>
</li>
</ul>
<hr data-path-to-node="32" />
<h3 data-path-to-node="33"><strong data-path-to-node="33" data-index-in-node="0">7. Uninstallation (If Required)</strong></h3>
<p data-path-to-node="34">If the agent needs to be removed or re-installed:</p>
<div class="code-block ng-tns-c541353407-468 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwili57E34-UAxUAAAAAHQAAAAAQ-gU">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c541353407-468 ng-star-inserted"><span class="ng-tns-c541353407-468">Bash</span>
<div class="buttons ng-tns-c541353407-468 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c541353407-468">
<div class="animated-opacity ng-tns-c541353407-468">
<pre class="ng-tns-c541353407-468"><code class="code-container formatted ng-tns-c541353407-468" data-test-id="code-content">./tmxbc uninstall
rm -rf /opt/TrendMicro/</code></pre>
</div>
</div>
</div>
<!-- Comments are visible in the HTML source only -->
