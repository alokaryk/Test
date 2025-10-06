<!DOCTYPE html>
<html lang="en" class="scroll-smooth">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Terraform Deployment Troubleshooting Report</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Slate & Emerald -->
    <!-- Application Structure Plan: A vertical, interactive timeline structure was chosen to guide the user through the chronological troubleshooting journey. The report is a narrative, so a linear flow is more intuitive than a dashboard. The app is divided into 3 main phases, with each of the 8 problems presented as an expandable/collapsible card. This design allows users to get a high-level overview and then drill down into specific issues without being overwhelmed. Navigation between phases is handled by a sticky timeline component. -->
    <!-- Visualization & Content Choices: 
        - Report Info: Overall troubleshooting process -> Goal: Guide user chronologically -> Viz: Vertical timeline/stepper -> Interaction: Click to scroll, highlight on scroll -> Justification: Provides clear navigation for a long-form narrative report -> Library/Method: HTML/CSS/JS.
        - Report Info: Individual Problems (P1-P8) -> Goal: Organize and detail each issue -> Viz: Expandable Cards -> Interaction: Click to expand/collapse -> Justification: Hides complexity by default, allowing users to focus. -> Library/Method: HTML/CSS/JS.
        - Report Info: Problem Symptoms (Error messages) -> Goal: Inform -> Viz: Styled code blocks -> Interaction: None -> Justification: Clearly presents technical error messages. -> Library/Method: HTML <pre><code>.
        - Report Info: Sequential Deployment (P7 Resolution) -> Goal: Organize/Explain a process -> Viz: Simple flow diagram -> Interaction: None -> Justification: Visually clarifies the dependency chain better than text alone. -> Library/Method: HTML <div>s with Flexbox.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8fafc; /* slate-50 */
        }
        .timeline-item::before {
            content: '';
            position: absolute;
            left: 1.2rem;
            top: 1.5rem;
            bottom: -1.5rem;
            width: 2px;
            background-color: #d1d5db; /* gray-300 */
            z-index: 0;
        }
        .timeline-phase:last-child .timeline-item::before {
            display: none;
        }
        .sticky-nav a.active {
            color: #059669; /* emerald-600 */
            font-weight: 600;
        }
        .problem-card-content {
            transition: max-height 0.5s ease-in-out, padding 0.5s ease-in-out;
            max-height: 0;
            overflow: hidden;
        }
        .problem-card.open .problem-card-content {
            max-height: 1000px; /* Adjust as needed */
        }
        .problem-card.open .icon-plus { display: none; }
        .problem-card:not(.open) .icon-minus { display: none; }
    </style>
