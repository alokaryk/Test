<div id="model-response-message-contentr_bee3bf5e3ed38d64" class="markdown markdown-main-panel enable-updated-hr-color" dir="ltr">
<p>&nbsp;</p>
<h2>Kubespray and Kubernetes Cluster Deployment Guide</h2>
<p>&nbsp;</p>
<p>This guide defines <strong>Kubespray</strong> and provides a step-by-step process for deploying a highly available Kubernetes (K8s) cluster using Kubespray on infrastructure provisioned by <strong>Terraform on Proxmox</strong>.</p>
<hr />
<p>&nbsp;</p>
<h2>I. What is Kubespray?</h2>
<p>&nbsp;</p>
<p><strong>Kubespray</strong> is a composition of <strong>Ansible playbooks, inventory, and provisioning tools</strong> for deploying and configuring a production-ready Kubernetes cluster. It is not a Kubernetes installer itself, but rather an <strong>automation framework</strong> that simplifies the complex, multi-step process of setting up a cluster by providing idempotent (repeatable) automation.</p>
<p>&nbsp;</p>
<h3>Key Features:</h3>
<p>&nbsp;</p>
<ul>
<li>
<p><strong>High Availability (HA):</strong> Easily deploys clusters with multiple control plane nodes for fault tolerance.</p>
</li>
<li>
<p><strong>Idempotency:</strong> Allows you to run the playbooks multiple times without breaking the cluster.</p>
</li>
<li>
<p><strong>Flexibility:</strong> Supports various operating systems (Ubuntu, CentOS, etc.) and networking plugins (Calico, Cilium, Flannel).</p>
</li>
<li>
<p><strong>Customization:</strong> Uses a variable-driven approach in Ansible, making configuration management straightforward.</p>
</li>
</ul>
<hr />
<p>&nbsp;</p>
<h2>II. Deployment Architecture</h2>
<p>&nbsp;</p>
<p>This guide assumes the following 6-node HA architecture:</p>
<table>
<thead>
<tr>
<td>Role</td>
<td>Count</td>
<td>Description</td>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Control Plane (Controller)</strong></td>
<td>3</td>
<td>Runs the API Server, Scheduler, Controller Manager, and etcd. Provides high availability.</td>
</tr>
<tr>
<td><strong>Worker Node</strong></td>
<td>3</td>
<td>Runs user applications via the Kubelet and Container Runtime.</td>
</tr>
</tbody>
</table>
<hr />
<p>&nbsp;</p>
<h2>III. Step-by-Step Deployment Process</h2>
<p>&nbsp;</p>
<p>The deployment is divided into three phases: Infrastructure, Preparation, and Deployment.</p>
<p>&nbsp;</p>
<h3>Phase 1: Infrastructure Provisioning (Terraform on Proxmox)</h3>
<p>&nbsp;</p>
<p>This phase creates the 6 virtual machines (VMs) and sets up the deployment host.</p>
<ol start="1">
<li>
<p><strong>Set up Deployment Host:</strong> Ensure you have a management machine (your local PC or a dedicated VM) with <strong>Git</strong>, <strong>Terraform</strong>, <strong>Ansible</strong>, and <strong>SSH access</strong> configured.</p>
</li>
<li>
<p><strong>Terraform Configuration:</strong> Write Terraform configuration files (<code>main.tf</code>, <code>variables.tf</code>, etc.) to interface with the Proxmox API. This script must create six identical VMs based on a common template (e.g., Ubuntu 22.04 Cloud Image) and assign fixed IP addresses.</p>
<ul>
<li>
<p><em>The Terraform output should provide the public IPs and SSH usernames/keys for all 6 nodes.</em></p>
</li>
</ul>
</li>
<li>
<p><strong>Deploy VMs:</strong> Run the Terraform commands:</p>
<div class="code-block ng-tns-c1485810703-1707 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjm45fokpKQAxUAAAAAHQAAAAAQ6wg">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-1707 ng-star-inserted"><span class="ng-tns-c1485810703-1707">Bash</span>
<div class="buttons ng-tns-c1485810703-1707 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-1707">
<div class="animated-opacity ng-tns-c1485810703-1707">
<pre class="ng-tns-c1485810703-1707"><code class="code-container formatted ng-tns-c1485810703-1707" data-test-id="code-content">terraform init
terraform plan
terraform apply
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p><strong>Configure SSH Access:</strong> Ensure your SSH key is available on the deployment host and authorized on all 6 provisioned nodes.</p>
</li>
</ol>
<hr />
<p>&nbsp;</p>
<h3>Phase 2: Kubespray Preparation</h3>
<p>&nbsp;</p>
<p>This phase involves downloading Kubespray and generating the Ansible inventory file.</p>
<ol start="1">
<li>
<p><strong>Clone Kubespray Repository:</strong></p>
<div class="code-block ng-tns-c1485810703-1708 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjm45fokpKQAxUAAAAAHQAAAAAQ7Ag">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-1708 ng-star-inserted"><span class="ng-tns-c1485810703-1708">Bash</span>
<div class="buttons ng-tns-c1485810703-1708 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-1708">
<div class="animated-opacity ng-tns-c1485810703-1708">
<pre class="ng-tns-c1485810703-1708"><code class="code-container formatted ng-tns-c1485810703-1708" data-test-id="code-content">git <span class="hljs-built_in">clone</span> https://github.com/kubernetes-sigs/kubespray.git
<span class="hljs-built_in">cd</span> kubespray
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p><strong>Install Python Dependencies:</strong></p>
<div class="code-block ng-tns-c1485810703-1709 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjm45fokpKQAxUAAAAAHQAAAAAQ7Qg">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-1709 ng-star-inserted"><span class="ng-tns-c1485810703-1709">Bash</span>
<div class="buttons ng-tns-c1485810703-1709 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-1709">
<div class="animated-opacity ng-tns-c1485810703-1709">
<pre class="ng-tns-c1485810703-1709"><code class="code-container formatted ng-tns-c1485810703-1709" data-test-id="code-content">sudo pip install -r requirements.txt
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p>Prepare Inventory Directory:</p>
<p>Copy the sample inventory directory and rename it to your cluster name (mycluster):</p>
<div class="code-block ng-tns-c1485810703-1710 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjm45fokpKQAxUAAAAAHQAAAAAQ7gg">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-1710 ng-star-inserted"><span class="ng-tns-c1485810703-1710">Bash</span>
<div class="buttons ng-tns-c1485810703-1710 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-1710">
<div class="animated-opacity ng-tns-c1485810703-1710">
<pre class="ng-tns-c1485810703-1710"><code class="code-container formatted ng-tns-c1485810703-1710" data-test-id="code-content">cp -rf inventory/sample inventory/mycluster
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p>Generate Inventory File:</p>
<p>Use the inventory.py script to automatically generate the Ansible inventory file (inventory/mycluster/inventory.ini) based on the IP addresses of your nodes.</p>
<p>Assuming the IPs are: Controllers (<code>192.168.1.101-103</code>), Workers (<code>192.168.1.201-203</code>).</p>
<div class="code-block ng-tns-c1485810703-1711 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjm45fokpKQAxUAAAAAHQAAAAAQ7wg">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-1711 ng-star-inserted"><span class="ng-tns-c1485810703-1711">Bash</span>
<div class="buttons ng-tns-c1485810703-1711 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-1711">
<div class="animated-opacity ng-tns-c1485810703-1711">
<pre class="ng-tns-c1485810703-1711"><code class="code-container formatted ng-tns-c1485810703-1711" data-test-id="code-content"><span class="hljs-built_in">declare</span> -a IPS=(
  <span class="hljs-string">"192.168.1.101"</span> <span class="hljs-string">"192.168.1.102"</span> <span class="hljs-string">"192.168.1.103"</span>
  <span class="hljs-string">"192.168.1.201"</span> <span class="hljs-string">"192.168.1.202"</span> <span class="hljs-string">"192.168.1.203"</span>
)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py <span class="hljs-variable">${IPS[@]}</span>
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p>Configure Inventory Roles (inventory/mycluster/hosts.yaml):</p>
<p>Manually edit the generated hosts.yaml file to assign the correct roles for HA deployment:</p>
<div class="code-block ng-tns-c1485810703-1712 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjm45fokpKQAxUAAAAAHQAAAAAQ8Ag">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-1712 ng-star-inserted"><span class="ng-tns-c1485810703-1712">YAML</span>
<div class="buttons ng-tns-c1485810703-1712 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-1712">
<div class="animated-opacity ng-tns-c1485810703-1712">
<pre class="ng-tns-c1485810703-1712"><code class="code-container formatted ng-tns-c1485810703-1712" data-test-id="code-content"><span class="hljs-attr">all:</span>
  <span class="hljs-attr">hosts:</span>
    <span class="hljs-comment"># Control Plane (Master) Nodes</span>
    <span class="hljs-attr">node1:</span> { <span class="hljs-attr">ip:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.1</span><span class="hljs-number">.101</span> }
    <span class="hljs-attr">node2:</span> { <span class="hljs-attr">ip:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.1</span><span class="hljs-number">.102</span> }
    <span class="hljs-attr">node3:</span> { <span class="hljs-attr">ip:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.1</span><span class="hljs-number">.103</span> }
    <span class="hljs-comment"># Worker Nodes</span>
    <span class="hljs-attr">node4:</span> { <span class="hljs-attr">ip:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.1</span><span class="hljs-number">.201</span> }
    <span class="hljs-attr">node5:</span> { <span class="hljs-attr">ip:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.1</span><span class="hljs-number">.202</span> }
    <span class="hljs-attr">node6:</span> { <span class="hljs-attr">ip:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.1</span><span class="hljs-number">.203</span> }
  <span class="hljs-attr">children:</span>
    <span class="hljs-attr">kube_control_plane:</span>
      <span class="hljs-attr">hosts:</span>
        <span class="hljs-attr">node1:</span>
        <span class="hljs-attr">node2:</span>
        <span class="hljs-attr">node3:</span>
    <span class="hljs-attr">etcd:</span>
      <span class="hljs-attr">hosts:</span>
        <span class="hljs-attr">node1:</span>
        <span class="hljs-attr">node2:</span>
        <span class="hljs-attr">node3:</span>
    <span class="hljs-attr">kube_node:</span>
      <span class="hljs-attr">hosts:</span>
        <span class="hljs-attr">node1:</span>  <span class="hljs-comment"># Control plane nodes also function as worker nodes by default</span>
        <span class="hljs-attr">node2:</span>
        <span class="hljs-attr">node3:</span>
        <span class="hljs-attr">node4:</span>
        <span class="hljs-attr">node5:</span>
        <span class="hljs-attr">node6:</span>
    <span class="hljs-comment"># ... other groups</span>
