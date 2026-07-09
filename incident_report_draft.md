# Incident Response Report

**Privileged and Confidential**

---

**Affected Organization:** Northbridge Financial Services
**Incident Name:** Workstation Compromise via Drive-By Exploit Kit and Xtreme RAT
**Incident Number:** IR-2026-001
**Date Published:** 2026-07-01
**Investigation Performed By:** Sentinel Ridge Cybersecurity
**Classification:** Privileged and Confidential

![Attack overview](capturas/fig-01.png)

*Figure 1. High-level attack diagram: infection via Exploit Kit, RAT installation, internal scanning, and exfiltration.*

---

## Table of Contents

1. Executive Summary
2. Timeline
3. Findings
4. Investigative Questions
5. Systems / People Involved
6. Indicators of Compromise
7. Evidence Collected
8. Remediation
9. Recommendations
10. Lessons Learned
11. Appendices
12. References

---

## 1. Executive Summary

Northbridge Financial Services engaged Sentinel Ridge Cybersecurity to investigate the compromise of an employee Windows workstation. The incident was discovered after security staff noticed anomalous behavior on the host and captured a live memory image and a full forensic disk image for analysis. The engagement was sponsored by Northbridge Financial Services' executive leadership team.

The investigation determined that the workstation was infected with **Xtreme RAT**, a commodity remote-access trojan that gives an attacker full remote control of a victim machine. The infection began when the user browsed to a legitimate but compromised website (`blog.mycompany.ex`). That site had been tampered with to silently redirect the browser to an **Exploit Kit** hosted at `blog.mysportclub.ex`. Because the workstation ran an outdated web browser and browser plugin, the Exploit Kit succeeded in running code without any user interaction beyond visiting the page — a "drive-by" attack. The malware installed itself, established persistence so it would survive reboots, and then downloaded a toolkit of hacking utilities. Over the following two hours the attacker profiled the machine, scanned the internal network, attempted to guess passwords for other systems, connected to at least one internal host, and transferred data using file-copy tools — behavior consistent with an active, hands-on-keyboard intruder attempting to move deeper into the network.

The response goal was to establish how the machine was compromised, what the attacker did, which systems and accounts were affected, and which indicators the organization should hunt for enterprise-wide. Based on the evidence, the affected workstation was isolated and reimaged, associated accounts were reset, malicious files and persistence were removed, and the attacker's infrastructure was blocked. The forensic activity captured in the evidence spans **2016-08-16 12:54 UTC (system boot) to approximately 14:50 UTC (last observed attacker tool execution)**. The analysis engagement itself was conducted from **[Engagement start date — administrative placeholder]** to **[Engagement end date — administrative placeholder]**.

---

## 2. Timeline

All times are UTC. The host clock was configured for UTC+2 (local time = UTC + 2 hours), as confirmed by registry evidence (13:03:10 UTC = 15:03:10 local).

| Date/Time (UTC) | Event | Source of Evidence |
|---|---|---|
| 2016-08-16 12:54:24 | Workstation booted | Volatility `pslist` (System process start) |
| 2016-08-16 13:02:46 | User browses to compromised site `blog.mycompany.ex` | Firefox history |
| 2016-08-16 13:02:57 | `svchost.exe` (Xtreme RAT) created in `%TEMP%`; `Run`/`RunOnce` keys modified | Filesystem timestamps; RegRipper `regtime` |
| 2016-08-16 13:03:03–13:03:04 | `update.exe` (Xtreme RAT) executed | Prefetch; `pslist` |
| 2016-08-16 13:03:10 | Persistence subkey `GhCtxq8t` created; `update.exe` first execution | RegRipper / WRR |
| 2016-08-16 13:03:16 | Firefox crash report tied to Flash plugin | Application logs |
| 2016-08-16 13:07:36 | `update.exe` spawns first `cmd.exe`; `whoami`/`ipconfig` run 13:08–13:10 | `pslist`; Prefetch |
| 2016-08-16 13:10:03 / 13:10:13 | `54948tp.exe` created in `%TEMP%` and executed (downloads toolkit) | Filesystem; Prefetch |
| 2016-08-16 13:14:47 | `%APPDATA%\EpUpdate` toolkit directory created; mimikatz and BPD run 13:14:48 | Filesystem; Prefetch |
| 2016-08-16 13:34:25–13:34:51 | Batch of local system-profiling commands; `sysinfo.txt` written | Prefetch; SystemProfile artifacts |
| 2016-08-16 ~13:56–13:59:36 | Nmap scans of 192.168.5.1, .10, .15 | Prefetch; `netscan/` XML |
| 2016-08-16 14:02:04–14:04:43 | Event ID 4798 group-membership enumeration; Hydra executed 14:04:44 | Windows Event Log; Prefetch |
| 2016-08-16 14:10:49–14:23:31 | `plink.exe` executed six times; PuTTY `SSHHostKeys` for 192.168.5.10 written 14:11:26 | Prefetch; RegRipper / WRR |
| 2016-08-16 14:47:12–14:50:09 | `pscp.exe` executed three times (data transfer) | Prefetch |