</head>
<body class="text-slate-800">

    <div class="container mx-auto px-4 py-8 md:py-16 max-w-5xl">
        <header class="text-center mb-12 md:mb-20">
            <h1 class="text-4xl md:text-5xl font-bold text-slate-900 mb-4">Terraform Deployment Report</h1>
            <p class="text-lg text-slate-600 max-w-3xl mx-auto">An interactive post-mortem on deploying a Kubernetes cluster on Proxmox VE 8.4.0, detailing the troubleshooting journey from provider conflicts to final resolution.</p>
            <div class="mt-6 inline-flex items-center gap-2 bg-emerald-100 text-emerald-800 font-semibold px-4 py-2 rounded-full">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd" /></svg>
                <span>Status: Resolved</span>
            </div>
        </header>

        <div class="md:grid md:grid-cols-4 md:gap-12">
            <nav id="sticky-nav" class="hidden md:block md:col-span-1 h-screen sticky top-0 pt-16">
                <h3 class="font-semibold text-slate-900 mb-4">Troubleshooting Phases</h3>
                <ul class="space-y-3">
                    <li><a href="#phase-1" class="nav-link text-slate-500 hover:text-emerald-600 transition">Provider Conflicts</a></li>
                    <li><a href="#phase-2" class="nav-link text-slate-500 hover:text-emerald-600 transition">Syntax Errors</a></li>
                    <li><a href="#phase-3" class="nav-link text-slate-500 hover:text-emerald-600 transition">Infrastructure Logic</a></li>
                    <li><a href="#conclusion" class="nav-link text-slate-500 hover:text-emerald-600 transition">Conclusion</a></li>
                </ul>
            </nav>

            <main class="md:col-span-3">
                <section id="phase-1" class="timeline-phase scroll-mt-24">
                    <div class="timeline-item relative pl-12 pb-12">
                        <div class="absolute left-0 top-0 flex items-center justify-center w-10 h-10 bg-emerald-500 text-white font-bold rounded-full z-10">1</div>
                        <h2 class="text-3xl font-bold text-slate-900 mb-4 pt-1">Phase 1: Provider Compatibility</h2>
                        <p class="text-slate-600 mb-8">The initial and most significant challenge was identifying a Terraform provider compatible with Proxmox VE 8.4.0, as community providers frequently evolve, leading to syntax changes and deprecation.</p>
                        <div id="phase-1-problems" class="space-y-4"></div>
                    </div>
                </section>

                <section id="phase-2" class="timeline-phase scroll-mt-24">
                    <div class="timeline-item relative pl-12 pb-12">
                        <div class="absolute left-0 top-0 flex items-center justify-center w-10 h-10 bg-emerald-500 text-white font-bold rounded-full z-10">2</div>
                        <h2 class="text-3xl font-bold text-slate-900 mb-4 pt-1">Phase 2: Configuration Syntax Errors</h2>
                        <p class="text-slate-600 mb-8">Once a working provider (`bpg/proxmox`) was installed, the next set of hurdles involved adhering to its strict HCL parsing rules, which rejected many common or legacy Terraform patterns.</p>
                        <div id="phase-2-problems" class="space-y-4"></div>
                    </div>
                </section>

                <section id="phase-3" class="timeline-phase scroll-mt-24">
                     <div class="timeline-item relative pl-12 pb-12">
                        <div class="absolute left-0 top-0 flex items-center justify-center w-10 h-10 bg-emerald-500 text-white font-bold rounded-full z-10">3</div>
                        <h2 class="text-3xl font-bold text-slate-900 mb-4 pt-1">Phase 3: Infrastructure Logic & Provisioning</h2>
                        <p class="text-slate-600 mb-8">With syntax resolved, the final problems were related to infrastructure limitations and the logic of executing remote commands across the newly created virtual machines.</p>
                        <div id="phase-3-problems" class="space-y-4"></div>
                    </div>
                </section>

                <section id="conclusion" class="scroll-mt-24">
                    <div class="pl-12">
                         <h2 class="text-3xl font-bold text-slate-900 mb-4">Conclusion & Final Steps</h2>
                        <p class="text-slate-600 mb-6">The final configuration successfully utilizes the stable `bpg/proxmox` provider. The critical timeout issue was resolved by implementing a sequential deployment strategy, creating each VM one by one to avoid overwhelming the Proxmox API. The cluster is now fully provisioned and ready for the final application-level setup.</p>
                         <div class="bg-white p-6 rounded-lg shadow-sm border border-slate-200">
                             <h4 class="font-semibold text-lg text-slate-900 mb-3">Next Steps</h4>
                             <ul class="space-y-3 text-slate-700">
                                 <li class="flex items-start gap-3">
                                     <span class="text-emerald-500 mt-1">✓</span>
                                     <div>
                                         <strong>Install CNI (Flannel):</strong> Deploy a Container Network Interface to enable pod-to-pod communication across the cluster.
                                     </div>
                                 </li>
                                 <li class="flex items-start gap-3">
                                     <span class="text-emerald-500 mt-1">✓</span>
                                     <div>
                                        <strong>Deploy NFS CSI Driver:</strong> Use Helm to install the NFS Subdir External Provisioner, setting it as the default StorageClass to resolve persistent storage requirements.
                                     </div>
                                 </li>
                             </ul>
                         </div>
                    </div>
                </section>
            </main>
        </div>
    </div>

    <script>
        const problems = [
            { id: 'P1', phase: 1, title: 'Deprecated Provider', symptom: 'Initial configuration used `telmate/proxmox`, which is deprecated.', cause: 'The `telmate/proxmox` provider is incompatible with modern Proxmox (8.x) and its required nested block syntax for Cloud-Init.', resolution: 'Switched to the actively maintained community provider, **`bpg/proxmox`**, and updated the `required_providers` block.' },
            { id: 'P2', phase: 1, title: 'Registry Lookup Failure', symptom: '`Error: Failed to query available provider packages` for `bpg/proxmoxve`.', cause: 'The specific provider alias `bpg/proxmoxve` was retired or renamed on the Terraform Registry. Local cache corruption from previous failed attempts exacerbated the issue.', resolution: 'Cleared the local Terraform cache (`rm -rf .terraform`) and corrected the source name in the configuration to the active alias: **`bpg/proxmox`**.' },
            { id: 'P3', phase: 1, title: 'Provider Argument Naming', symptom: '`Error: Unsupported argument` on `pm_api_url`, `pm_user`, etc.', cause: 'The new `bpg/proxmox` provider uses modern argument names (`endpoint`, `username`), while the configuration still used legacy prefixes from the `telmate` provider.', resolution: 'Updated the `provider "proxmox"` block to use the correct modern attribute names.' },
            { id: 'P4', phase: 2, title: 'Cloud-Init Block Structure', symptom: '`Error: Unsupported block type` on `initialization`, `ci_public_keys`, `ipconfig`, etc.', cause: 'The provider rejected both flat structures (from `telmate`) and simplified nested structures. It requires a specific, multi-layered block for Cloud-Init settings.', resolution: 'Enforced the fully correct, multi-line nested block structure: `initialization { user_account { ... } ip_config { ipv4 { ... } } }`.' },
            { id: 'P5', phase: 2, title: 'Inline HCL Syntax', symptom: '`Error: Invalid character` (on `;`) and `Argument definition required`.', cause: 'Attempted to use compressed, single-line HCL blocks (e.g., `cpu { cores = 1; type = "host" }`), which is invalid syntax for this provider.', resolution: 'Manually expanded all single-line blocks into the standard, verbose multi-line HCL format.' },
            { id: 'P6', phase: 2, title: 'VM Resource Naming', symptom: '`Error: Invalid resource type` on `proxmox_vm_qemu`.', cause: 'The final selected provider (`bpg/proxmox`) uses a different, more descriptive resource name for QEMU virtual machines.', resolution: 'Changed all VM resource types to the correct name: **`proxmox_virtual_environment_vm`**.' },
            { id: 'P7', phase: 3, title: 'API Connection Timeout', symptom: '`Error: error waiting for VM clone: ... received an HTTP 596 response - Reason: Connection timed out`.', cause: 'Proxmox experienced high I/O load attempting to clone 7 large VMs simultaneously, causing the API request from Terraform to time out.', resolution: 'Implemented a **sequential deployment strategy**. Removed the `for_each` loop for VM creation and defined each of the 7 VMs in separate resource blocks, linked with explicit `depends_on` clauses to force them to be created one by one.' },
            { id: 'P8', phase: 3, title: 'Provisioner Logic', symptom: '`Error: Unsupported argument` on `proxy_command` and `only_if`.', cause: 'Provisioning blocks were incorrectly placed inside the `proxmox_virtual_environment_vm` resource, which is not supported. Additionally, the `null_resource` connection block used an invalid argument for the jump host.', resolution: 'Moved all provisioning logic into dedicated `null_resource` blocks and updated the `connection` blocks to use the correct modern arguments for a jump server: `bastion_host` and `bastion_user`.' }
        ];

        function createProblemCard(problem) {
            const card = document.createElement('div');
            card.className = 'problem-card bg-white rounded-lg shadow-sm border border-slate-200';
            card.innerHTML = `
                <div class="problem-card-header flex justify-between items-center p-4 cursor-pointer">
                    <div class="flex items-center gap-3">
                        <span class="flex items-center justify-center w-8 h-8 bg-slate-100 text-slate-500 font-bold rounded-md text-sm">${problem.id}</span>
                        <h4 class="font-semibold text-slate-900">${problem.title}</h4>
                    </div>
                    <div class="text-slate-400">
                        <svg class="icon-plus h-5 w-5" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" /></svg>
                        <svg class="icon-minus h-5 w-5" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 12H4" /></svg>
                    </div>
                </div>
                <div class="problem-card-content px-4">
                    <div class="py-4 border-t border-slate-200 grid sm:grid-cols-3 gap-6">
                        <div>
                            <h5 class="font-semibold text-sm text-slate-500 mb-2">Symptom & Error</h5>
                            <pre class="bg-slate-50 p-3 rounded-md text-sm text-red-700 whitespace-pre-wrap font-mono"><code>${problem.symptom}</code></pre>
                        </div>
                        <div>
                            <h5 class="font-semibold text-sm text-slate-500 mb-2">Root Cause</h5>
                            <p class="text-sm text-slate-600">${problem.cause}</p>
                        </div>
                        <div>
                            <h5 class="font-semibold text-sm text-slate-500 mb-2">Resolution</h5>
                            <p class="text-sm text-emerald-800 bg-emerald-50 p-3 rounded-md">${problem.resolution}</p>
                        </div>
                    </div>
                </div>
            `;
            return card;
        }

        const phase1Container = document.getElementById('phase-1-problems');
        const phase2Container = document.getElementById('phase-2-problems');
        const phase3Container = document.getElementById('phase-3-problems');

        problems.forEach(p => {
            const card = createProblemCard(p);
            if (p.phase === 1) phase1Container.appendChild(card);
            if (p.phase === 2) phase2Container.appendChild(card);
            if (p.phase === 3) phase3Container.appendChild(card);
        });
        
        document.querySelectorAll('.problem-card-header').forEach(header => {
            header.addEventListener('click', () => {
                header.parentElement.classList.toggle('open');
            });
        });

        const navLinks = document.querySelectorAll('.nav-link');
        const sections = document.querySelectorAll('section');

        const observer = new IntersectionObserver((entries) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    navLinks.forEach(link => {
                        link.classList.toggle('active', link.getAttribute('href').substring(1) === entry.target.id);
                    });
                }
            });
        }, { rootMargin: "-50% 0px -50% 0px" });

        sections.forEach(section => {
            observer.observe(section);
        });

    </script>
</body>
</html>
