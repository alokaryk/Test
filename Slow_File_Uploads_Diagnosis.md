<h1 data-pm-slice="1 4 []">Case Study: Slow File Uploads Diagnosis (CLI Runbook)</h1>
<h2>Overview</h2>
<p>This document outlines the diagnostic plan executed to isolate the bottleneck causing slow file upload performance on a high-specification server. The findings demonstrate how to systematically rule out common internal server issues (CPU, Disk I/O) and pinpoint the external <strong>Network Bandwidth</strong> as the true limitation.</p>
<h2>I. Incident Details</h2>
<table>
<tbody>
<tr>
<th>
<p>Field</p>
</th>
<th>
<p>Detail</p>
</th>
</tr>
<tr>
<td>
<p><strong>Symptom</strong></p>
</td>
<td>
<p>Client reports "File uploads are very slow" to the hosted website.</p>
</td>
</tr>
<tr>
<td>
<p><strong>Environment</strong></p>
</td>
<td>
<p>Physical Server: Intel Xeon E5-2620 v4 (32 CPU threads)</p>
</td>
</tr>
<tr>
<td>
<p><strong>Operating System</strong></p>
</td>
<td>
<p>AlmaLinux 9.6 (cPanel/WHM)</p>
</td>
</tr>
<tr>
<td>
<p><strong>Hypothesis</strong></p>
</td>
<td>
<p>The slow upload is caused by an insufficient network bandwidth allocation or slow disk write speed.</p>
</td>
</tr>
</tbody>
</table>
<h2>II. Diagnostic Procedure (The 4-Point Plan)</h2>
<p>The diagnostic plan was executed from the root command line to measure the three critical components of file upload speed: <strong>Network Link Quality</strong>, <strong>Disk Write Speed</strong>, and <strong>External Bandwidth</strong>.</p>
<h3>1. Network Link Speed Check</h3>
<p><strong>Purpose:</strong> Verify the physical connection speed between the server and the immediate switch/router.</p>
<table>
<tbody>
<tr>
<th>
<p>Command</p>
</th>
<th>
<p>Metric</p>
</th>
<th>
<p>Expected Good Result</p>
</th>
</tr>
<tr>
<td>
<p><code>ethtool [interface]</code></p>
</td>
<td>
<p>Speed / Duplex</p>
</td>
<td>
<p>1000Mb/s (Full Duplex)</p>
</td>
</tr>
</tbody>
</table>
<h3>2. System Load and I/O Wait Check</h3>
<p><strong>Purpose:</strong> Determine if the server is currently under load or if disk I/O requests are waiting, slowing down the system.</p>
<table>
<tbody>
<tr>
<th>
<p>Command</p>
</th>
<th>
<p>Metric</p>
</th>
<th>
<p>Expected Good Result</p>
</th>
</tr>
<tr>
<td>
<p><code>iostat -x 1 5</code></p>
</td>
<td>
<p><code>%iowait</code></p>
</td>
<td>
<p>Less than 5%</p>
</td>
</tr>
</tbody>
</table>
<h3>3. Internal Disk I/O Speed Check</h3>
<p><strong>Purpose:</strong> Measure the raw, sustained speed at which the server can write large files to the local disk (where temporary uploads are stored).</p>
<table>
<tbody>
<tr>
<th>
<p>Command</p>
</th>
<th>
<p>Metric</p>
</th>
<th>
<p>Expected Good Result</p>
</th>
</tr>
<tr>
<td>
<p><code>dd if=/dev/zero of=/tmp/test_io.tmp bs=1M count=1024 conv=fdatasync</code></p>
</td>
<td>
<p>MB/s copied</p>
</td>
<td>
<p>&gt; 100 MB/s (HDD/RAID) or &gt; 300 MB/s (SSD/NVMe)</p>
</td>
</tr>
</tbody>
</table>
<h3>4. External Bandwidth Check (Upload/Download)</h3>
<p><strong>Purpose:</strong> Measure the actual network capacity that the ISP/datacenter provides to the server for receiving and sending data. This is the <strong>hard limit</strong> for uploads.</p>
<table>
<tbody>
<tr>
<th>
<p>Command</p>
</th>
<th>
<p>Metric</p>
</th>
<th>
<p>Expected Result (Depends on ISP Plan)</p>
</th>
</tr>
<tr>
<td>
<p><code>speedtest</code></p>
</td>
<td>
<p>Upload / Download (Mbps)</p>
</td>
<td>
<p>Should match the contracted bandwidth for the port.</p>
</td>
</tr>
</tbody>
</table>
<h2>III. Findings and Analysis</h2>
<table>
<tbody>
<tr>
<th>
<p>Diagnostic Area</p>
</th>
<th>
<p>Command Executed</p>
</th>
<th>
<p>Result</p>
</th>
<th>
<p>Analysis</p>
</th>
</tr>
<tr>
<td>
<p><strong>Network Link</strong></p>
</td>
<td>
<p><code>ethtool eno1</code></p>
</td>
<td>
<p><strong>Speed: 1000Mb/s (1 Gbps)</strong></p>
</td>
<td>
<p><strong>PASS.</strong> Physical connection is perfect.</p>
</td>
</tr>
<tr>
<td>
<p><strong>Disk I/O Speed</strong></p>
</td>
<td>
<p><code>dd...</code></p>
</td>
<td>
<p><strong>408 MB/s</strong></p>
</td>
<td>
<p><strong>PASS.</strong> Extremely fast disk writes (SSD performance). <strong>Disk is NOT the bottleneck.</strong></p>
</td>
</tr>
<tr>
<td>
<p><strong>System Load</strong></p>
</td>
<td>
<p><code>iostat</code></p>
</td>
<td>
<p><strong>%iowait: 0.00% - 0.07%</strong></p>
</td>
<td>
<p><strong>PASS.</strong> Server is idle. <strong>System load is NOT the bottleneck.</strong></p>
</td>
</tr>
<tr>
<td>
<p><strong>External Bandwidth</strong></p>
</td>
<td>
<p><code>speedtest</code></p>
</td>
<td>
<p><strong>Upload: 89.65 Mbit/s</strong></p>
</td>
<td>
<p><strong>FAILURE POINT.</strong> The external network port is capped at 89.65 Mbps (approx. <strong>11 MB/s</strong>).</p>
</td>
</tr>
</tbody>
</table>
<h3>Conclusion: Root Cause</h3>
<p>The slow file upload experienced by the customer is not a configuration issue within cPanel/PHP or a hardware failure (CPU/Disk I/O) on the server itself.</p>
<p>The <strong>Root Cause</strong> is the hard limit on the server's external internet connection, which caps the maximum transfer rate at <strong>89.65 Megabits per second (Mbps)</strong>.</p>
<p>From a customer's perspective, this means large file transfers will be throttled to approximately <strong>11 MB/s</strong>, regardless of how fast the server's internal components are.</p>
<h2>IV. Resolution and Mitigation</h2>
<p>The only way to resolve a network limitation confirmed by the <code>speedtest</code> is to address the infrastructure outside the server's operating system.</p>
<table>
<tbody>
<tr>
<th>
<p>Action Category</p>
</th>
<th>
<p>Action Item</p>
</th>
</tr>
<tr>
<td>
<p><strong>Resolution</strong></p>
</td>
<td>
<p>Contact the Datacenter/Hosting Provider to purchase a <strong>Bandwidth Upgrade</strong> (e.g., from 100 Mbps to 500 Mbps or 1 Gbps guaranteed).</p>
</td>
</tr>
<tr>
<td>
<p><strong>Mitigation</strong></p>
</td>
<td>
<p>Ensure all future server deployments that require high-speed uploads are provisioned with network ports guaranteed to provide the required throughput (e.g., 500 Mbps symmetrical link).</p>
</td>
</tr>
<tr>
<td>
<p><strong>Follow-up</strong></p>
</td>
<td>
<p>After the bandwidth upgrade is complete, re-run the <code>speedtest</code> command to confirm the new upload speed meets requirements.</p>
</td>
</tr>
</tbody>
</table>
<!-- Comments are visible in the HTML source only -->
