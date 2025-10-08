# Network Throttling — RCA (Root Cause Analysis) Template

> Purpose: A reusable, step-by-step RCA-style note to investigate, document, and resolve suspected network throttling (bandwidth limits, traffic shaping, or abnormal latency). Use this as a checklist and as the basis for an incident report.

---

## 1. Title & Metadata
- **Incident title:** Network performance degradation / suspected throttling
- **Server/Host/VM:**
- **IP / Interface(s) observed:**
- **Date / Time (start):**
- **Date / Time (investigated):**
- **Author / Investigator:**
- **Severity:** (P1 / P2 / P3)
- **Stakeholders / Escalation contacts:**

---

## 2. Executive Summary (1–2 sentences)
Summarise what was observed, the impact, and the final conclusion (e.g., "Observed slow outbound transfers; investigation found no OS-level shaping; upstream datacenter/ISP asymmetric policy suspected"). Keep this short for management.

---

## 3. Scope & Impact
- **Services affected:** (web, mail, backups, FTP, app API, etc.)
- **Users impacted / Regions:**
- **Observable symptoms:** (slow downloads, slow uploads, intermittent timeouts, increased latency, retransmissions)
- **Business impact:** (tickets, SLA breach, revenue/time loss)

---

## 4. Timeline of Events (fill as you go)
| Time (UTC+?) | Action / Event | Command / Evidence Reference | Result |
|---:|---|---|---|
| 2025-10-08 12:00 | User reports slow outbound uploads | Ticket #12345 | Uploads failing > 2 Mbps |

**Notes:** Record exact timestamps and every command you ran with output references (logs, screenshots, pcap files).

---

## 5. Evidence Collected (how to collect)
Collect and attach (or paste references to) outputs and logs. Keep originals.

**Suggested commands (run as root or with sudo):**

### a) Inspect traffic control (kernel shaping)
```bash
# show qdisc and classes
sudo tc qdisc show
sudo tc -s qdisc ls dev <iface>
sudo tc class show dev <iface>
sudo tc filter show dev <iface>
```
**What to look for:** `tbf`, `htb`, `netem`, `rate`, or explicit classes/filters that limit bandwidth.


### b) Check NIC link / speed / duplex
```bash
sudo ethtool <iface>
# quick grep
sudo ethtool <iface> | grep -E "Speed|Duplex|Link"
```
**What to look for:** `Speed` lower than expected, `Duplex: Half`, `Link detected: no`.


### c) Interface stats (errors/drops)
```bash
ip -s link show <iface>
netstat -i
ifconfig <iface>
```
**What to look for:** non-zero `errors`, `dropped`, `overrun` values.


### d) Real-time bandwidth and active flows
```bash
# interactive
sudo iftop -i <iface>
sudo nload <iface>
sudo bmon -p <iface>
```
**What to look for:** single flow pegged at a fixed value, many retransmissions, or sudden identical rate caps.


### e) Historical metrics
```bash
# sysstat/sar
sar -n DEV 1 10
sar -n DEV -f /var/log/sysstat/sa<day>
```
**What to look for:** interface util spikes, long-term saturation.


### f) External throughput tests
```bash
# speedtest (may be blocked) or curl wrapper
speedtest-cli
curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python3 -
# iperf3 to a known server
iperf3 -c <server>
# or host your own server
# on remote: iperf3 -s
# on local: iperf3 -c <remote-ip>
```
**What to look for:** consistent low upload vs download -> indicates upstream shaping or asymmetric link.


### g) Path and latency analysis
```bash
mtr -r -c 100 <destination>
traceroute -n <destination>
ping -c 50 <destination>
```
**What to look for:** packet loss at a certain hop, high and variable latency beyond the first couple hops.


### h) Packet capture (when needed)
```bash
sudo tcpdump -i <iface> -w /tmp/capture.pcap
# if storage limited, capture with filters (port 80 or host <ip>)
```
Use Wireshark/tshark to analyze retransmissions, dup ACKs, RSTs, CWND issues.


### i) Application & OS checks
```bash
# sockets and connections
ss -tunap | head -n 40
# process network usage
sudo ss -p -o state established
# system I/O
iostat -x 1 3
vmstat 1 5
# check CPU and IOWAIT
sar -u 1 5
```
**What to look for:** CPU/IO wait causing perceived network slowness; processes hitting limits.


### j) Firewall / NAT / Rate Limiters
```bash
# iptables / nftables
sudo iptables -L -n -v
sudo nft list ruleset
# cloud/dc ACL or firewall rules (check provider UI)
```
**What to look for:** packet/byte counters showing policing; explicit rate-limiting rules.


