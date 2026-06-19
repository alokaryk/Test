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




<div id="model-response-message-contentr_6019ed25fb20fe55" class="markdown markdown-main-panel enable-luminous-fast-follows enable-updated-hr-color" dir="ltr">
<p data-path-to-node="0">apiVersion: v1<br />clusters:<br />- cluster:<br /> certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM2akNDQWRLZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJMk1EWXhPVEEyTWpRek0xb1hEVE0yTURZeE5qQTJNamt6TTFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBT3lhCnVONmtsVUlSYW8wMnZ4L1c2ZmtFb0RhcjhFNkN2QmNVV0F0YkhLS1dUMVNLc29FSFFucDFoQnp3RnUrRFZtaUcKUzB5QjVWa0IwVk1ZM3FxMHlCQk12ekNRY04zUDhmL2k3TkNZamZ2TVdhU0hZbUxJWHpKRHFZM1Azd1VzU2NCcgpiUTdWemN4QnNaUzBvS2Fmd3FjejlvSEJqN1NDMFliSkZnZ29RN1RXSE51YmhVWG84RzBDSnl6eEhvbWt1eGkrCkhST1J2cEthV2FZTk5Rbll6aHhERm1TQ3o2cHBxRzR2bDI2R3dVWVVwckdwdGpIK1FIc20wc20wc1M4MHNnekoKUjVKamZWcXRiN0Z0TlFnNEx5R09XRmFBM3VxN1MwL2I3dkl2UXVBRkIvUEkvRUdybjVTblRZVW1CQ0tNVzM2NApwcU1XakRFOVU0UHR4U3R6clJzQ0F3RUFBYU5GTUVNd0RnWURWUjBQQVFIL0JBUURBZ0trTUJJR0ExVWRFd0VCCi93UUlNQVlCQWY4Q0FRQXdIUVlEVlIwT0JCWUVGS0NqNzVJeFFranJUNTREdVVvYjh1MFpnelV6TUEwR0NTcUcKU0liM0RRRUJDd1VBQTRJQkFRRFNxS3NtcXFQckhHQXMwNzVZaEhTM0ZxTmhGOVB5d0tvRVptRVprdUNLUkhwdwpTK3JqNjQ4UExUQnRyc3M5UG92QXRPYzYrMyszSGJXUzFoaGxpeDI5UXd0enVMTGVzVE42YWxUbWVMczhNSDV5CmVzNlcwUnBrWndQaW5FQXRoR3lBN0dtRitrTlF6Rkd3ZVlYWDdpUzNVQnhud1BaTmYrNkgyZTFyQk1yVUQvbFcKVUlINnRqT3VoYXVTb1k2UmJvTFRsTFNLTmM0dXcya0F5RWVJSHIvL1FEZTNpcGhJemh5VGpLelZmOGJaa2VMdAo5dVpZdjNzSDMySHU1UE5xTlJZcTZjc3FUS3JWVWJqQzN6emJ5ZGtTaWlPN1I2cVpOanVhb1gxMWZhc2Q2ZlpuCkdXM251azdiVElkUWRBaElXQkNQN0xCUjFGb2tVbHZpS3lBdllnWXQKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=<br /> server: https://49.50.117.30:6443<br /> name: backuptest<br />contexts:<br />- context:<br /> cluster: backuptest<br /> user: backuptest-admin<br /> name: backuptest-admin@backuptest<br />current-context: backuptest-admin@backuptest<br />kind: Config<br />preferences: {}<br />users:<br />- name: backuptest-admin<br /> user:<br /> client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURFekNDQWZ1Z0F3SUJBZ0lJS1dIWUhJLzFudEF3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TmpBMk1Ua3dOakkwTXpOYUZ3MHlOekEyTVRrd05qSTVNelJhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXFid0ZiZFMrTCsxOVZDU2MKMk94M3hldmlKUVpYeW1ZZTRaTU5DWCs2d2lxVFFHSHozaTFlY3dVSkQ0anpUckdHMlVEQjZUMXU4Vy8yOElBNQo1OW5EUXRxUHFESnhBMzZCZ1AxQmJ4SUtCbjRJK25aNDJsbVA3c2RpaFNIZ2tXSEZ1Z1pzM3BNVll3b3NCdXBCCnZ3UUVhS0VIYzJMS3Y3dlRKc2NNR1FFS2M2ZU9XcEdvVEdvcEVqMEVQQ3A3VWJTTmlMaERtRW10Q2JielZxb3oKbEJkdUZMMVJjYThyNUFLTWphT2F0ZUlNRWtsRkpubGtxN1dvZ1hndlh1d2ZFZjJTNEJDV2dJdGw0WHhacytpNAovc0tEK3prdnB4TmtMOXZzS01TeE5MczJEUFBkZGordTQ4QmFyeXExcTMvWGhQYVQ1YWpNR1BsNW9OSEhvR2ozCjFLcXNOd0lEQVFBQm8wZ3dSakFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0h3WURWUjBqQkJnd0ZvQVVvS1B2a2pGQ1NPdFBuZ081U2h2eTdSbUROVE13RFFZSktvWklodmNOQVFFTApCUUFEZ2dFQkFFMXFyRjQ3VERFb2NvcFUvZ29oWG9jd25uSFUvbWxqcEUxamIxTkYxR3U4RHdBWCtqQkF4SHpmClJQY1d3QjJTcGEzYnlxRHRROEdIam9Vb2NJVEd6OHZxNENDNk1qTEFva3lrVkkza3V5Vjc2d3V5L0JFUStwTGYKeTR0MWs5a2JDd3FYb0p0UG1lcG9vS2luaWNNWEpPaEMzbmkwdHBkZFdmL1FqNmJtLzhlNXh3UGFYTzVUM2daYwpIMDNXcDg1L1pSdUVBM1JhQzhFWCtGK0VtWTFhbjJGbzlrdTF4QktLbms1ZlpzMC9uVGFmREFLZFM1bGRsWUlkClRBYUFSS0NyMkdCWWI1M0Z2L3dhTTRLdWZSK3dHUTU0NThKM1JYN2tmeWtpankwUmZhZGJ5bHExcVVSbEt3ZjQKOWtyMGVsTTIwMzJyY0U4YW9nY3NDU29NdVNqZFJGQT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=<br /> client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcFFJQkFBS0NBUUVBcWJ3RmJkUytMKzE5VkNTYzJPeDN4ZXZpSlFaWHltWWU0Wk1OQ1grNndpcVRRR0h6CjNpMWVjd1VKRDRqelRyR0cyVURCNlQxdThXLzI4SUE1NTluRFF0cVBxREp4QTM2QmdQMUJieElLQm40SStuWjQKMmxtUDdzZGloU0hna1dIRnVnWnMzcE1WWXdvc0J1cEJ2d1FFYUtFSGMyTEt2N3ZUSnNjTUdRRUtjNmVPV3BHbwpUR29wRWowRVBDcDdVYlNOaUxoRG1FbXRDYmJ6VnFvemxCZHVGTDFSY2E4cjVBS01qYU9hdGVJTUVrbEZKbmxrCnE3V29nWGd2WHV3ZkVmMlM0QkNXZ0l0bDRYeFpzK2k0L3NLRCt6a3ZweE5rTDl2c0tNU3hOTHMyRFBQZGRqK3UKNDhCYXJ5cTFxMy9YaFBhVDVhak1HUGw1b05ISG9HajMxS3FzTndJREFRQUJBb0lCQUJieStoVHdoOHA1Sk5IawpwV1JiREpLeEl3RjRpeFF0bkkxSlVhRHdLVE1waUlGUy9TTVVKVW9ONnp5emVwb3dQSmhSUGlhb0RNRU9MMmd6CkhpRXYrMHVsdTNpMVlUeGt0V1BZV2ltSFdkMm8ydFBxZ3NxYkEyLzRlMlNld1B0SEtmSE4vcGhWY0xYVVlVR0sKR051WDBuVEhHUGZMNnJmajBGZlUzOWpkb0Nra3pSRUVPZU9ta3lYL0Fxd0pMQ256aDBxb0tCT3N2TXBnMFFaegpDbFRDRzNra0plQ2xndzN5RlRmRzAyZUpMYWJwdnNoYmxTaUlKT0FXNExSenNlQi9CbGdVT0h0TVJSdEtyOC9mCm5jSUh0SnNNRjVzek5hZzJ1UERrSHZuU2x6YndDRGNrNmhneElJTnlPR2ZxeUxZSksvSWtUT0xHeWM1MWZ4WFUKTVJUWEtua0NnWUVBM0ZJNkJJdUp5S0dlSDZnSjlLNUhYYjV5S1o5UURUaXZ5ZkZuaVNGTTRBc1V2V3Rhd2NvZApNV2gyeFhXSWpjcStIYnBaYWorZEJVUFFEVDNaRnFSVGFwMU5NdWljK3FjZUdMb1liMDUwVFZVS0pJZmp2dEs2CjB2TklFNk45aDFVbTVsOWwrOWZmVkV1eit1VE1CUDR2b3UzZ01VdzZaaEcrN2duOE9yODFzbzBDZ1lFQXhUaWoKbUl3alBFZis3MXArcFZoMjFFSmIrcmUrMFlGTkdDUGhCWk5IMFdkTjFRamNZZWx4UFdSVUxVVk5TOGFtbzBZSgpWNGlTM0xuNjkvOUN3REk5VFV1c2svL001TjdLY2o5QkhkQmp2MTNvZVQ2WWR5Qmo0NkRCaGdYcDlOSC9VVEd2Cm9MS2hJSnJNREVYUTF0MkNuWTB5NitURGtRWVlKbTFrWE9mckN0TUNnWUVBMVNrZEJ1NjJaUTJ3L2ZISGlCcmgKNDFueC9FRXQ0MHpZRHlybndDNGV3WmtXL05rVUd2QmdaRUFzd1lpZ2tJYjZmNnFlL2dRRCtHY09zUEVJeXpZNwptcWdNVzlmZ29VWDFBWGx6VUtWNE1xVzNSWGJRS3Z5WlVxV2NNNXYzUkd0blVuWTBKb0JLaTdVZXNmeVJRN2RtCkFWM2lqZSt4aExWZ284KzNZR05GSWJVQ2dZRUF1N0pmNERyZS9ldjNNSTBMb3dsQ2JsaUplVitPZDQxN2RkSVAKd21LZEl5b1I1UUg5M0NGejZGMm56ejM1aG5pVTl5Y2NsUFloNzNnQTVpMUZ0ZTBhY29ubXdTUGVPS1BnUUVMNwpFZ2pXSmJhV0hXMzVaTWFxeW1TZ1NaRXZUQlRmek1KZ3g2dTV0WElkYUVDNXRRUXozd1lXeEhKL3ZYSHpZZjhTCldXbFBSSWtDZ1lFQWlTYlFWU3gxaHQ2alFYYkhhK3JLL1pVcS8rL1pqQmszWWpsTTg5TkNHdEhuc2JPT1pnWGQKS0tHb1NDTXBIZmU3L2l1eXNVeThTNEJHVjhQWWlXbC9jRUwwTnJoaTg3V1owSFNKZVl4U0FNd3o1VWZYQlErUgpOVDNISDNtMXdBQ0NvZE91WkNncm9INXRGZXlTZTNLTmFxNWNyZ2VwektuT2NKbGhXR01GZ2tFPQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=</p>
<p data-path-to-node="0">&nbsp;</p>
<div id="model-response-message-contentr_86c421869f91652d" class="markdown markdown-main-panel enable-luminous-fast-follows enable-updated-hr-color" dir="ltr">
<p data-path-to-node="0">This file is your cluster's official <strong data-path-to-node="0" data-index-in-node="37"><code data-path-to-node="0" data-index-in-node="37">kubeconfig</code></strong> credential mapping payload, and the accompanying connection parameters detail the administrative access route to the <strong data-path-to-node="0" data-index-in-node="165">external load balancer/gateway</strong> managing your <code data-path-to-node="0" data-index-in-node="210">backuptest</code> cluster topology.</p>
<h3 data-path-to-node="2">1. What This File Contains</h3>
<p data-path-to-node="3">The text block is a fully populated standard Kubernetes configuration file split into three distinct segments:</p>
<ul data-path-to-node="4">
<li>
<p data-path-to-node="4,0,0"><strong data-path-to-node="4,0,0" data-index-in-node="0">Cluster Endpoint &amp; Certificate Authority (<code data-path-to-node="4,0,0" data-index-in-node="42">certificate-authority-data</code>):</strong> This maps the destination API endpoint (<code data-path-to-node="4,0,0" data-index-in-node="111">https://49.50.117.30:6443</code>) and embeds the base64-encoded root certificate so your terminal knows how to establish a trusted TLS connection without throwing security warnings.</p>
</li>
<li>
<p data-path-to-node="4,1,0"><strong data-path-to-node="4,1,0" data-index-in-node="0">User Credentials:</strong> The blocks <code data-path-to-node="4,1,0" data-index-in-node="29">client-certificate-data</code> and <code data-path-to-node="4,1,0" data-index-in-node="57">client-key-data</code> represent your high-level administrative cryptographic user keys (<code data-path-to-node="4,1,0" data-index-in-node="139">backuptest-admin</code>). This functions as a cluster root password.</p>
</li>
<li>
<p data-path-to-node="4,2,0"><strong data-path-to-node="4,2,0" data-index-in-node="0">Context:</strong> It links user <code data-path-to-node="4,2,0" data-index-in-node="23">backuptest-admin</code> to cluster <code data-path-to-node="4,2,0" data-index-in-node="51">backuptest</code> so that when you run any command, it automatically authenticates you with cluster-admin privileges.</p>
</li>
</ul>
<h3 data-path-to-node="6">2. What It Is Used For</h3>
<p data-path-to-node="7">This file is used to grant your local machine or external control management servers remote administrative access to manage the cluster without needing to SSH directly into the master nodes.</p>
<p data-path-to-node="8">Since you are standing up Kasten on the replica node, you need to save this file to let your automation layers communicate with your testing cluster API cleanly.</p>
<h3 data-path-to-node="10">🛠️ How to Apply and Use This File Now</h3>
<p data-path-to-node="11">If you want to manage this new cluster from your remote deployment node or configure external tooling, execute these commands on the control machine:</p>
<ol start="1" data-path-to-node="12">
<li>
<p data-path-to-node="12,0,0">Copy the entire text block starting from <code data-path-to-node="12,0,0" data-index-in-node="41">apiVersion: v1</code> to the bottom of the private key line.</p>
</li>
<li>
<p data-path-to-node="12,1,0">Save it to your root deployment workspace directory:</p>
<div class="code-block ng-tns-c1277634023-199 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjf49zuupGVAxUAAAAAHQAAAAAQhAM">
<div class="formatted-code-block-internal-container ng-tns-c1277634023-199">
<div class="animated-opacity ng-tns-c1277634023-199">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c1277634023-199 ng-star-inserted"><span class="ng-tns-c1277634023-199">Bash</span>
<div class="buttons ng-tns-c1277634023-199 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c1277634023-199"><code class="code-container formatted ng-tns-c1277634023-199" data-test-id="code-content">nano ~/.kube/config-backuptest
</code></pre>
</div>
</div>
</div>
<p data-path-to-node="12,1,2"><em data-path-to-node="12,1,2" data-index-in-node="0">(Paste the configuration text inside and save).</em></p>
</li>
<li>
<p data-path-to-node="12,2,0">Initialize your runtime shell to point to this connection mapping profile:</p>
<div class="code-block ng-tns-c1277634023-200 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjf49zuupGVAxUAAAAAHQAAAAAQhQM">
<div class="formatted-code-block-internal-container ng-tns-c1277634023-200">
<div class="animated-opacity ng-tns-c1277634023-200">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c1277634023-200 ng-star-inserted"><span class="ng-tns-c1277634023-200">Bash</span>
<div class="buttons ng-tns-c1277634023-200 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c1277634023-200"><code class="code-container formatted ng-tns-c1277634023-200" data-test-id="code-content"><span class="hljs-built_in">export</span> KUBECONFIG=~/.kube/config-backuptest
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p data-path-to-node="12,3,0">Verify that you can seamlessly communicate with the control plane from across the management network layer:</p>
<div class="code-block ng-tns-c1277634023-201 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjf49zuupGVAxUAAAAAHQAAAAAQhgM">
<div class="formatted-code-block-internal-container ng-tns-c1277634023-201">
<div class="animated-opacity ng-tns-c1277634023-201">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c1277634023-201 ng-star-inserted"><span class="ng-tns-c1277634023-201">Bash</span>
<div class="buttons ng-tns-c1277634023-201 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c1277634023-201"><code class="code-container formatted ng-tns-c1277634023-201" data-test-id="code-content">kubectl get nodes
</code></pre>
</div>
</div>
</div>
<p data-path-to-node="12,3,2">This will immediately display your <code data-path-to-node="12,3,2" data-index-in-node="35">backuptest</code> control-plane and worker nodes straight from your control workspace.</p>
</li>
</ol>
</div>
</div>
<!-- Comments are visible in the HTML source only -->
