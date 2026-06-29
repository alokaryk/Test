<div id="model-response-message-contentr_eb664892acd87b7d" class="markdown markdown-main-panel enable-luminous-fast-follows enable-updated-hr-color tutor-markdown-rendering" dir="ltr">
<p data-path-to-node="0">&nbsp;</p>
<h1 data-path-to-node="2">🛡️ Enterprise Linux Incident Response Guide: Miner Removal &amp; OS Recovery</h1>
<p data-path-to-node="3">This guide outlines how to handle an aggressive cryptocurrency miner infection that evades detection by breaking core OS administration commands (like <code data-path-to-node="3" data-index-in-node="151">top</code>, <code data-path-to-node="3" data-index-in-node="156">ps</code>, <code data-path-to-node="3" data-index-in-node="160">reboot</code>) and locking itself down via kernel-level file attributes.</p>
<h2 data-path-to-node="5">📋 Phase 1: Breaking the Memory Lock &amp; Identifying the Malicious Process</h2>
<p data-path-to-node="6">When an infection consumes all physical RAM via Hugepages, the system will throw <code data-path-to-node="6" data-index-in-node="81">Cannot allocate memory</code> or <code data-path-to-node="6" data-index-in-node="107">Killed</code> errors whenever you try to run diagnostics like <code data-path-to-node="6" data-index-in-node="162">top</code>.</p>
<h3 data-path-to-node="7">1. Reclaim Emergency Memory Safely</h3>
<p data-path-to-node="8">If the system cannot fork a new process, bypass user space and force the Linux kernel to release memory allocations directly:</p>
<div class="code-block ng-tns-c602925248-418 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwickPKW06uVAxUAAAAAHQAAAAAQnAY">
<div class="formatted-code-block-internal-container ng-tns-c602925248-418">
<div class="animated-opacity ng-tns-c602925248-418">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c602925248-418 ng-star-inserted"><span class="ng-tns-c602925248-418">Bash</span>
<div class="buttons ng-tns-c602925248-418 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c602925248-418"><code class="code-container formatted ng-tns-c602925248-418" data-test-id="code-content"><span class="hljs-comment"># Force the kernel core to invoke the OOM-killer to free up stuck memory chunks</span>
<span class="hljs-built_in">echo</span> f &gt; /proc/sysrq-trigger

<span class="hljs-comment"># Flush structural kernel file caches and slab allocations</span>
<span class="hljs-built_in">echo</span> 3 &gt; /proc/sys/vm/drop_caches

<span class="hljs-comment"># Reclaim memory locked up by the miner's Hugepages allocation</span>
<span class="hljs-built_in">echo</span> 0 &gt; /proc/sys/vm/nr_hugepages
</code></pre>
</div>
</div>
</div>
<h3 data-path-to-node="10">2. Bypass Hijacked/Broken System Diagnostics</h3>
<p data-path-to-node="11">If <code data-path-to-node="11" data-index-in-node="3">top</code> or standard <code data-path-to-node="11" data-index-in-node="19">ps</code> commands are corrupted or actively terminated by a malware loop, query the kernel filesystem directly to see what process is consuming memory:</p>
<div class="code-block ng-tns-c602925248-419 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwickPKW06uVAxUAAAAAHQAAAAAQnQY">
<div class="formatted-code-block-internal-container ng-tns-c602925248-419">
<div class="animated-opacity ng-tns-c602925248-419">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c602925248-419 ng-star-inserted"><span class="ng-tns-c602925248-419">Bash</span>
<div class="buttons ng-tns-c602925248-419 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c602925248-419"><code class="code-container formatted ng-tns-c602925248-419" data-test-id="code-content"><span class="hljs-comment"># Direct /proc scan using shell built-ins (bypasses broken system binaries)</span>
<span class="hljs-keyword">for</span> pid <span class="hljs-keyword">in</span> /proc/[0-9]*; <span class="hljs-keyword">do</span> 
  <span class="hljs-keyword">if</span> [ -f <span class="hljs-string">"<span class="hljs-variable">$pid</span>/stat"</span> ]; <span class="hljs-keyword">then</span> 
    <span class="hljs-built_in">echo</span> <span class="hljs-string">"<span class="hljs-subst">$(cat $pid/cmdline)</span> | PID: <span class="hljs-subst">$(basename $pid)</span> | RSS: <span class="hljs-subst">$(grep VmRSS $pid/status 2&gt;/dev/null)</span>"</span>
  <span class="hljs-keyword">fi</span>
