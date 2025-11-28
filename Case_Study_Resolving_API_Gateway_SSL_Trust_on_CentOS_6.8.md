<h2>📚 Case Study: Resolving API Gateway SSL Trust on CentOS 6.8</h2>
<p>&nbsp;</p>
<p data-path-to-node="4">&nbsp;</p>
<h3>🎯 Objective</h3>
<p>&nbsp;</p>
<p data-path-to-node="5">To establish a trusted HTTPS connection from a legacy <strong>CentOS release 6.8 (Final)</strong> server to an external API Gateway (<code>sandesh.indianoil.co.in</code>) secured by a certificate chain issued by the <strong>GeoTrust/DigiCert CA</strong>.</p>
<p data-path-to-node="6">&nbsp;</p>
<h3>🔍 Symptoms Encountered</h3>
<p>&nbsp;</p>
<table data-path-to-node="7">
<thead>
<tr>
<td><strong>Step</strong></td>
<td><strong>Initial Symptom/Error</strong></td>
<td><strong>Cause/Diagnosis</strong></td>
<td><strong>Resolution (Action Taken)</strong></td>
</tr>
</thead>
<tbody>
<tr>
<td><span data-path-to-node="7,1,0,0"><strong>Initial Connection</strong></span></td>
<td><span data-path-to-node="7,1,1,0"><code>SSL certificate problem: unable to get local issuer certificate</code></span></td>
<td><span data-path-to-node="7,1,2,0">Server did not trust the <strong>Intermediate CA</strong> (<code>GeoTrust TLS RSA CA G1</code>) that signed the API server's certificate.</span></td>
<td><span data-path-to-node="7,1,3,0">Required installing the Intermediate CA certificate file.</span></td>
</tr>
<tr>
<td><span data-path-to-node="7,2,0,0"><strong>Download Attempt 1</strong></span></td>
<td><span data-path-to-node="7,2,1,0"><code>wget: unable to resolve host address "downloads.digicert.com"</code></span></td>
<td><span data-path-to-node="7,2,2,0"><strong>DNS Resolution Failure:</strong> Specific DNS record failed to resolve, though other sites (e.g., <code>google.com</code>) worked.</span></td>
<td><span data-path-to-node="7,2,3,0">Bypassed DNS using the IP address and the <code>--header="Host:..."</code> flag.</span></td>
</tr>
<tr>
<td><span data-path-to-node="7,3,0,0"><strong>Download Attempt 2</strong></span></td>
<td><span data-path-to-node="7,3,1,0"><code>ERROR: cannot verify cacerts.digicert.com&rsquo;s certificate</code></span></td>
<td><span data-path-to-node="7,3,2,0"><strong>Trust Chain Loop:</strong> The server did not trust the certificate of the download server itself.</span></td>
<td><span data-path-to-node="7,3,3,0">Used the <code>--no-check-certificate</code> flag to force the download.</span></td>
</tr>
<tr>
<td><span data-path-to-node="7,4,0,0"><strong>Download Attempt 3</strong></span></td>
<td><span data-path-to-node="7,4,1,0"><code>404 Not Found</code></span></td>
<td><span data-path-to-node="7,4,2,0"><strong>Incorrect URL Path:</strong> Attempting to use a working IP with an incorrect file path.</span></td>
<td><span data-path-to-node="7,4,3,0">Used a successful download path for a different DigiCert file, which provided a <strong>Root CA</strong> (<code>DigiCertAssuredIDRootCA.crt.pem</code>).</span></td>
</tr>
<tr>
<td><span data-path-to-node="7,5,0,0"><strong>Final Resolution</strong></span></td>
<td><span data-path-to-node="7,5,1,0">N/A</span></td>
<td><span data-path-to-node="7,5,2,0">The downloaded <strong>Root CA</strong> was sufficient to complete the trust chain for the API Gateway.</span></td>
<td><span data-path-to-node="7,5,3,0">Copied the file to <code>/etc/pki/ca-trust/source/anchors/</code> and ran <code>update-ca-trust extract</code>.</span></td>
</tr>
<tr>
<td><span data-path-to-node="7,6,0,0"><strong>Post-Resolution</strong></span></td>
<td><span data-path-to-node="7,6,1,0"><code>HTTP/1.1 405 Method Not Allowed</code></span></td>
<td><span data-path-to-node="7,6,2,0">API requires a <strong>POST</strong> request, but <code>curl</code> defaulted to <code>GET</code>.</span></td>
<td><span data-path-to-node="7,6,3,0">Switched to <code>curl -X POST</code> with the correct JSON payload (future step).</span></td>
</tr>
</tbody>
</table>
<hr data-path-to-node="8" />
<p data-path-to-node="9">&nbsp;</p>
<h2>💡 Future Resolution Flowchart</h2>
<p>&nbsp;</p>
<p data-path-to-node="10">Use this guide whenever you encounter an <strong>"unable to get local issuer certificate"</strong> or <strong>"certificate verify failed"</strong> error on a CentOS/RHEL 6 server.</p>
<p data-path-to-node="11">&nbsp;</p>
<h3>Phase 1: Identify and Obtain the Missing CA</h3>
<p>&nbsp;</p>
<ol start="1" data-path-to-node="12">
<li>
<p data-path-to-node="12,0,0"><strong>Identify the Issuer:</strong> Run <code>curl -v https://your-api-url.com/</code>. Look for the line starting with <code>issuer: CN=...</code>. This is the name of the missing <strong>Intermediate CA</strong>.</p>
</li>
<li>
<p data-path-to-node="12,1,0"><strong>Locate the CA File:</strong> Search for the issuer name (e.g., "GeoTrust TLS RSA CA G1") on the CA's website (e.g., DigiCert, Sectigo). Download the certificate file in <strong>PEM format</strong> (usually <code>.crt</code> or <code>.pem</code>).</p>
</li>
</ol>
<p data-path-to-node="13">&nbsp;</p>
<h3>Phase 2: System Download (Attempt 1)</h3>
<p>&nbsp;</p>
<ol start="1" data-path-to-node="14">
<li>
<p data-path-to-node="14,0,0"><strong>Attempt Direct Download:</strong> Try to download the file directly to the anchors directory:</p>
<div class="code-block ng-tns-c4228668850-235 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjZvfbx6ZSRAxUAAAAAHQAAAAAQvQM">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c4228668850-235 ng-star-inserted"><span class="ng-tns-c4228668850-235">Bash</span>
<div class="buttons ng-tns-c4228668850-235 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c4228668850-235">
<div class="animated-opacity ng-tns-c4228668850-235">
<pre class="ng-tns-c4228668850-235"><code class="code-container formatted ng-tns-c4228668850-235" data-test-id="code-content"><span class="hljs-built_in">cd</span> /etc/pki/ca-trust/<span class="hljs-built_in">source</span>/anchors/
wget https://ca-download-link/file.pem
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p data-path-to-node="14,1,0"><strong>If Download Fails (DNS Error):</strong></p>
<ul data-path-to-node="14,1,1">
<li>
<p data-path-to-node="14,1,1,0,0">Find the host's IP address (e.g., using Google DNS tools).</p>
</li>
<li>
<p data-path-to-node="14,1,1,1,0">Use the IP address and bypass hostname checks:</p>
<div class="code-block ng-tns-c4228668850-236 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjZvfbx6ZSRAxUAAAAAHQAAAAAQvgM">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c4228668850-236 ng-star-inserted"><span class="ng-tns-c4228668850-236">Bash</span>
<div class="buttons ng-tns-c4228668850-236 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c4228668850-236">
<div class="animated-opacity ng-tns-c4228668850-236">
<pre class="ng-tns-c4228668850-236"><code class="code-container formatted ng-tns-c4228668850-236" data-test-id="code-content">wget -O file.pem --header=<span class="hljs-string">"Host: ca-domain.com"</span> https://IP_ADDRESS/path/file.pem
</code></pre>
</div>
</div>
</div>
</li>
</ul>
</li>
<li>
<p data-path-to-node="14,2,0"><strong>If Download Fails (SSL Error):</strong></p>
<ul data-path-to-node="14,2,1">
<li>
<p data-path-to-node="14,2,1,0,0">The server's bundle is too old to trust the download site.</p>
</li>
<li>
<p data-path-to-node="14,2,1,1,0">Use the <code>--no-check-certificate</code> flag combined with the IP address workaround:</p>
<div class="code-block ng-tns-c4228668850-237 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjZvfbx6ZSRAxUAAAAAHQAAAAAQvwM">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c4228668850-237 ng-star-inserted"><span class="ng-tns-c4228668850-237">Bash</span>
<div class="buttons ng-tns-c4228668850-237 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c4228668850-237">
<div class="animated-opacity ng-tns-c4228668850-237">
<pre class="ng-tns-c4228668850-237"><code class="code-container formatted ng-tns-c4228668850-237" data-test-id="code-content">wget --no-check-certificate -O file.pem --header=<span class="hljs-string">"Host: ca-domain.com"</span> https://IP_ADDRESS/path/file.pem
</code></pre>
</div>
</div>
</div>
</li>
</ul>
</li>
</ol>
<p data-path-to-node="15">&nbsp;</p>
<h3>Phase 3: System Download (Fallback Method)</h3>
<p>&nbsp;</p>
<p data-path-to-node="16">If all system download attempts fail, use a working machine (your desktop) to transfer the file.</p>
<ol start="1" data-path-to-node="17">
<li>
<p data-path-to-node="17,0,0"><strong>Desktop Download:</strong> Download the required <code>.pem</code> or <code>.crt</code> file directly on your desktop/laptop.</p>
</li>
<li>
<p data-path-to-node="17,1,0"><strong>Secure Copy (SCP):</strong> Transfer the file to the server's anchors directory:</p>
<div class="code-block ng-tns-c4228668850-238 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjZvfbx6ZSRAxUAAAAAHQAAAAAQwAM">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c4228668850-238 ng-star-inserted"><span class="ng-tns-c4228668850-238">Bash</span>
<div class="buttons ng-tns-c4228668850-238 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c4228668850-238">
<div class="animated-opacity ng-tns-c4228668850-238">
<pre class="ng-tns-c4228668850-238"><code class="code-container formatted ng-tns-c4228668850-238" data-test-id="code-content"><span class="hljs-comment"># Run this from your desktop</span>
scp /path/to/<span class="hljs-built_in">local</span>/file.pem root@your_server_ip:/etc/pki/ca-trust/<span class="hljs-built_in">source</span>/anchors/
</code></pre>
</div>
</div>
</div>
</li>
</ol>
<p data-path-to-node="18">&nbsp;</p>
<h3>Phase 4: Final Installation</h3>
<p>&nbsp;</p>
<ol start="1" data-path-to-node="19">
<li>
<p data-path-to-node="19,0,0"><strong>Verify File:</strong> Ensure the certificate file is in: <code>/etc/pki/ca-trust/source/anchors/</code>.</p>
</li>
<li>
<p data-path-to-node="19,1,0"><strong>Update Trust Store:</strong> This command integrates the new certificate into the system-wide bundle (<code>ca-bundle.crt</code>):</p>
<div class="code-block ng-tns-c4228668850-239 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjZvfbx6ZSRAxUAAAAAHQAAAAAQwQM">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c4228668850-239 ng-star-inserted"><span class="ng-tns-c4228668850-239">Bash</span>
<div class="buttons ng-tns-c4228668850-239 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c4228668850-239">
<div class="animated-opacity ng-tns-c4228668850-239">
<pre class="ng-tns-c4228668850-239"><code class="code-container formatted ng-tns-c4228668850-239" data-test-id="code-content">update-ca-trust extract
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p data-path-to-node="19,2,0"><strong>Test Connection:</strong> Verify the fix using <code>curl</code> (the <code>-v</code> flag is essential):</p>
<div class="code-block ng-tns-c4228668850-240 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjZvfbx6ZSRAxUAAAAAHQAAAAAQwgM">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c4228668850-240 ng-star-inserted"><span class="ng-tns-c4228668850-240">Bash</span>
<div class="buttons ng-tns-c4228668850-240 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c4228668850-240">
<div class="animated-opacity ng-tns-c4228668850-240">
<pre class="ng-tns-c4228668850-240"><code class="code-container formatted ng-tns-c4228668850-240" data-test-id="code-content">curl -v https://your-api-url.com/
</code></pre>
</div>
</div>
</div>
</li>
</ol>
<p data-path-to-node="20">&nbsp;</p>
<h3>Phase 5: Post-SSL Troubleshooting</h3>
<p>&nbsp;</p>
<p data-path-to-node="21">If the <code>curl</code> test still fails, but <strong>without a certificate error</strong>, the issue is now related to the API's requirements (e.g., HTTP method, payload, or authentication).</p>
<table data-path-to-node="22">
<thead>
<tr>
<td><strong>Status Code</strong></td>
<td><strong>Meaning</strong></td>
<td><strong>Action</strong></td>
</tr>
</thead>
<tbody>
<tr>
<td><span data-path-to-node="22,1,0,0"><strong>405 Method Not Allowed</strong></span></td>
<td><span data-path-to-node="22,1,1,0">You used the wrong HTTP verb (e.g., used <code>GET</code> instead of <code>POST</code>).</span></td>
<td><span data-path-to-node="22,1,2,0">Use <code>curl -X POST</code> or the required verb.</span></td>
</tr>
<tr>
<td><span data-path-to-node="22,2,0,0"><strong>400 Bad Request</strong></span></td>
<td><span data-path-to-node="22,2,1,0">The JSON/XML body is missing required fields or is improperly formatted.</span></td>
<td><span data-path-to-node="22,2,2,0">Consult API docs; correct payload data structure.</span></td>
</tr>
<tr>
<td><span data-path-to-node="22,3,0,0"><strong>401 Unauthorized</strong></span></td>
<td><span data-path-to-node="22,3,1,0">Missing or invalid authentication (Token, API Key, User/Pass).</span></td>
<td><span data-path-to-node="22,3,2,0">Add the correct authorization header (e.g., <code>-H "Authorization: Bearer..."</code>).</span></td>
</tr>
</tbody>
</table>
<!-- Comments are visible in the HTML source only -->
