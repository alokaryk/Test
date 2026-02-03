<p data-path-to-node="0">Since <code data-path-to-node="0" data-index-in-node="6">crsctl check has</code> shows that the High Availability Services (HAS) are <strong data-path-to-node="0" data-index-in-node="75">online</strong>, but <code data-path-to-node="0" data-index-in-node="87">srvctl</code> is failing with communication errors, it indicates that the underlying CSS (Cluster Synchronization Services) daemon is likely not running or is stuck.</p>
<p data-path-to-node="1">Because you are working on <code data-path-to-node="1" data-index-in-node="27">erpproddb1-dr</code> (a Disaster Recovery node), it is common for these services to be down or in a "waiting" state.</p>
<p data-path-to-node="2">Here is how to resolve this:</p>
<h3 data-path-to-node="3">1. Start the services as ROOT</h3>
<p data-path-to-node="4">The <code data-path-to-node="4" data-index-in-node="4">grid</code> user does not have permission to stop or start the high-level cluster stack (<code data-path-to-node="4" data-index-in-node="86">has</code>). You must use the <strong data-path-to-node="4" data-index-in-node="109">full path</strong> as root.</p>
<p data-path-to-node="5"><strong data-path-to-node="5" data-index-in-node="0">Run these as root:</strong></p>
<div class="code-block ng-tns-c986751816-109 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwi5nIDNorqSAxUAAAAAHQAAAAAQsgE">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c986751816-109 ng-star-inserted"><span class="ng-tns-c986751816-109">Bash</span>
<div class="buttons ng-tns-c986751816-109 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c986751816-109">
<div class="animated-opacity ng-tns-c986751816-109">
<pre class="ng-tns-c986751816-109"><code class="code-container formatted ng-tns-c986751816-109" data-test-id="code-content"><span class="hljs-comment"># Force a restart of the High Availability stack</span>
/oracle/app/grid/19.3.0/gridhome_1/bin/crsctl stop has -f
/oracle/app/grid/19.3.0/gridhome_1/bin/crsctl start has
</code></pre>
</div>
</div>
</div>
<h3 data-path-to-node="7">2. Monitor the stack startup</h3>
<p data-path-to-node="8">Wait about 2&ndash;3 minutes, then run this command as <strong data-path-to-node="8" data-index-in-node="49">root</strong> to see which components are actually coming online:</p>
<div class="code-block ng-tns-c986751816-110 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwi5nIDNorqSAxUAAAAAHQAAAAAQswE">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c986751816-110 ng-star-inserted"><span class="ng-tns-c986751816-110">Bash</span>
<div class="buttons ng-tns-c986751816-110 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c986751816-110">
<div class="animated-opacity ng-tns-c986751816-110">
<pre class="ng-tns-c986751816-110"><code class="code-container formatted ng-tns-c986751816-110" data-test-id="code-content">/oracle/app/grid/19.3.0/gridhome_1/bin/crsctl <span class="hljs-built_in">stat</span> res -t
</code></pre>
</div>
</div>
</div>
<p data-path-to-node="10"><em data-path-to-node="10" data-index-in-node="0">Look for <code data-path-to-node="10" data-index-in-node="9">ora.asm</code> and <code data-path-to-node="10" data-index-in-node="21">ora.cssd</code>. They need to be <strong data-path-to-node="10" data-index-in-node="47">ONLINE</strong>.</em></p>
<hr data-path-to-node="11" />
<h3 data-path-to-node="12">3. If CSS is still down (The "PRKH-3003" fix)</h3>
<p data-path-to-node="13">If the command above shows <code data-path-to-node="13" data-index-in-node="27">ora.cssd</code> is OFFLINE or STARTING, you cannot start ASM. As root, try to start the specific resources:</p>
<div class="code-block ng-tns-c986751816-111 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwi5nIDNorqSAxUAAAAAHQAAAAAQtAE">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c986751816-111 ng-star-inserted"><span class="ng-tns-c986751816-111">Bash</span>
<div class="buttons ng-tns-c986751816-111 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c986751816-111">
<div class="animated-opacity ng-tns-c986751816-111">
<pre class="ng-tns-c986751816-111"><code class="code-container formatted ng-tns-c986751816-111" data-test-id="code-content"><span class="hljs-comment"># Start the CSS daemon (required for ASM)</span>
/oracle/app/grid/19.3.0/gridhome_1/bin/crsctl start resource ora.cssd

<span class="hljs-comment"># Once CSS is online, start ASM</span>
/oracle/app/grid/19.3.0/gridhome_1/bin/crsctl start resource ora.asm</code></pre>
</div>
</div>
</div>
<!-- Comments are visible in the HTML source only -->