<span class="hljs-keyword">done</span> | grep -v <span class="hljs-string">"RSS: $"</span> | sort -nk5 | tail -n 10
</code></pre>
</div>
</div>
</div>
<p data-path-to-node="13"><em data-path-to-node="13" data-index-in-node="0">Identify the rogue PID (e.g., matching strings like <code data-path-to-node="13" data-index-in-node="52">softirq</code>, <code data-path-to-node="13" data-index-in-node="61">rondo</code>, or anomalous scripts) and forcefully kill it:</em></p>
<div class="code-block ng-tns-c602925248-420 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwickPKW06uVAxUAAAAAHQAAAAAQngY">
<div class="formatted-code-block-internal-container ng-tns-c602925248-420">
<div class="animated-opacity ng-tns-c602925248-420">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c602925248-420 ng-star-inserted"><span class="ng-tns-c602925248-420">Bash</span>
<div class="buttons ng-tns-c602925248-420 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c602925248-420"><code class="code-container formatted ng-tns-c602925248-420" data-test-id="code-content"><span class="hljs-built_in">kill</span> -9 &lt;ROGUE_PID&gt;
</code></pre>
</div>
</div>
</div>
<h2 data-path-to-node="16">🛠️ Phase 2: Overriding Immutable File Locks Without <code data-path-to-node="16" data-index-in-node="53">chattr</code></h2>
<p data-path-to-node="17">Advanced botnets remove or replace the <code data-path-to-node="17" data-index-in-node="39">chattr</code> binary and apply the immutable (<code data-path-to-node="17" data-index-in-node="78">+i</code>) and append-only (<code data-path-to-node="17" data-index-in-node="99">+a</code>) flags to their own folders to prevent administrators from running <code data-path-to-node="17" data-index-in-node="169">rm -rf</code>.</p>
<h3 data-path-to-node="18">1. Unlocking Files via Python System Calls</h3>
<p data-path-to-node="19">If the malware has removed the <code data-path-to-node="19" data-index-in-node="31">chattr</code> command, use Python's native system calls to talk directly to the filesystem driver and drop the attribute locks:</p>
<div class="code-block ng-tns-c602925248-421 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwickPKW06uVAxUAAAAAHQAAAAAQnwY">
<div class="formatted-code-block-internal-container ng-tns-c602925248-421">
<div class="animated-opacity ng-tns-c602925248-421">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c602925248-421 ng-star-inserted"><span class="ng-tns-c602925248-421">Bash</span>
<div class="buttons ng-tns-c602925248-421 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c602925248-421"><code class="code-container formatted ng-tns-c602925248-421" data-test-id="code-content"><span class="hljs-comment"># Unlock the core miner execution layout</span>
python3 -c <span class="hljs-string">"import os, fcntl, struct; f = os.open('/bin/softirq', os.O_RDONLY); EXT2_IOC_SETFLAGS = 0x40086602; fcntl.ioctl(f, EXT2_IOC_SETFLAGS, struct.pack('i', 0)); os.close(f)"</span>

<span class="hljs-comment"># Unlock the persistent malware installation repository</span>
python3 -c <span class="hljs-string">"import os, fcntl, struct; f = os.open('/etc/rondo/rondo', os.O_RDONLY); EXT2_IOC_SETFLAGS = 0x40086602; fcntl.ioctl(f, EXT2_IOC_SETFLAGS, struct.pack('i', 0)); os.close(f)"</span>
</code></pre>
</div>
</div>
</div>
<h3 data-path-to-node="21">2. Immediate Obliteration</h3>
<p data-path-to-node="22">The second the flags drop, execute the deletion immediately before any monitoring script can restore them:</p>
<div class="code-block ng-tns-c602925248-422 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwickPKW06uVAxUAAAAAHQAAAAAQoAY">
<div class="formatted-code-block-internal-container ng-tns-c602925248-422">
<div class="animated-opacity ng-tns-c602925248-422">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c602925248-422 ng-star-inserted"><span class="ng-tns-c602925248-422">Bash</span>
<div class="buttons ng-tns-c602925248-422 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c602925248-422"><code class="code-container formatted ng-tns-c602925248-422" data-test-id="code-content">rm -f /bin/softirq
rm -rf /etc/rondo
</code></pre>
</div>
</div>
</div>
<h2 data-path-to-node="25">🕵️&zwj;♂️ Phase 3: Purging Automated Persistence Mechanisms</h2>
<p data-path-to-node="26">If you delete the malware binary without clearing its persistence engines, it will re-download or re-generate itself within minutes.</p>
<h3 data-path-to-node="27">1. Force-Unlock and Clear Cron Droppings</h3>
<div class="code-block ng-tns-c602925248-423 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwickPKW06uVAxUAAAAAHQAAAAAQoQY">
<div class="formatted-code-block-internal-container ng-tns-c602925248-423">
<div class="animated-opacity ng-tns-c602925248-423">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c602925248-423 ng-star-inserted"><span class="ng-tns-c602925248-423">Bash</span>
<div class="buttons ng-tns-c602925248-423 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c602925248-423"><code class="code-container formatted ng-tns-c602925248-423" data-test-id="code-content"><span class="hljs-comment"># Unlock and delete automated system cron files</span>
python3 -c <span class="hljs-string">"import os, fcntl, struct; f = os.open('/etc/cron.d/rondo', os.O_RDONLY); EXT2_IOC_SETFLAGS = 0x40086602; fcntl.ioctl(f, EXT2_IOC_SETFLAGS, struct.pack('i', 0)); os.close(f)"</span> 2&gt;/dev/null
rm -f /etc/cron.d/rondo
rm -f /etc/init.d/rondo

