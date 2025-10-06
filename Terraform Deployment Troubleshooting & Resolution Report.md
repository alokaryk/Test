<div class="ProseMirror" contenteditable="true" translate="no">
<h1>Terraform Deployment Troubleshooting &amp; Resolution Report (Proxmox K8s Cluster)</h1>
<p>Goal: Deploy a 7-node Kubernetes cluster (1 Bastion, 3 Control Plane, 3 Worker) on Proxmox VE 8.4.0 using Terraform, ensuring sequential VM creation and correct provider syntax.</p>
<p>Status: RESOLVED (terraform apply successful after applying final syntax fixes).</p>
<p>Terraform Host: 192.168.1.100</p>
<h2>1. Phase 1: Provider Compatibility Conflicts</h2>
<p>The initial challenge was determining the correct, compatible Terraform provider for Proxmox VE 8.4.0, as the community providers frequently change syntax and maintenance status.</p>
<p>|</p>
<p>| Problem | Symptoms &amp; Errors | Root Cause | Resolution |</p>
<p>| P1: Deprecated Provider (Initial) | Configuration used source = "telmate/proxmox" and resource proxmox_vm_qemu. | The telmate/proxmox provider is deprecated and incompatible with modern Proxmox (8.x) and required nested blocks. | Action: Switched to the actively maintained community provider, bpg/proxmox, and updated the required_providers block and resource type to proxmox_vm_qemu (later confirmed proxmox_virtual_environment_vm). |</p>
<p>| P2: Registry Lookup Failure | Error: Failed to query available provider packages for bpg/proxmoxve. | The specific provider name bpg/proxmoxve was retired/renamed on the Terraform Registry, or the local cache was corrupted. | Action: Cleared the local cache (rm -rf .terraform), and switched the source name to the currently active bpg/proxmox (without the ve suffix). |</p>
<p>| P3: Provider Argument Naming | Error: Unsupported argument on pm_api_url, pm_user, etc. | The new bpg/proxmox provider uses modern, explicit attribute names, while the configuration used legacy telmate prefixes (pm_api_url). | Action: Updated the provider "proxmox" block attributes to the correct names (endpoint, username, password, insecure). |</p>
<h2>2. Phase 2: Configuration Syntax Errors</h2>
<p>The strict HCL parsing rules of the <code>bpg/proxmox</code> provider rejected several common Terraform patterns.</p>
<p>| Problem | Symptoms &amp; Errors | Root Cause | Resolution |</p>
<p>| P4: Cloud-Init Block Structure | Error: Unsupported block type on initialization, ci_public_keys, ipconfig, etc. | The provider requires a specific, multi-layered nesting for Cloud-Init settings, rejecting both the telmate flat structure and simplified modern structures. | Action: Enforced the fully correct, multi-line nested block structure (initialization { ip_config { ipv4 { ... } } }) across all VM definitions. |</p>
<p>| P5: Inline HCL Syntax | Error: Invalid character (on ;) and Argument definition required. | Attempted to use compressed, single-line HCL blocks (e.g., cpu { cores = 1; type = "host" }), which is invalid in this provider. | Action: Manually expanded all compressed blocks (e.g., cpu, clone, initialization) to the strict, verbose multi-line HCL format. |</p>
<p>| P6: VM Resource Naming | Error: Invalid resource type on proxmox_vm_qemu. | The final provider (bpg/proxmox) uses the modern resource name for VMs. | Action: Changed all VM resource names to proxmox_virtual_environment_vm. |</p>
<h2>3. Phase 3: Infrastructure Logic &amp; Provisioning</h2>
<p>| Problem | Symptoms &amp; Errors | Root Cause | Resolution |</p>
<p>| P7: API Connection Timeout (HTTP 596) | Error: error waiting for VM clone: ... received an HTTP 596 response - Reason: Connection timed out. | Proxmox experienced high I/O load trying to clone and start 7 large VMs simultaneously, causing the API request from the Terraform Host to time out. | Action: Removed the for_each loop for VM creation and implemented a sequential deployment strategy by breaking the VM creation into 7 individual blocks (k8s_control_01, k8s_control_02, etc.) and linking them with explicit depends_on clauses. |</p>
<p>| P8: Provisioner Logic | Error: Unsupported argument on proxy_command and only_if. | The provisioning blocks were initially incorrectly placed inside the proxmox_virtual_environment_vm resources, and the null_resource used the wrong connection syntax. | Action: Moved all provisioning logic into dedicated null_resource blocks and updated the connection blocks to use the correct modern arguments (bastion_host, bastion_user). |</p>
<h2>Conclusion</h2>
<p>The final, successful configuration utilizes the stable <code>bpg/proxmox</code> provider and addresses the infrastructure's resource limitations by forcing sequential deployment.</p>
<p>The deployment will proceed in the following fixed order:</p>
<ol>
<li>
<p><code>proxmox_virtual_environment_vm.bastion</code> (VMID 401)</p>
</li>
<li>
<p><code>proxmox_virtual_environment_vm.k8s_control_01</code> (VMID 402)</p>
</li>
<li>
<p>...</p>
</li>
<li>
<p><code>proxmox_virtual_environment_vm.k8s_worker_03</code> (VMID 407)</p>
</li>
<li>
<p><code>null_resource.k8s_init</code> (Initializes K8s on Control-01)</p>
</li>
<li>
<p><code>null_resource.k8s_join_nodes_controls</code> (Joins C02, C03)</p>
</li>
<li>
<p><code>null_resource.k8s_join_nodes_workers</code> (Joins W01, W02, W03)</p>
</li>
</ol>
<p>The cluster is now ready for the final configuration steps: CNI installation (Flannel) and NFS CSI deployment using Helm.</p>
</div>
<!-- Comments are visible in the HTML source only -->