</code></pre>
</div>
</div>
</div>
</li>
</ol>
<hr />
<p>&nbsp;</p>
<h3>Phase 3: Cluster Deployment (Kubespray Playbook)</h3>
<p>&nbsp;</p>
<p>This phase executes the main Kubespray playbook to install and configure K8s.</p>
<ol start="1">
<li>
<p>Configure Cluster Variables:</p>
<p>Review and modify critical variables in the configuration files (inventory/mycluster/group_vars/*.yml).</p>
<ul>
<li>
<p><strong><code>all.yml</code>:</strong> Set <code>ansible_user</code> (e.g., to <code>ubuntu</code>) and <code>ntp_enabled: true</code>.</p>
</li>
<li>
<p><strong><code>k8s-cluster.yml</code>:</strong> Set the Pod Network CIDR (<code>kube_pods_cidr</code>) and the Service CIDR (<code>kube_service_addresses</code>). Crucially, set the Container Networking Interface (CNI) plugin (e.g., <code>kube_network_plugin: calico</code>).</p>
</li>
</ul>
</li>
<li>
<p>Execute the Deployment Playbook:</p>
<p>Run the main Ansible playbook, pointing it to your custom inventory file:</p>
<div class="code-block ng-tns-c1485810703-1713 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjm45fokpKQAxUAAAAAHQAAAAAQ8Qg">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-1713 ng-star-inserted"><span class="ng-tns-c1485810703-1713">Bash</span>
<div class="buttons ng-tns-c1485810703-1713 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-1713">
<div class="animated-opacity ng-tns-c1485810703-1713">
<pre class="ng-tns-c1485810703-1713"><code class="code-container formatted ng-tns-c1485810703-1713" data-test-id="code-content">ansible-playbook -i inventory/mycluster/hosts.yaml --become --user=[SSH_USER] cluster.yml
</code></pre>
</div>
</div>
</div>
<p><em>Replace <code>[SSH_USER]</code> with the username you set up in your Proxmox templates (e.g., <code>ubuntu</code> or <code>centos</code>).</em></p>
</li>
<li>
<p>Verification:</p>
<p>Once the playbook completes, copy the Kubeconfig file from one of the control plane nodes (e.g., node1) to your deployment host.</p>
<div class="code-block ng-tns-c1485810703-1714 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjm45fokpKQAxUAAAAAHQAAAAAQ8gg">
<div class="code-block-decoration header-formatted gds-title-s ng-tns-c1485810703-1714 ng-star-inserted"><span class="ng-tns-c1485810703-1714">Bash</span>
<div class="buttons ng-tns-c1485810703-1714 ng-star-inserted">&nbsp;</div>
</div>
<div class="formatted-code-block-internal-container ng-tns-c1485810703-1714">
<div class="animated-opacity ng-tns-c1485810703-1714">
<pre class="ng-tns-c1485810703-1714"><code class="code-container formatted ng-tns-c1485810703-1714" data-test-id="code-content"><span class="hljs-comment"># Run this on your deployment host</span>
scp [SSH_USER]@192.168.1.101:~/.kube/config ~/.kube/config
<span class="hljs-built_in">export</span> KUBECONFIG=~/.kube/config
kubectl get nodes
</code></pre>
</div>
</div>
</div>
<p><strong>Expected Output:</strong> All 6 nodes should appear with a <code>Ready</code> status, confirming a successful 3-Controller/3-Worker HA cluster deployment.</p>
</li>
</ol>
</div>