<span class="hljs-comment"># Unlock and completely wipe clean the root cron tracking spool</span>
python3 -c <span class="hljs-string">"import os, fcntl, struct; f = os.open('/var/spool/cron/crontabs/root', os.O_RDONLY); EXT2_IOC_SETFLAGS = 0x40086602; fcntl.ioctl(f, EXT2_IOC_SETFLAGS, struct.pack('i', 0)); os.close(f)"</span> 2&gt;/dev/null
&gt; /var/spool/cron/crontabs/root
</code></pre>
</div>
</div>
</div>
<h3 data-path-to-node="29">2. Scan Common Persistence Vector Paths</h3>
<p data-path-to-node="30">Search these files to ensure the malware didn't append an execution string to the end of legitimate system initialization steps:</p>
<ul data-path-to-node="31">
<li>
<p data-path-to-node="31,0,0"><code data-path-to-node="31,0,0" data-index-in-node="0">/etc/rc.local</code></p>
</li>
<li>
<p data-path-to-node="31,1,0"><code data-path-to-node="31,1,0" data-index-in-node="0">/etc/profile</code></p>
</li>
<li>
<p data-path-to-node="31,2,0"><code data-path-to-node="31,2,0" data-index-in-node="0">/root/.bashrc</code></p>
</li>
</ul>
<p data-path-to-node="32"><em data-path-to-node="32" data-index-in-node="0">If you see any unfamiliar string running a command out of <code data-path-to-node="32" data-index-in-node="58">/etc/</code> or a remote IP download (<code data-path-to-node="32" data-index-in-node="89">curl</code>/<code data-path-to-node="32" data-index-in-node="94">wget</code>), delete that line completely.</em></p>
<h2 data-path-to-node="34">🚀 Phase 4: Recovering the Damaged Operating System</h2>
<p data-path-to-node="35">Malware will often disrupt system library bindings (<code data-path-to-node="35" data-index-in-node="52">systemd</code>), causing interactive commands to break even after the miner is deleted.</p>
<h3 data-path-to-node="36">1. Reinstall Corrupted Core Interface Utilities</h3>
<p data-path-to-node="37">Run the following commands to reconstruct the broken symbolic links between the administration tools and the kernel init manager:</p>
<div class="code-block ng-tns-c602925248-424 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwickPKW06uVAxUAAAAAHQAAAAAQogY">
<div class="formatted-code-block-internal-container ng-tns-c602925248-424">
<div class="animated-opacity ng-tns-c602925248-424">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c602925248-424 ng-star-inserted"><span class="ng-tns-c602925248-424">Bash</span>
<div class="buttons ng-tns-c602925248-424 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c602925248-424"><code class="code-container formatted ng-tns-c602925248-424" data-test-id="code-content"><span class="hljs-comment"># Update local package manager tables</span>
apt-get update

<span class="hljs-comment"># Reinstall core storage filesystem flags utilities (restores chattr/lsattr)</span>
apt-get install --reinstall -y e2fsprogs

