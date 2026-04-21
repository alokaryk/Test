https://olcsupport.office.com/
<h2 data-path-to-node="3">Step-by-Step Microsoft IP Whitelisting Process</h2>
<h3 data-path-to-node="4">Step 1: The Initial Request (SNDS/JMRP)</h3>
<p data-path-to-node="5">Before raising a ticket, you should ideally have the IP registered in Microsoft&rsquo;s monitoring portals.</p>
<ol start="1" data-path-to-node="6">
<li>
<p data-path-to-node="6,0,0"><strong data-path-to-node="6,0,0" data-index-in-node="0">Register for SNDS/JMRP:</strong> Sign up at <a class="ng-star-inserted" href="https://sendersupport.olc.protection.outlook.com/snds/" target="_blank" rel="noopener" data-hveid="0" data-ved="0CAAQ_4QMahgKEwjNhZKe2fuTAxUAAAAAHQAAAAAQ1wI">Outlook.com SNDS</a>. This allows you to see your IP's reputation and complaint rates.</p>
</li>
<li>
<p data-path-to-node="6,1,0"><strong data-path-to-node="6,1,0" data-index-in-node="0">Submit the Deliverability Form:</strong> This is the primary "ticket" mechanism.</p>
<ul data-path-to-node="6,1,1">
<li>
<p data-path-to-node="6,1,1,0,0"><strong data-path-to-node="6,1,1,0,0" data-index-in-node="0">Form Link:</strong> <a class="ng-star-inserted" href="https://www.google.com/search?q=https://support.microsoft.com/en-us/supportrequestform/8ad563e3-6019-09c3-ef8e-da9024092124" target="_blank" rel="noopener" data-hveid="0" data-ved="0CAAQ_4QMahgKEwjNhZKe2fuTAxUAAAAAHQAAAAAQ2AI">New Support Request</a></p>
</li>
<li>
<p data-path-to-node="6,1,1,1,0"><strong data-path-to-node="6,1,1,1,0" data-index-in-node="0">Required Info:</strong> Contact email, Sending IP, and a description of the bounce error (e.g., "550 5.7.1 Service unavailable; Client host [IP] blocked using Spamhaus").</p>
</li>
</ul>
</li>
</ol>
<h3 data-path-to-node="7">Step 2: The Automated Response (Wait 2&ndash;4 Hours)</h3>
<p data-path-to-node="8">Microsoft will send an automated email with a <strong data-path-to-node="8" data-index-in-node="46">Service Request Number</strong> (like the one you received: <code data-path-to-node="8" data-index-in-node="97">7098992114</code>).</p>
<ul data-path-to-node="9">
<li>
<p data-path-to-node="9,0,0"><strong data-path-to-node="9,0,0" data-index-in-node="0">Result A (Mitigation):</strong> They tell you the IP is qualified for mitigation (what you just received). You wait 48 hours for it to take effect.</p>
</li>
<li>
<p data-path-to-node="9,1,0"><strong data-path-to-node="9,1,0" data-index-in-node="0">Result B (Not Qualified):</strong> They tell you the IP does not qualify for mitigation. <strong data-path-to-node="9,1,0" data-index-in-node="80">Do not stop here.</strong></p>
</li>
</ul>
<h3 data-path-to-node="10">Step 3: Escalation (The "Advocate" Phase)</h3>
<p data-path-to-node="11">If you get a "Not Qualified" response, you must <strong data-path-to-node="11" data-index-in-node="48">reply directly to that email</strong>. This moves the ticket from an automated bot to a human "Deliverability Advocate."</p>
<ul data-path-to-node="12">
<li>
<p data-path-to-node="12,0,0"><strong data-path-to-node="12,0,0" data-index-in-node="0">Who to email:</strong> Reply to the email you received from <code data-path-to-node="12,0,0" data-index-in-node="51">deliv-inv@microsoft.com</code> (or the specific support address provided in the footer).</p>
</li>
<li>
<p data-path-to-node="12,1,0"><strong data-path-to-node="12,1,0" data-index-in-node="0">What to include:</strong> Provide technical proof of your 10/10 Mail-Tester score, verify that your SPF/DKIM/DMARC are passing, and state that the IP is not on any public blacklists.</p>
</li>
</ul>
<h3 data-path-to-node="13">Step 4: Final Escalation (Manual Review)</h3>
<p data-path-to-node="14">If the advocate still refuses mitigation:</p>
<ul data-path-to-node="15">
<li>
<p data-path-to-node="15,0,0"><strong data-path-to-node="15,0,0" data-index-in-node="0">Contact via Sender Support:</strong> Go to the <a class="ng-star-inserted" href="https://sendersupport.olc.protection.outlook.com/" target="_blank" rel="noopener" data-hveid="0" data-ved="0CAAQ_4QMahgKEwjNhZKe2fuTAxUAAAAAHQAAAAAQ2QI">Outlook Postmaster Tools</a> and use the "Contact Support" link.</p>
</li>
<li>
<p data-path-to-node="15,1,0"><strong data-path-to-node="15,1,0" data-index-in-node="0">Provide specific headers:</strong> Send a copy of the "Original Headers" of a failed email.</p>
</li>
<li>
<p data-path-to-node="15,2,0"><strong data-path-to-node="15,2,0" data-index-in-node="0">Verify PTR:</strong> Ensure your PTR record is <strong data-path-to-node="15,2,0" data-index-in-node="38">not</strong> generic. It must match your sending domain (e.g., <code data-path-to-node="15,2,0" data-index-in-node="92">mail.spacebyte.in</code>). Microsoft will often ignore requests for IPs with generic provider PTRs.</p>
</li>
</ul>
<hr data-path-to-node="16" />
<h2 data-path-to-node="17">Key Email Addresses &amp; Resources</h2>
<div class="horizontal-scroll-wrapper">
<table data-path-to-node="18">
<thead>
<tr>
<td><strong>Level</strong></td>
<td><strong>Contact Point</strong></td>
<td><strong>Purpose</strong></td>
</tr>
</thead>
<tbody>
<tr>
<td><span data-path-to-node="18,1,0,0"><strong data-path-to-node="18,1,0,0" data-index-in-node="0">Initial Ticket</strong></span></td>
<td><span data-path-to-node="18,1,1,0"><a class="ng-star-inserted" href="https://www.google.com/search?q=https://support.microsoft.com/en-us/supportrequestform/8ad563e3-6019-09c3-ef8e-da9024092124" target="_blank" rel="noopener" data-hveid="0" data-ved="0CAAQ_4QMahgKEwjNhZKe2fuTAxUAAAAAHQAAAAAQ2gI">Sender Support Form</a></span></td>
<td><span data-path-to-node="18,1,2,0">Standard entry point for IP whitelisting.</span></td>
</tr>
<tr>
<td><span data-path-to-node="18,2,0,0"><strong data-path-to-node="18,2,0,0" data-index-in-node="0">Advocate Review</strong></span></td>
<td><span data-path-to-node="18,2,1,0">Reply to <code data-path-to-node="18,2,1,0" data-index-in-node="9">deliv-inv@microsoft.com</code></span></td>
<td><span data-path-to-node="18,2,2,0">Escalating a "Not Qualified" bot response.</span></td>
</tr>
<tr>
<td><span data-path-to-node="18,3,0,0"><strong data-path-to-node="18,3,0,0" data-index-in-node="0">SNDS Portal</strong></span></td>
<td><span data-path-to-node="18,3,1,0"><a class="ng-star-inserted" href="https://sendersupport.olc.protection.outlook.com/snds/" target="_blank" rel="noopener" data-hveid="0" data-ved="0CAAQ_4QMahgKEwjNhZKe2fuTAxUAAAAAHQAAAAAQ2wI">SNDS Login</a></span></td>
<td><span data-path-to-node="18,3,2,0">To monitor real-time spam complaints and IP health.</span></td>
</tr>
<tr>
<td><span data-path-to-node="18,4,0,0"><strong data-path-to-node="18,4,0,0" data-index-in-node="0">Postmaster</strong></span></td>
<td><span data-path-to-node="18,4,1,0"><code data-path-to-node="18,4,1,0" data-index-in-node="0">postmaster@outlook.com</code></span></td>
<td><span data-path-to-node="18,4,2,0">Generally used for high-level infrastructure inquiries.</span></td>
</tr>
</tbody>
</table>
</div>
<hr data-path-to-node="19" />
<h3 data-path-to-node="20">Important Checklist for Alok (Cyfuture Team):</h3>
<ol start="1" data-path-to-node="21">
<li>
<p data-path-to-node="21,0,0"><strong data-path-to-node="21,0,0" data-index-in-node="0">Wait 48 Hours:</strong> Microsoft&rsquo;s "mitigation" is not instant. Do not send bulk tests immediately or the filter might reset.</p>
</li>
<li>
<p data-path-to-node="21,1,0"><strong data-path-to-node="21,1,0" data-index-in-node="0">Join JMRP:</strong> If you haven't already, join the <strong data-path-to-node="21,1,0" data-index-in-node="44">Junk Email Reporting Program</strong>. It sends you a copy of any email a Hotmail/Outlook user marks as "Spam." This helps you identify and remove "spammy" users from your server immediately.</p>
</li>
<li>
<p data-path-to-node="21,2,0"><strong data-path-to-node="21,2,0" data-index-in-node="0">Check IP 49.50.109.212 in 48 hours:</strong> Send a test mail specifically to a <code data-path-to-node="21,2,0" data-index-in-node="71">@hotmail.com</code> address and check the <code data-path-to-node="21,2,0" data-index-in-node="106">X-Microsoft-Antispam</code> header. If it says <code data-path-to-node="21,2,0" data-index-in-node="146">BCL:0</code> or <code data-path-to-node="21,2,0" data-index-in-node="155">BCL:1</code>, you are in the clear!</p>
</li>
</ol>