## 6. Interpretation Guide (how to read outputs)
- **`tc` shows `tbf`, `htb`, `netem`** → *local shaping* present. Locate class/filter and remove/adjust.
- **`fq_codel` only** → default latency control; *not* throttling.
- **`ethtool` speed less than expected (e.g., 100Mb vs 1Gb)** → check switch port, VM settings, or host vSwitch.
- **High `RX/TX errors` or `dropped`** → potential NIC/sfp/switch issue or driver bug.
- **`iftop` shows many flows but each limited to same small rate** → could be per-flow shaping (router/ISP) or congestion control behavior.
- **Speedtest shows asymmetry (download >> upload)** → upstream shaping or asymmetric plan.
- **`mtr` shows loss starting at provider hop** → escalate to ISP/Datacenter with hop evidence.
- **`tcpdump` shows lots of retransmits & dup acks** → packet loss on path → network device or upstream issue.

---

## 7. Common Root Causes (with examples)
1. **Local OS/Kernel shaping** — `tc` rules, `netem`, or systemd-networkd config.
2. **Hypervisor / vSwitch limits** — cloud provider applied QoS, virtual NIC bandwidth limits.
3. **Datacenter / ISP shaping** — upstream policy enforcing download/upload limits.
4. **Physical link negotiation** — switch port at 100Mb; NIC at full but upstream at lower speed.
5. **Firewall / NAT device rate-limiting** — provider or edge device.
6. **Application-layer congestion** — web server threads/process limits, upload processing slow.
7. **DDoS / noisy neighbors** — bandwidth hogging by other tenants.
8. **Hardware faults** — failing SFP, cable, or port errors.

---

## 8. Corrective Actions (by root cause)
- **If `tc` rules found:**
  - Inspect with `tc -s qdisc ls dev <iface>` and remove/mask rule after assessing impact:
    ```bash
    sudo tc qdisc del dev <iface> root
    ```
  - *Caution:* removing qdiscs can affect QoS; document and plan rollback.

- **If hypervisor limit discovered:**
  - Open a ticket with provider; request removal/increase of vNIC bandwidth.

- **If ISP/datacenter shaping suspected:**
  - Provide `mtr`/`traceroute`/speedtest evidence to provider; request clarification.

- **If link negotiation issue:**
  - Force `ethtool -s <iface> speed 1000 duplex full autoneg off` *only after confirming partner switch supports it*.

- **If firewall/NAT rate-limiting:**
  - Adjust or remove rate-limit rules in provider UI or edge device.

- **If application bottleneck:**
  - Profile app (threads, DB queries), scale horizontally, or tune worker counts.

- **If packet loss / hardware error:**
  - Replace cable/SFP, move VM to different host, or update NIC driver.

---

## 9. Validation & Post-fix Checks
After each remediation, validate with:
- `iperf3` (controlled remote server)
- `speedtest` (if permitted)
- `iftop`, `nload` to watch real-time behavior
- `sar -n DEV` for historical confirmation
- Re-run `tc`, `ethtool`, and `ip -s link` to confirm changes

Document before/after results with timestamps.

---

## 10. Preventive Actions & Monitoring
- Add interface utilization alerts (thresholds: 70%, 85%, 95%).
- Monitor packet drops/errors and set alerts on non-zero counts.
- Keep a network diagram and inventory of expected link speeds.
- Document standard `tc` rules used by the platform and keep change history (git).
- Periodically run synthetic tests (iperf/speedtest) and log results.

---

## 11. Communication & Postmortem Template
- **Summary of incident**
- **Root cause**
- **Corrective action taken**
- **Mitigation plan**
- **Next steps & owners**
- **ETA for permanent fix**

---

## 12. Appendix — Quick Command Cheat Sheet
```bash
# traffic control
sudo tc qdisc show
sudo tc -s qdisc ls dev <iface>
# link
sudo ethtool <iface>
ip -s link show <iface>
# real-time
sudo iftop -i <iface>
nload <iface>
# historical
sar -n DEV 1 10
# external test
iperf3 -c <server>
# traceroute/mtr
mtr -r -c 100 <ip>
# capture
sudo tcpdump -i <iface> -w /tmp/cap.pcap
# sockets
ss -tunap
# firewall
sudo iptables -L -n -v
sudo nft list ruleset
```

---

## 13. Example Findings (fill in for your incident)
- **Observed:** Outbound upload rates ~3 Mbps vs expected >100 Mbps
- **Evidence:** `speedtest` output (attach), `iperf3` (attach), `tc qdisc show` returned only `fq_codel`, `ethtool` 10Gb/s
- **Root cause:** Upstream provider asymmetric policy limiting upload
- **Resolution:** Escalated to provider, temporary workaround implemented (off-peak bulk transfer), planned change: purchase symmetric uplink or dedicated uplink

---

*End of template — modify to fit your environment and team process.*