<span class="hljs-comment"># Reinstall base initialization system interface mappings (fixes broken top/reboot commands)</span>
apt-get install --reinstall -y systemd-sysv
</code></pre>
</div>
</div>
</div>
<h3 data-path-to-node="39">2. Perform a Synchronized System Reboot</h3>
<p data-path-to-node="40">Once the interface packages are cleanly reinstalled, cycle the server to clear memory structures and guarantee the malware is gone:</p>
<div class="code-block ng-tns-c602925248-425 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwickPKW06uVAxUAAAAAHQAAAAAQowY">
<div class="formatted-code-block-internal-container ng-tns-c602925248-425">
<div class="animated-opacity ng-tns-c602925248-425">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c602925248-425 ng-star-inserted"><span class="ng-tns-c602925248-425">Bash</span>
<div class="buttons ng-tns-c602925248-425 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c602925248-425"><code class="code-container formatted ng-tns-c602925248-425" data-test-id="code-content">reboot
</code></pre>
</div>
</div>
</div>
<p data-path-to-node="42"><em data-path-to-node="42" data-index-in-node="0">If a server is too broken to process the command, force a raw hardware reset directly via the kernel:</em></p>
<div class="code-block ng-tns-c602925248-426 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwickPKW06uVAxUAAAAAHQAAAAAQpAY">
<div class="formatted-code-block-internal-container ng-tns-c602925248-426">
<div class="animated-opacity ng-tns-c602925248-426">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c602925248-426 ng-star-inserted"><span class="ng-tns-c602925248-426">Bash</span>
<div class="buttons ng-tns-c602925248-426 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c602925248-426"><code class="code-container formatted ng-tns-c602925248-426" data-test-id="code-content"><span class="hljs-built_in">echo</span> 1 &gt; /proc/sys/kernel/sysrq
<span class="hljs-built_in">echo</span> b &gt; /proc/sysrq-trigger
</code></pre>
</div>
</div>
</div>
<h2 data-path-to-node="45">🔒 Phase 5: Closing Access Points to Prevent Reinfection</h2>
<p data-path-to-node="46">Miners typically gain root entry by abusing unpatched security vulnerabilities in internet-exposed Java/web stacks, or by brute-forcing compromised SSH keys.</p>
<ol start="1" data-path-to-node="47">
<li>
<p data-path-to-node="47,0,0"><strong data-path-to-node="47,0,0" data-index-in-node="0">Clear SSH Backdoors:</strong> Wipe out all unauthorized public keys introduced by the botnet:</p>
<div class="code-block ng-tns-c602925248-427 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwickPKW06uVAxUAAAAAHQAAAAAQpQY">
<div class="formatted-code-block-internal-container ng-tns-c602925248-427">
<div class="animated-opacity ng-tns-c602925248-427">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c602925248-427 ng-star-inserted"><span class="ng-tns-c602925248-427">Bash</span>
<div class="buttons ng-tns-c602925248-427 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c602925248-427"><code class="code-container formatted ng-tns-c602925248-427" data-test-id="code-content">&gt; /root/.ssh/authorized_keys
&gt; /home/mvdghes/.ssh/authorized_keys
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p data-path-to-node="47,1,0"><strong data-path-to-node="47,1,0" data-index-in-node="0">Reset System Credentials:</strong> Change administrative passwords immediately:</p>
<div class="code-block ng-tns-c602925248-428 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwickPKW06uVAxUAAAAAHQAAAAAQpgY">
<div class="formatted-code-block-internal-container ng-tns-c602925248-428">
<div class="animated-opacity ng-tns-c602925248-428">
<div class="code-block-decoration header-formatted gds-emphasized-body-m ng-tns-c602925248-428 ng-star-inserted"><span class="ng-tns-c602925248-428">Bash</span>
<div class="buttons ng-tns-c602925248-428 ng-star-inserted">&nbsp;</div>
</div>
<pre class="ng-tns-c602925248-428"><code class="code-container formatted ng-tns-c602925248-428" data-test-id="code-content">passwd root
passwd mvdghes
</code></pre>
</div>
</div>
</div>
</li>
<li>
<p data-path-to-node="47,2,0"><strong data-path-to-node="47,2,0" data-index-in-node="0">Audit Active Network Ports:</strong> Run <code data-path-to-node="47,2,0" data-index-in-node="32">netstat -ntpl</code> or <code data-path-to-node="47,2,0" data-index-in-node="49">ss -tupn</code> to verify that only authorized software (like HAProxy, Apache, or custom Java containers) are listening on external network networks.</p>
</li>
</ol>
</div>
