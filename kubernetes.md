Run kubectl get pods -A and kubectl get deployments -A on the new fresh master node.


<div id="model-response-message-contentr_6019ed25fb20fe55" class="markdown markdown-main-panel enable-luminous-fast-follows enable-updated-hr-color" dir="ltr">
<p data-path-to-node="0">if you are getting error root@backuptest-control-plane-node-pool-ntnzs:~# kubectl get pods -A<br />E0619 07:21:08.125515 8260 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused<br />E0619 07:21:08.125931 8260 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused</p>
<p data-path-to-node="0">Excellent initiative setting up the <code data-path-to-node="0" data-index-in-node="36">backuptest</code> environment. Before we can run any Kasten recovery policies, we have a critical blocker to solve on this new master node.</p>
<p data-path-to-node="1">The error <strong data-path-to-node="1" data-index-in-node="10"><code data-path-to-node="1" data-index-in-node="10">connect: connection refused - did you specify the right host or port?</code></strong> means <code data-path-to-node="1" data-index-in-node="86">kubectl</code> cannot talk to the Kubernetes API server. Your <code data-path-to-node="1" data-index-in-node="141">ip a</code> output shows that the <strong data-path-to-node="1" data-index-in-node="168">Antrea CNI</strong> and <strong data-path-to-node="1" data-index-in-node="183">Open vSwitch (OVS)</strong> networking stacks are already running, which means the node configuration is mostly intact, but the local administrative context or the master control components are offline.</p>
<p data-path-to-node="1">root@backuptest-control-plane-node-pool-ntnzs:~# ip a<br />1: lo: &lt;LOOPBACK,UP,LOWER_UP&gt; mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000<br /> link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00<br /> inet 127.0.0.1/8 scope host lo<br /> valid_lft forever preferred_lft forever<br />2: ens192: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc prio state UP group default qlen 1000<br /> link/ether 00:50:56:01:44:6e brd ff:ff:ff:ff:ff:ff<br /> altname enp11s0<br /> inet 192.168.0.18/24 brd 192.168.0.255 scope global ens192<br /> valid_lft forever preferred_lft forever<br />3: ovs-system: &lt;BROADCAST,MULTICAST&gt; mtu 1500 qdisc noop state DOWN group default qlen 1000<br /> link/ether 1a:8c:30:18:e2:04 brd ff:ff:ff:ff:ff:ff<br />4: genev_sys_6081: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 65000 qdisc noqueue master ovs-system state UNKNOWN group default qlen 1000<br /> link/ether 6e:8c:6a:65:b2:9e brd ff:ff:ff:ff:ff:ff<br />5: antrea-gw0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1450 qdisc noqueue state UNKNOWN group default qlen 1000<br /> link/ether a2:5c:26:c5:08:61 brd ff:ff:ff:ff:ff:ff<br /> inet 100.96.0.1/24 brd 100.96.0.255 scope global antrea-gw0<br /> valid_lft forever preferred_lft forever<br />6: antrea-egress0: &lt;BROADCAST,NOARP&gt; mtu 1500 qdisc noop state DOWN group default<br /> link/ether ce:05:e0:39:a1:06 brd ff:ff:ff:ff:ff:ff<br />7: csi-vcd--f85c64@if2: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1450 qdisc noqueue master ovs-system state UP group default<br /> link/ether 9e:94:7d:24:4d:a8 brd ff:ff:ff:ff:ff:ff link-netns cni-ed9bf7f2-84e9-4c53-7eae-0a509ffa3e3b<br />8: coredns--962031@if2: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1450 qdisc noqueue master ovs-system state UP group default<br /> link/ether 36:86:10:ee:e4:ac brd ff:ff:ff:ff:ff:ff link-netns cni-6ccc3b3a-2bbc-5d1b-9b56-39f162bed2fb<br />9: coredns--4b7cc6@if2: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1450 qdisc noqueue master ovs-system state UP group default<br /> link/ether ca:10:77:60:59:b2 brd ff:ff:ff:ff:ff:ff link-netns cni-4888e739-9474-68d4-7d83-3ab2aec211db<br />root@backuptest-control-plane-node-pool-ntnzs:~#</p>
<p data-path-to-node="2">Let's debug and fix this so you can start the restore test.</p>
<h3 data-path-to-node="4">Step 1: Fix the <code data-path-to-node="4" data-index-in-node="16">KUBECONFIG</code> environment variable</h3>
<p data-path-to-node="5">If this is a fresh Ubuntu 22.04 installation or if you are logged in as <code data-path-to-node="5" data-index-in-node="72">root</code>, <code data-path-to-node="5" data-index-in-node="78">kubectl</code> looks for authentication at port <code data-path-to-node="5" data-index-in-node="119">8080</code> by default unless you point it to the admin configuration file.</p>
<p data-path-to-node="6">Run these commands to link the correct cluster credentials:</p>
<div class="code-block ng-tns-c1277634023-141 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjf49zuupGVAxUAAAAAHQAAAAAQlgI">
<div class="formatted-code-block-internal-container ng-tns-c1277634023-141">
<div class="animated-opacity ng-tns-c1277634023-141">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c1277634023-141 ng-star-inserted"><span class="ng-tns-c1277634023-141">Bash</span>
<div class="buttons ng-tns-c1277634023-141 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c1277634023-141"><code class="code-container formatted ng-tns-c1277634023-141" data-test-id="code-content"><span class="hljs-built_in">export</span> KUBECONFIG=/etc/kubernetes/admin.conf
mkdir -p <span class="hljs-variable">$HOME</span>/.kube
cp -i /etc/kubernetes/admin.conf <span class="hljs-variable">$HOME</span>/.kube/config
chown $(id -u):$(id -g) <span class="hljs-variable">$HOME</span>/.kube/config
</code></pre>
</div>
</div>
</div>
<p data-path-to-node="8">Try running <code data-path-to-node="8" data-index-in-node="12">kubectl get nodes</code> right after this. If it works, the issue was just a missing path.</p>
<h3 data-path-to-node="10">Step 2: Check the Kubernetes Control Plane Services</h3>
<p data-path-to-node="11">If Step 1 still yields a connection error, it means the master services (<code data-path-to-node="11" data-index-in-node="73">kube-apiserver</code> and <code data-path-to-node="11" data-index-in-node="92">etcd</code>) are crashing or stopped. Let's see if the underlying containers are running:</p>
<div class="code-block ng-tns-c1277634023-142 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjf49zuupGVAxUAAAAAHQAAAAAQlwI">
<div class="formatted-code-block-internal-container ng-tns-c1277634023-142">
<div class="animated-opacity ng-tns-c1277634023-142">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c1277634023-142 ng-star-inserted"><span class="ng-tns-c1277634023-142">Bash</span>
<div class="buttons ng-tns-c1277634023-142 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c1277634023-142"><code class="code-container formatted ng-tns-c1277634023-142" data-test-id="code-content">crictl ps -a | grep kube-apiserver
</code></pre>
</div>
</div>
</div>
<ul data-path-to-node="13">
<li>
<p data-path-to-node="13,0,0"><strong data-path-to-node="13,0,0" data-index-in-node="0">If nothing shows up or they show as <code data-path-to-node="13,0,0" data-index-in-node="36">Exited</code>:</strong> Check the system service engine logs to see why Kubernetes failed to start:</p>
<div class="code-block ng-tns-c1277634023-143 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjf49zuupGVAxUAAAAAHQAAAAAQmAI">
<div class="formatted-code-block-internal-container ng-tns-c1277634023-143">
<div class="animated-opacity ng-tns-c1277634023-143">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c1277634023-143 ng-star-inserted"><span class="ng-tns-c1277634023-143">Bash</span>
<div class="buttons ng-tns-c1277634023-143 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c1277634023-143"><code class="code-container formatted ng-tns-c1277634023-143" data-test-id="code-content">journalctl -xeu kubelet --no-pager
</code></pre>
</div>
</div>
</div>
</li>
</ul>
<h3 data-path-to-node="15">💡 Common Causes on Fresh Replicas to Check in the Logs:</h3>
<ol start="1" data-path-to-node="16">
<li>
<p data-path-to-node="16,0,0"><strong data-path-to-node="16,0,0" data-index-in-node="0">Swap is Enabled:</strong> Ubuntu 22.04 sometimes reactivates swap on reboot. Kubernetes will instantly stop working if swap is active. Disable it immediately by running:</p>
<div class="code-block ng-tns-c1277634023-144 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjf49zuupGVAxUAAAAAHQAAAAAQmQI">
<div class="formatted-code-block-internal-container ng-tns-c1277634023-144">
<div class="animated-opacity ng-tns-c1277634023-144">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c1277634023-144 ng-star-inserted"><span class="ng-tns-c1277634023-144">Bash</span>
<div class="buttons ng-tns-c1277634023-144 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c1277634023-144"><code class="code-container formatted ng-tns-c1277634023-144" data-test-id="code-content">swapoff -a
<span class="hljs-comment"># Permanently disable it by commenting out the swap line here:</span>
sed -i <span class="hljs-string">'/swap/s/^/#/'</span> /etc/fstab
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p data-path-to-node="16,1,0"><strong data-path-to-node="16,1,0" data-index-in-node="0">Expired or Hostname Mismatched Certificates:</strong> If this replica node was cloned or restored from a template image, the IP listed in <code data-path-to-node="16,1,0" data-index-in-node="129">ens192</code> (<code data-path-to-node="16,1,0" data-index-in-node="137">192.168.0.18</code>) might not be whitelisted inside the original API server SAN certificates.</p>
</li>
</ol>
<p data-path-to-node="17">Once <code data-path-to-node="17" data-index-in-node="5">kubectl get pods -A</code> successfully lists your running system components without a connection refusal, let me know, and we will immediately proceed with Phase 2 to mount the <code data-path-to-node="17" data-index-in-node="176">exportbackup</code> profile and trigger your application restoration test!</p>
</div>
<!-- Comments are visible in the HTML source only -->