---

## 3. Findings

An employee workstation belonging to Northbridge Financial Services was fully compromised by an attacker using the **Xtreme RAT** remote-access trojan. The compromise was not the result of the employee downloading an obviously malicious file; instead, the user simply visited a trusted website that had been secretly modified by attackers. That website redirected the browser, in the background, to an **Exploit Kit** that took advantage of outdated software on the workstation (Mozilla Firefox 33.0.3 and Adobe Flash 18.0.0.194) to install malware automatically. This is a classic drive-by compromise and highlights the risk of running unpatched browser software.

Once installed, the malware made itself persistent so it would run every time the user logged in, and it downloaded a full attacker toolkit including credential-stealing tools (mimikatz, a browser-password dumper, Pwdump), a network scanner (Nmap), a password-guessing tool (THC Hydra), and remote-access/file-transfer tools (plink and pscp from PuTTY). Evidence shows the attacker used these tools in a logical progression: they first learned about the machine and the internal network, scanned three internal hosts (192.168.5.1, 192.168.5.10, 192.168.5.15), attempted to crack passwords, connected to internal host 192.168.5.10, and then ran file-transfer tools consistent with moving data in or out of the environment.

The affected workstation and the user account that was logged in at the time are considered compromised, and internal host 192.168.5.10 is a strong candidate for follow-on compromise and requires its own examination. The strongest indicators for enterprise-wide hunting are the malicious file paths (`%APPDATA%\HostData\update.exe`, `%TEMP%\svchost.exe`, `%APPDATA%\EpUpdate\`), the persistence registry key `GhCtxq8t`, the attacker domains `blog.mycompany.ex` and `blog.mysportclub.ex`, and the associated IP address 151.80.137.2. Evidence used to reach these conclusions included a memory image, a full disk image, Prefetch files, Windows Event Logs, and registry hives. In response, the workstation was isolated and reimaged, affected credentials were reset, the WDigest cleartext-credential change was reverted, persistence and malicious files were removed, and attacker infrastructure was blocked at the network perimeter.

---

## 4. Investigative Questions

### Question 1: How was the workstation initially compromised?

**Answer:** The workstation was compromised through a drive-by download. The user visited `blog.mycompany.ex`, a legitimate site that had been injected with a hidden `<iframe>` pointing to an Exploit Kit at `http://blog.mysportclub.ex/wp-content/uploads/hk/task/opspy/index.php`. The Exploit Kit served multiple exploits targeting the workstation's outdated browser and Flash plugin, one of which downloaded and executed the initial payload.
**Supporting Evidence:** Firefox history showing the visit to `blog.mycompany.ex` at 13:02:46; the cached `blog.mycompany.ex.htm` containing the iframe injection; the pattern of similarly named `.html` files downloaded from `blog.mysportclub.ex`; and a Firefox/Flash crash report at 13:03:16 (Figures 17–22).

### Question 2: What malware was installed, and where does it live on disk?

**Answer:** The system was infected with **Xtreme RAT**. Its code was found in memory in three processes (`svchost.exe` PID 4888, `explorer.exe` PID 4872, `update.exe` PID 5172) and confirmed on disk by antivirus. The primary persistent copy resides at `%APPDATA%\HostData\update.exe`, with an additional copy at `%TEMP%\svchost.exe`.
**Supporting Evidence:** Volatility `yarascan` hits for Xtreme/xtreme_rat/xtremrat rules; UPX-packing overlap on the same processes; ClamAV confirmation of Xtreme RAT in `update.exe`; memory-analysis summary (Figures 5–7, 16).

### Question 3: Did the attacker establish persistence?

**Answer:** Yes. At 13:02:57 the `Run` and `RunOnce` subkeys of `NTUSER.DAT` were modified, and at 13:03:10 a subkey named `GhCtxq8t` was created. WRR analysis showed `GhCtxq8t` is associated with `update.exe`, and its `FirstExecution` value confirms `update.exe` first ran at 13:03:10 UTC (15:03:10 local). This ensures the RAT restarts at user logon.
**Supporting Evidence:** RegRipper `regtime` timeline and WRR inspection of `NTUSER.DAT` (Figures 34–35).

### Question 4: What tools did the attacker deploy, and how did they arrive?

**Answer:** A downloader, `54948tp.exe` (a py2exe-compiled Python program), was created at 13:10:03 and executed at 13:10:13. It fetched `data_32.bin` from `http://blog.mysportclub.ex/wp-content/uploads/hk/files/data_32.bin`, decrypted and unpacked it into `%APPDATA%\EpUpdate\`. That directory contained mimikatz, BrowserPasswordDump, NirCmd, Nmap, Pwdump, plink/pscp (SSH), THC Hydra, a `passwords.txt` wordlist, and `wdigest.reg` (which enables cleartext credential caching via `UseLogonCredential`).
**Supporting Evidence:** Decompiled `54948tp.exe`/`3568226350.exe` (`DOWNLOAD_URL`, `get_toolz`); `EpUpdate` directory listing created at 13:14:47; Prefetch execution of mimikatz/BPD at 13:14:48 (Figures 23–24, 29).

### Question 5: Was there evidence of internal reconnaissance, lateral movement, or exfiltration?

**Answer:** Yes to all three. The attacker profiled the local system (13:34:25–13:34:51, plus earlier `whoami`/`ipconfig`), scanned three internal hosts (192.168.5.1, 192.168.5.10, 192.168.5.15) with Nmap 7.12 around 13:56–13:59, ran THC Hydra at 14:04:44 (correlated with Event ID 4798 at 14:02–14:04), then executed `plink.exe` six times (14:10:49–14:23:31) and `pscp.exe` three times (14:47:12–14:50:09). A PuTTY `SSHHostKeys` entry for 192.168.5.10 was written at 14:11:26, indicating an SSH connection was made to that host.
**Supporting Evidence:** Prefetch execution records; `netscan/` Nmap XML; Windows Event Log 4798; RegRipper PuTTY `SSHHostKeys` (Figures 26, 30–33, 36–37).

### Question 6: What vulnerabilities allowed the exploit to succeed, and which IOCs should be hunted enterprise-wide?

**Answer:** The workstation ran outdated **Mozilla Firefox 33.0.3** and **Adobe Flash Plugin 18.0.0.194**; the Exploit Kit code referenced multiple exploits (including **CVE-2012-3993**). Enterprise-wide, defenders should hunt for the file paths `%APPDATA%\HostData\update.exe`, `%TEMP%\svchost.exe`, and `%APPDATA%\EpUpdate\`; the registry key `GhCtxq8t`; the domains `blog.mycompany.ex` and `blog.mysportclub.ex`; the IP 151.80.137.2; and the filenames `update.exe`, `54948tp.exe`, and `3568226350.exe`.
**Supporting Evidence:** Registry Uninstall information; ClamAV detection of CVE-2012-3993 exploit code in the Firefox cache; consolidated IOC table (Figures 38–40).

---

## 5. Systems / People Involved

| Hostname / Person | IP Address | Role / Purpose | Date/Time of Pertinent Evidence (UTC) | Compromise Found |
|---|---|---|---|---|
| Affected workstation (hostname not identified in the provided evidence) | Not identified in the provided evidence | User endpoint; source of memory and disk images | 2016-08-16 12:54–14:50 | Yes — Xtreme RAT infection, persistence, attacker tooling |
| Logged-in user account (username not identified in the provided evidence) | N/A | Interactive user whose `NTUSER.DAT` was modified for persistence | 2016-08-16 13:02:57 onward | Yes — session used for persistence and tool execution |
| Internal host | 192.168.5.10 | Internal system targeted by Nmap and SSH (plink) | 2016-08-16 13:59 / 14:11:26 | Suspected — SSH host key recorded; requires investigation |
| Internal host | 192.168.5.1 | Internal system scanned by Nmap (likely gateway) | 2016-08-16 ~13:59 | Scanned — no confirmed compromise |
| Internal host | 192.168.5.15 | Internal system scanned by Nmap | 2016-08-16 ~13:59 | Scanned — no confirmed compromise |
| Attacker infrastructure | 151.80.137.2 | Resolution of `blog.mycompany.ex` (compromised site) | Day of investigation | N/A — attacker/external |
| Attacker infrastructure | Not identified in the provided evidence (`blog.mysportclub.ex`) | Exploit Kit and payload host | 2016-08-16 13:02–13:10 | N/A — attacker/external |

---

## 6. Indicators of Compromise

| # | IOC Type | IOC Value | Why It Matters | Suggested Enterprise Search |
|---|---|---|---|---|
| 1 | Malware family | Xtreme RAT | Confirmed RAT giving full remote control | Hunt YARA rules xtreme_rat/xtremrat across memory and endpoints |
| 2 | File path | `%APPDATA%\HostData\update.exe` | Primary persistent RAT binary | File search across all `AppData` profiles |
| 3 | File path | `%TEMP%\svchost.exe` | Masqueraded RAT copy dropped by exploit | Search `%TEMP%` for `svchost.exe` (never legitimate location) |
| 4 | Directory | `%APPDATA%\EpUpdate\` | Attacker toolkit (mimikatz, Hydra, Nmap, plink/pscp, pwdump) | Search all endpoints for an `EpUpdate` directory |
| 5 | Filename | `54948tp.exe` | py2exe downloader that fetched the toolkit | Filename and hash search across `%TEMP%` |
| 6 | Filename | `3568226350.exe` (`3568226350[1].exe`) | py2exe initial payload from the Exploit Kit | Search proxy/AV logs and file systems |
| 7 | Domain | `blog.mysportclub.ex` | Exploit Kit / payload host | Block and hunt in DNS and proxy logs |
| 8 | Domain / IP | `blog.mycompany.ex` → 151.80.137.2 | Compromised site used as redirector | Block; search DNS/proxy for domain and IP |
| 9 | URL | `http://blog.mysportclub.ex/wp-content/uploads/hk/task/opspy/index.php` | Exploit Kit landing page | Search web proxy logs for the URL path |
| 10 | Registry key | `NTUSER.DAT\...\GhCtxq8t` (plus `Run`/`RunOnce` mods) | Xtreme RAT persistence marker | Sweep endpoints for the `GhCtxq8t` subkey and anomalous `Run` entries |

Additional evidence-supported indicators (secondary): URL `http://blog.mysportclub.ex/wp-content/uploads/hk/files/data_32.bin`; `wdigest.reg` enabling `UseLogonCredential`; exploit reference **CVE-2012-3993**; vulnerable software Firefox 33.0.3 and Adobe Flash 18.0.0.194; internal targets 192.168.5.1 / .10 / .15.

---

## 7. Evidence Collected

### 7.1 Memory Image

**Description:** A live RAM capture taken shortly after the incident was discovered.
**Collection / Review Method:** Analyzed with the Volatility framework. `imageinfo` suggested profile `Win10x86_44B89EEA`; subsequent plugins were run with `--dtb`, `--kdbg`, `--kpcr`, and `--profile` parameters. Plugins used: `yarascan`, `pslist`, `dlllist`, `netscan`.
**Relevant Facts Found:** Xtreme RAT code (and UPX packing) in `svchost.exe` (PID 4888), `explorer.exe` (PID 4872), and `update.exe` (PID 5172); system boot at 12:54:24; malicious processes starting ~13:02:57; two `explorer.exe` processes indicating a RunPE masquerade; `update.exe` command line pointing to `%APPDATA%\HostData\update.exe`; multiple `cmd.exe` children; nonstandard TCP ports plus 80/443 connections.
**How It Supports the Investigation:** Establishes the malware family, the initial process activity, and the persistence/masquerade technique.
**Replication Steps:** Load the image in Volatility with the documented profile/offsets; run `yarascan` against the known-bad rule set, then `pslist`, `dlllist -p <pid>`, and `netscan`; compare process start times and parent PIDs.
**Screenshots:**

![imageinfo output](capturas/fig-02.png)
*Figure 2. Volatility `imageinfo` output.*

![yarascan results](capturas/fig-05.png)
*Figure 5. `yarascan` results.*

![Suspected processes](capturas/fig-07.png)
*Figure 7. Processes where Xtreme RAT / UPX-packed code was detected.*

![pslist](capturas/fig-08.png)
*Figure 8. Process listing (`pslist`).*

![Two explorer processes](capturas/fig-12.png)
*Figure 12. Two `explorer.exe` processes (RunPE masquerade).*

![netscan](capturas/fig-13.png)
*Figure 13. Network connections (`netscan`).*

### 7.2 Forensic Disk Image

**Description:** A full disk image captured after the memory dump and mounted read-only on a forensics workstation.
**Collection / Review Method:** ClamAV antivirus scan of the mounted file system; manual review of file paths, timestamps, and browser cache/history.
**Relevant Facts Found:** CVE-2012-3993 exploit code in the Firefox cache; `3568226350[1].exe` (Xtreme RAT) in INetCache; `%TEMP%\svchost.exe`; `%APPDATA%\HostData\update.exe` confirmed as Xtreme RAT; the `%APPDATA%\EpUpdate` toolkit (created 13:14:47); `%TEMP%\54948tp.exe` (created 13:10:03); browser history showing the visit to `blog.mycompany.ex` and the Exploit-Kit download pattern from `blog.mysportclub.ex`; the injected iframe in `blog.mycompany.ex.htm`.
**How It Supports the Investigation:** Confirms the initial infection vector, payload delivery, and the staged attacker toolkit; corroborates memory findings.
**Replication Steps:** Mount the image read-only; run `clamscan -r`; enumerate Firefox `places.sqlite` history and cache; inspect the cached HTML for injected iframes; review `%APPDATA%\EpUpdate` and `%TEMP%` contents and MAC times.
**Screenshots:**

![ClamAV results](capturas/fig-16.png)
*Figure 16. ClamAV scan results.*

![Firefox history](capturas/fig-17.png)
*Figure 17. Firefox browsing history.*

![iframe injection](capturas/fig-19.png)
*Figure 19. Iframe injection to the Exploit Kit landing page.*

![Download+execute code](capturas/fig-21.png)
*Figure 21. Code that downloads and executes `3568226350.exe`.*

### 7.3 Payload / Downloader Analysis (`3568226350.exe` and `54948tp.exe`)

**Description:** PE32 executables built from Python via `py2exe`, recovered from disk and decompiled.
**Collection / Review Method:** Static decompilation of the embedded Python.
**Relevant Facts Found:** A `DOWNLOAD_URL` pointing to `data_32.bin` on `blog.mysportclub.ex`; a `get_toolz` function that downloads, decrypts, and unpacks the toolkit to `%APPDATA%\EpUpdate`; a `%TMP%\SystemProfile` working directory; automatic execution of mimikatz and BPD. The `SystemProfile` directory held `bpd.log`, `mimikatz.log` (13:14:48), `sysinfo.txt` (13:34:25), and a `netscan/` directory (13:52:21) with Nmap XML for 192.168.5.1/.10/.15.
**How It Supports the Investigation:** Explains how the toolkit arrived and confirms credential theft and network reconnaissance objectives.
**Replication Steps:** Extract the Python from the py2exe binary; decompile; review `DOWNLOAD_URL`, `get_toolz`, and `main`; correlate output artifacts under `%TMP%\SystemProfile`.
**Screenshots:**

![DOWNLOAD_URL / decrypt](capturas/fig-23.png)
*Figure 23. `DOWNLOAD_URL` and decryption routine.*

![get_toolz](capturas/fig-24.png)
*Figure 24. `get_toolz` download/unpack function.*

![netscan results](capturas/fig-26.png)
*Figure 26. Nmap scan output in `netscan/`.*

### 7.4 Prefetch

**Description:** Windows Prefetch files documenting program execution.
**Collection / Review Method:** Prefetch parsing to obtain executable names, run counts, and last-run times.
**Relevant Facts Found:** `update.exe` run at 13:03:03–13:03:04; `54948tp.exe` at 13:10:13; `mimikatz.exe` and `browserprocessdump.exe` at 13:14:47; system-profiling tools 13:34:25–13:34:51 (and `whoami`/`ipconfig` at 13:08–13:10); `nmap.exe` last run 13:59:34 (earlier ~13:56); `hydra.exe` at 14:04:44; `plink.exe` six times (14:10:49–14:23:31); `pscp.exe` at 14:47:12–14:50:09.
**How It Supports the Investigation:** Confirms that dropped tools were actually executed and provides an execution timeline for the attacker's actions.
**Replication Steps:** Parse `C:\Windows\Prefetch\*.pf` with a Prefetch parser; sort by last-run time; correlate with filesystem and registry timestamps.
**Screenshots:**

![Prefetch overview](capturas/fig-27.png)
*Figure 27. Prefetch analysis.*

![nmap/hydra prefetch](capturas/fig-31.png)
*Figure 31. `nmap.exe` and `hydra.exe` execution in Prefetch.*

### 7.5 Windows Event Logs

**Description:** Security event logs around the Hydra execution window.
**Collection / Review Method:** Filtered query for events between 14:03:00 and 14:05:00.
**Relevant Facts Found:** Three Event ID **4798** ("A user's local group membership was enumerated") entries at 14:02:04, 14:03:21, and 14:04:43; two reference `hydra.exe` in EventData, matching the Prefetch Hydra run time.
**How It Supports the Investigation:** Independently corroborates attacker enumeration/credential-attack activity.
**Replication Steps:** Open the Security log; filter Event ID 4798 for the window; inspect EventData for the process name.
**Screenshots:**

![Event 4798](capturas/fig-32.png)
*Figure 32. Event 4798 entries related to `hydra.exe`.*

### 7.6 Registry (RegRipper / WRR)

**Description:** `NTUSER.DAT` hive analysis.
**Collection / Review Method:** RegRipper `regtime` timeline and manual WRR inspection.
**Relevant Facts Found:** `Run`/`RunOnce` modified at 13:02:57; `GhCtxq8t` created at 13:03:10 (tied to `update.exe`, `FirstExecution` 13:03:10 UTC / 15:03:10 local); PuTTY keys modified at 14:11:26; `SSHHostKeys` containing an RSA key for **192.168.5.10**; Uninstall data showing Firefox 33.0.3 and Adobe Flash 18.0.0.194.
**How It Supports the Investigation:** Confirms persistence, the timezone offset, lateral movement to 192.168.5.10, and the vulnerable software that enabled the drive-by.
**Replication Steps:** Run RegRipper `regtime` against `NTUSER.DAT`; open the hive in WRR; inspect `GhCtxq8t`, PuTTY `SSHHostKeys`, and Uninstall subkeys.
**Screenshots:**

![Registry timeline](capturas/fig-34.png)
*Figure 34. `NTUSER.DAT` registry timeline (RegRipper).*

![SSHHostKeys](capturas/fig-37.png)
*Figure 37. `SSHHostKeys` with the RSA key for 192.168.5.10.*

![Firefox version](capturas/fig-39.png)
*Figure 39. Outdated Mozilla Firefox 33.0.3.*

---

## 8. Remediation

The following actions were taken to expel the attacker and restore normal operations:

- **Isolated the affected workstation** from the network to stop any active attacker session (plink/pscp activity was ongoing as late as 14:50 UTC).
- **Reset the compromised user account credentials**, and, because mimikatz, Pwdump, BrowserPasswordDump, and the `wdigest.reg` (`UseLogonCredential`) change indicate credential theft, reset credentials for any account that had logged into the host and prioritized privileged accounts.
- **Reverted the WDigest change** by restoring `UseLogonCredential` to 0 (disabled) to stop cleartext credential caching.
- **Removed the malicious files**: `%APPDATA%\HostData\update.exe`, `%TEMP%\svchost.exe`, `%TEMP%\54948tp.exe`, `%TEMP%\3568226350.exe`, the `%APPDATA%\EpUpdate\` toolkit, and the `%TMP%\SystemProfile\` output directory.
- **Removed persistence** by deleting the `GhCtxq8t` subkey and cleaning the malicious `Run`/`RunOnce` entries from `NTUSER.DAT`.
- **Blocked attacker infrastructure** at the proxy/firewall: domains `blog.mycompany.ex` and `blog.mysportclub.ex` and IP address 151.80.137.2.
- **Reimaged the workstation** from a known-good build rather than attempting in-place cleanup, given the depth of compromise.
- **Investigated internal host 192.168.5.10** (SSH host key recorded, target of plink) and reviewed 192.168.5.1 and 192.168.5.15 for scan follow-through.
- **Verified no additional IOC hits** existed on other endpoints using the indicators in Section 6, and **increased monitoring** (DNS, proxy, and endpoint) for the blocked domains, file paths, and the `GhCtxq8t` key after containment.

---

## 9. Recommendations

### 9.1 Short-Term Recommendations

- **Patch or remove the exploited software** — upgrade Mozilla Firefox from 33.0.3 to a current supported release and remove or fully update the Adobe Flash plugin (18.0.0.194), which was end-of-life and directly enabled the drive-by.
- **Reset affected passwords** for the workstation user and any account exposed to credential theft on the host; force a domain-wide reset of privileged credentials as a precaution.
- **Block the IOCs** from Section 6 (domains, IP, URLs) at the perimeter and add the file/registry indicators to endpoint detection.
- **Reimage the affected host** and remove the `GhCtxq8t` persistence and all dropped tooling (already performed; verify on rebuild).
- **Sweep peer workstations** for the same IOCs, especially any endpoint that browsed `blog.mycompany.ex` or `blog.mysportclub.ex`.
- **Examine internal host 192.168.5.10** for signs of successful lateral movement (SSH logins, new accounts, transferred files).

### 9.2 Long-Term Recommendations

- **Implement vulnerability and patch management** with a focus on browsers and plugins; retire legacy plugins such as Flash entirely.
- **Deploy EDR** across endpoints to detect RAT behavior, RunPE masquerading, and credential-dumping tools like mimikatz.
- **Centralize logs in a SIEM** (Windows Security, PowerShell, proxy, DNS) to enable correlation like the Event 4798 / Hydra timeline reconstructed here.
- **Enforce least privilege** so that a single workstation compromise cannot readily access internal hosts or store cleartext credentials.
- **Implement web filtering / DNS security** to block Exploit-Kit domains and known-bad infrastructure proactively.
- **Establish regular threat hunting** using the IOC and behavioral patterns documented in this report.
- **Deliver user awareness training** and maintain an incident response playbook for drive-by and RAT scenarios.

---

## 10. Lessons Learned

This investigation reinforced the value of building a single, correlated timeline from independent evidence sources. Memory analysis identified the malware family and masquerade technique; the disk image and browser artifacts revealed the initial vector; Prefetch confirmed which tools actually executed; Event Logs and the registry corroborated the timing of credential attacks, persistence, and lateral movement. No single artifact told the whole story, but together they produced a coherent Who/What/When/Where/Why/How narrative. The exercise also demonstrated how quickly investigative leads convert into actionable IOCs suitable for enterprise-wide hunting.

The engagement highlighted the operational importance of disciplined evidence handling and repeatable procedures: documenting the Volatility profile and offsets, the ClamAV scan method, and the Prefetch/registry parsing steps means a third party can reproduce every conclusion. Going forward, the team will continue to prioritize timeline correlation, keep reports concise for executive readers while confining deep technical detail to the evidence and appendix sections, and preserve original images so findings remain defensible.

---

## 11. Appendices

### Appendix A: Key Artifact Timestamps (UTC)

- 12:54:24 — System boot
- 13:02:46 — Visit to `blog.mycompany.ex`
- 13:02:57 — `svchost.exe` created in `%TEMP%`; `Run`/`RunOnce` modified
- 13:03:04 — `update.exe` executed
- 13:03:10 — `GhCtxq8t` persistence created (`update.exe` first execution)
- 13:10:03 / 13:10:13 — `54948tp.exe` created / executed
- 13:14:47 — `EpUpdate` toolkit created
- 13:34:25–13:34:51 — Local system profiling
- 13:59:xx — Nmap scans of 192.168.5.1/.10/.15
- 14:04:44 — Hydra executed
- 14:10:49–14:23:31 — `plink.exe` (six executions)
- 14:47:12–14:50:09 — `pscp.exe` (three executions)

### Appendix B: `%APPDATA%\EpUpdate` Toolkit Contents

`bpd/` (BrowserPasswordDump.exe), `mmktz/` (mimikatz), `nircmd/` (NirCmd), `nmap/` (Nmap 7.12), `pwdump/` (Pwdump), `ssh/` (plink, pscp), `thc/` (THC Hydra), `passwords.txt` (common-password list), `wdigest.reg` (sets `UseLogonCredential`).

### Appendix C: Consolidated Attack Timeline Figures

![Overall timeline part 1](capturas/fig-41.png)
*Figure 41. Overall event timeline — part 1.*

![Overall timeline part 2](capturas/fig-42.png)
*Figure 42. Overall event timeline — part 2.*

![Overall timeline part 3](capturas/fig-43.png)
*Figure 43. Overall event timeline — part 3.*

### Appendix D: Additional Supporting Screenshots

![Volatility parameters](capturas/fig-03.png)
*Figure 3. Volatility command with specified parameters.*

![YARA rules](capturas/fig-04.png)
*Figure 4. YARA rules collected from prior known compromises.*

![YARA hit count](capturas/fig-06.png)
*Figure 6. Count of triggered YARA rules.*

![Malicious PIDs](capturas/fig-09.png)
*Figure 9. Malicious processes filtered by PID.*

![Parent PIDs](capturas/fig-10.png)
*Figure 10. Parent PIDs of malicious processes.*

![dlllist](capturas/fig-11.png)
*Figure 11. Command line used to start each process (`dlllist`).*

![netscan 2](capturas/fig-14.png)
*Figure 14. Network connections (`netscan`) — part 2.*

![netscan 3](capturas/fig-15.png)
*Figure 15. Network connections (`netscan`) — part 3.*

![Suspicious script](capturas/fig-18.png)
*Figure 18. Suspicious script invoked in `blog.mycompany.ex.htm`.*

![Exploit HTML files](capturas/fig-20.png)
*Figure 20. Exploit HTML files referencing `svchost.exe`.*

![Exploit-Kit cache pattern](capturas/fig-22.png)
*Figure 22. Exploit-Kit download pattern in the Firefox cache.*

![SystemProfile contents](capturas/fig-25.png)
*Figure 25. Contents of `%TMP%\SystemProfile`.*

![update.exe prefetch](capturas/fig-28.png)
*Figure 28. `update.exe` executions in Prefetch.*

![Tool execution](capturas/fig-29.png)
*Figure 29. Execution of `54948tp.exe`, `mimikatz.exe`, and `browserprocessdump.exe`.*

![System info tools](capturas/fig-30.png)
*Figure 30. Execution of standard system information-gathering tools.*

![Event 4798 at 14:02:04](capturas/fig-33.png)
*Figure 33. Additional Event 4798 logged at 14:02:04.*

![GhCtxq8t subkey](capturas/fig-35.png)
*Figure 35. `GhCtxq8t` subkey associated with `update.exe` (WRR).*

![PuTTY subkeys](capturas/fig-36.png)
*Figure 36. PuTTY-related subkeys modified at 14:11:26.*

![Uninstall info](capturas/fig-38.png)
*Figure 38. Installed software per registry Uninstall information.*

![Flash version](capturas/fig-40.png)
*Figure 40. Outdated Adobe Flash Plugin 18.0.0.194.*

---

## 12. References

No external sources were used. This report was prepared from the provided evidence document and its supporting figures.
