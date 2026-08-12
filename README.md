# SOC & SIEM — Fundamentals

A **Security Operations Center (SOC)** is the team (and the function) responsible for **continuously monitoring, detecting, analysing, and responding to cybersecurity threats** across an organisation. Think of it as the organisation's security "control room" — people, processes, and tooling working together to catch attacks early and respond fast when something gets through. (Note a SOC is primarily about **detection and response**, not prevention — keeping attackers out in the first place is mostly the job of firewalls, patching, and access controls.)

> **Analogy — cameras, not a guard in every room.** You can't post a security guard in every room of a building, but you _can_ put a **camera** in each one and watch them all from a single control room. A SOC works the same way: it can't place a person on every server, laptop, and account, so instead everything is **instrumented** — logs and telemetry are the "cameras" — and analysts watch those feeds centrally through the **SIEM** to spot where the problem is. That's the whole idea behind centralised monitoring.

A SOC is fundamentally a **blue team** (defensive) operation. It's usually structured in two overlapping ways at once:

1. **Tiers / lines** — a triage-and-escalation ladder (Tier 1 → Tier 2 → Tier 3), so cheap, high-volume work is filtered first and only the hard cases reach senior people.
2. **Functional divisions** — specialised teams (threat intel, hunting, incident response, detection engineering, etc.) that the tiers draw on.

---

## The tiered model (tiers / lines of support)

This is the classic SOC structure. Alerts flow **up** the tiers; only what can't be resolved at one level is **escalated** to the next.

### Tier 1 — Monitoring / Triage Analyst _(1st line SOC)_

The **front line** — "eyes on glass." They watch the alert queue in real time and are the first humans to see anything suspicious.

> **The core three:** in practice the day-to-day heart of the 1st line is **monitoring → detection → triage**. The rest below (initial investigation, first-response containment, reporting, improvement) are lighter touches or shared with higher tiers — but monitor/detect/triage is where a Tier 1 analyst spends most of the shift.

**What they do**

- Continuously monitor the **SIEM** and other alert sources (EDR, IDS/IPS, firewalls).
- **Triage** each alert: is it real or a false positive? How urgent?
- Do initial investigation and **categorise/prioritise** incidents.
- Resolve simple, known issues using **runbooks/playbooks**.
- **Escalate** anything genuine or unclear to Tier 2, with the context they've gathered.

#### Dashboards

The analyst's main working surface is a **dashboard** in the SIEM — a real-time visual summary of the alert feed and the environment's health. Building and tuning these is part of the monitoring job (often set up with SOC/detection engineers) so the signals that matter stand out at a glance:

- The live **alert queue** — open incidents by **severity**, and what needs attention right now.
- **Trends & metrics** — alert volumes over time, false-positive rates, top-triggering rules, the noisiest systems.
- **Health / coverage** — are all log sources still reporting? (a silent source is a **blind spot**).

These same dashboards feed the **reports** that show stakeholders how the SOC is performing (see [Reporting](#reporting) below) — the dashboard is the live view, the report is the periodic summary drawn from it.

#### Detection — what they're looking for

Detection is the other half of the 1st-line job: monitoring is _watching_ the feeds; detection is _recognising_ which activity in them is suspicious. Common red flags a Tier 1 analyst scans for:

- **Suspicious patterns** — anything that deviates from the normal **baseline**: activity at odd hours, spikes in traffic or resource use, repeated failed logins (possible brute force), or many alerts clustering together.
- **Malware indicators** — EDR/antivirus hits, unrecognised or malicious processes, known-bad file hashes, connections to known command-and-control (C2) servers.
- **Unusual account activity** — logins at strange times or from new devices, **privilege escalation**, dormant accounts suddenly active, or **impossible travel** (the same account logging in from two distant locations minutes apart).
- **Abnormal data transfer** — large or unexpected outbound transfers, uploads to unknown external services — classic signs of **data exfiltration**.
- **IP / location changes** — logins from new, foreign, or blocklisted IPs; connections to/from known-malicious addresses; sudden geolocation changes for a user or host.

The skill is knowing what **"normal" looks like** for the environment, so that **anomalies** stand out. Most detections are automated by rules/signatures in the SIEM and EDR — the analyst's job is to interpret what those detections mean and decide whether they're real.

#### Triage — sorting real threats from noise

Once something is detected, triage answers **two questions**:

**1. Is the issue real?** _(true positive or false positive?)_

**The vast majority of alerts are false positives** — benign activity that merely _looks_ suspicious. The classic example: an account logs in **10 times in a row**, which trips a "possible brute force" rule. Suspicious on the surface — but the mundane explanation is usually the right one: the user **forgot their password** and kept retrying, their phone was auto-retrying a saved (stale) password, or a legitimate script was re-authenticating. None of that is an attack. A _real_ brute force would more likely show **hundreds of attempts**, from an **unfamiliar IP**, against **many accounts**, at machine speed — a different shape entirely. The analyst decides using **context**: What's the baseline for this user/system? Same IP as always? Successes or all failures? Odd hour? Followed by anything else unusual?

**2. What priority does it have?** _(how urgent / severe?)_

A confirmed-real alert still isn't all equal — analysts **rank** each one so the worst are handled first. Priority typically weighs:

- **Impact** — how critical is the affected system or data? (A domain controller or customer database beats a test box.)
- **Scope** — one machine, or spreading across many?
- **Stage of attack** — a blocked probe is lower priority than active data exfiltration or ransomware already encrypting files.
- **Confidence** — how sure are we it's real?

This priority (often a Critical/High/Medium/Low severity) drives how fast it's escalated and who gets pulled in.

Getting both questions right matters both ways: **miss a true positive** and an attack slips through; **over-escalate false positives** and you flood Tier 2 and burn out on **alert fatigue**. Tuning detection rules to cut false-positive noise is a constant background task (and feeds back to Detection Engineering).

#### Investigation (initial)

For anything that survives triage, the analyst does a **first-pass investigation** to build the story before escalating. Three questions frame it:

- **When did it happen?** Build a **timeline** — first occurrence, frequency, still ongoing or finished? Timing is a clue in itself (3 a.m. activity on a 9-to-5 account is a flag).
- **What happened?** What exactly did the alert fire on — which user, host, IP, process, action? Pull the surrounding logs to see what came before and after.
- **Is it normal?** Compare against the **baseline**. Is this expected behaviour for this user/system, or a genuine deviation?

Tier 1 does the _initial_ dig and gathers context; **deep investigation and forensics escalate to Tier 2/3.**

#### Incident response (first-response actions)

When an incident is confirmed, the 1st line often takes **immediate containment steps** from a **playbook** to limit damage while the incident is escalated. For a **compromised account**, typical first actions:

- **Disable / block the account** so the attacker loses access.
- **Force a password reset** (change the credentials).
- **Force logout / revoke active sessions and tokens**, so existing logins are killed, not just future ones.
- **Isolate an affected host** from the network if a machine is involved.

These are containment _first aid_ — stop the bleeding fast. Full **eradication and recovery** (root-cause removal, rebuilding systems) is led by Tier 2/3 and the incident-response process.

#### Reporting

The 1st line **documents** what it sees and hands it on:

- **Log every incident** — a clear record of the alert, the investigation, actions taken, and the outcome (for the audit trail, handovers, and later analysis).
- **Shift handover** — pass ongoing issues to the next shift in a 24×7 SOC.
- Feed **metrics** — alert volumes, false-positive rates, response times — that show **how well the SOC is working** and where it's struggling.

#### Continuous improvement

The SOC gets better by closing the loop. The 1st line contributes by:

- **Flagging noisy/false-positive rules** so detections get **tuned** (fewer false alarms, less alert fatigue).
- **Updating playbooks/runbooks** when a case reveals a gap or a better procedure.
- Surfacing **new threat patterns** worth a detection, feeding Detection Engineering and Threat Hunting.

The goal is a feedback loop: every incident should make the _next_ one faster to catch and handle.

**Focus:** high volume, speed, filtering noise. The main job is to separate the small fraction of alerts that matter from the flood of false positives, and to escalate the real ones quickly and cleanly.

**Escalates to Tier 2 when:** an alert is confirmed suspicious/malicious, or is beyond a documented runbook.

#### Concrete example — AWS CloudWatch monitoring EC2 CPU usage

A practical picture of what "monitoring" means in the cloud:

1. **The source.** **Amazon CloudWatch** collects metrics from **EC2 instances** (virtual servers), including **CPU utilisation**. You set an **alarm** — e.g. "fire if CPU stays above 90% for 5 minutes."
2. **The alert.** One night an instance that normally sits at ~15% CPU pins at 100%. CloudWatch trips the alarm and pushes it into the SOC's queue (often via the SIEM, so it lands next to all the other alerts the Tier 1 analyst is watching).
3. **Triage (Tier 1).** The analyst asks the triage questions: _Is this expected?_ A batch job or traffic spike could explain it (a likely **false positive**). But if nothing scheduled accounts for it, an unexplained, sustained CPU spike is a red flag — a classic sign of **cryptojacking** (malware secretly mining cryptocurrency), a compromised host, or a DoS.
4. **Correlate.** They pull related signals — did CloudTrail show unusual API calls? New outbound network connections? A process the EDR doesn't recognise? A CPU alarm _plus_ connections to a known mining pool is no longer ambiguous.
5. **Escalate.** Confirmed suspicious → hand it to **Tier 2** with the gathered context, who investigate scope and **contain** the instance (isolate it, snapshot for forensics, terminate/rebuild).

The takeaway: a plain infrastructure metric (CPU %) becomes a **security signal** once you monitor it, alarm on the abnormal, and triage _why_. This is exactly the Tier 1 loop — watch a source, catch the anomaly, decide real-vs-noise, escalate the real ones. (CloudWatch/CloudTrail are common cloud **alert sources** feeding a modern SOC.)

### Tier 2 — Incident Responder / Analyst

The **investigators.** They take escalations from Tier 1 and dig in.

**What they do**

- Perform **deeper investigation** on escalated incidents — pull logs, correlate events, work out what actually happened.
- Determine **scope and impact**: which systems, accounts, and data are affected.
- **Contain and remediate** — isolate hosts, block indicators, reset credentials, kill malicious processes.
- Turn findings into **improved detections** and feedback for Tier 1.

**Focus:** understanding the full picture of a confirmed incident and stopping it. Less volume than Tier 1, more depth per case.

**Escalates to Tier 3 when:** the incident is severe, novel, or requires advanced forensics/malware analysis.

### Tier 3 — Threat Hunter / Senior Analyst / SME

The **experts.** Senior specialists who handle the hardest incidents and, crucially, work **proactively** rather than just reacting to alerts.

**What they do**

- **Threat hunting** — proactively search for threats that evaded automated detection, using hypotheses and threat intel (assume breach, go looking).
- **Advanced investigation & forensics** — deep-dive analysis of major incidents (DFIR).
- **Malware analysis / reverse engineering**.
- Develop and tune **detection rules** so future instances get caught automatically (feeding Detection Engineering).
- Handle **major/critical incidents** and mentor lower tiers.

**Focus:** proactive defence, advanced expertise, closing the gaps that alerts miss.

### Tier 4 — SOC Manager / Lead

**Management and strategy** (sometimes counted as "Tier 4," sometimes as a separate function).

**What they do**

- Own the SOC's **overall strategy, staffing, and processes**.
- Manage **escalations to leadership** and coordinate major incidents.
- Report **metrics** to executives; own budgets and tooling decisions.
- Liaise with other business units, legal, comms, and external parties.

> **Note:** many modern SOCs are moving _away_ from rigid tiers toward flatter, skill-based models — partly because automation (SOAR) now handles a lot of the repetitive Tier 1 work. But the tier vocabulary is still the standard way the structure is described, and it maps cleanly onto "who does what."

---

## Functional divisions / specialised teams

Beyond the tier ladder, a mature SOC contains (or works closely with) these specialised functions. In a small SOC one person may wear several of these hats; in a large one each is its own team.

| Team / role                               | What they do                                                          | Core responsibility                                   |
| ----------------------------------------- | --------------------------------------------------------------------- | ----------------------------------------------------- |
| **SOC Analysts (Tiers 1–3)**              | Monitor, triage, investigate, respond                                 | Detect and handle threats day to day                  |
| **Incident Response (IR / CSIRT / CERT)** | Coordinate the full response to confirmed incidents                   | Contain, eradicate, recover; run the incident process |
| **Threat Hunting**                        | Proactively hunt for hidden/undetected threats                        | Find what automated detection missed                  |
| **Threat Intelligence (CTI)**             | Gather & analyse intel on attackers, TTPs, IOCs                       | Give the SOC context on _who_ attacks and _how_       |
| **Detection Engineering**                 | Build & tune detection rules/content for the SIEM/EDR                 | Turn threats into reliable automated detections       |
| **Security Engineering / SecOps**         | Build and maintain the SOC's tooling (SIEM, EDR, SOAR, log pipelines) | Keep the detection & response platform running        |
| **Digital Forensics & IR (DFIR)**         | Deep forensic analysis, evidence handling, malware analysis           | Reconstruct incidents; support legal/root-cause       |
| **Vulnerability Management**              | Scan, prioritise, and track remediation of vulnerabilities            | Reduce the attack surface before it's exploited       |
| **Red Team**                              | Simulate real attacks against the org (offensive)                     | Test defences by thinking like an attacker            |
| **Blue Team**                             | The defenders — the SOC itself                                        | Prevent, detect, and respond to attacks               |
| **Purple Team**                           | Facilitate red + blue working _together_                              | Turn attack findings into better detections fast      |
| **SOC Manager / Leadership**              | Strategy, people, process, reporting                                  | Run the SOC as a function                             |

### Red vs Blue vs Purple (a common point of confusion)

- **Blue team = defence.** The SOC _is_ the blue team — monitoring and responding.
- **Red team = offence.** Ethical hackers who simulate real adversaries to find gaps before real attackers do.
- **Purple team = collaboration**, not a permanent team so much as a _way of working_: red shares its attack techniques with blue, blue builds detections for them, and they verify together. "Purple" = the loop that makes both sides better.

---

## The incident response lifecycle

Most SOCs run incidents through a standard lifecycle. The widely used **NIST** model has four phases:

1. **Preparation** — tooling, playbooks, training, and access in place _before_ anything happens.
2. **Detection & Analysis** — spot the incident and understand it (Tier 1 → Tier 2 territory).
3. **Containment, Eradication & Recovery** — stop the spread, remove the threat, restore systems.
4. **Post-Incident Activity** — the "lessons learned" review; feed improvements back into detections and processes.

(The **SANS** model is similar but splits it into six steps: Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned.)

---

## Data: the pipeline and what flows through it

A SOC is only as good as the data reaching it — analysts can't detect what was never collected. Getting telemetry from thousands of sources into a searchable, alertable form is a real **engineering** job, and analysing the right **types of data** is what makes detection possible.

### Building the pipeline (DevOps / engineering)

The **data pipeline** carries raw telemetry from its sources to the analyst's screen. Standing it up and keeping it running is the work of **security/DevOps engineers** (sometimes "SecOps," "detection infrastructure," or "platform" engineers). The typical stages:

1. **Collection** — agents, log forwarders, and APIs pull data from sources (e.g. the **CloudWatch agent** on EC2, Sysmon on Windows, syslog from network gear, cloud APIs).
2. **Ingestion / transport** — a pipeline moves that data reliably at scale (log shippers, message queues like Kafka).
3. **Parsing & normalisation** — varied log formats are converted into a **common schema** (consistent field names, timestamps) so they can be correlated together.
4. **Enrichment** — add context: geo-IP, asset/owner info, threat-intel tags on IPs and hashes.
5. **Storage** — indexed in the **SIEM** (and/or a data lake) so it's fast to query.
6. **Detection & analysis** — correlation rules, dashboards, and queries run over it; matches become alerts.

> **What a SIEM actually is:** the **SIEM** (Security Information & Event Management) is the platform at the centre of all this. In one line: it **ingests machine-generated data** — the flood of logs and events from every source — and **parses it into a form humans can actually use**: normalised, searchable, correlated across sources, shown on dashboards, and turned into alerts. Raw machine output goes in; human-readable, actionable signal comes out. It's the analyst's primary cockpit, and most of the pipeline stages above happen inside it or feed into it.

**Getting alerts right (correct setup for immediate response)**

In a SIEM like **Splunk**, an _alert_ is a saved search that runs on a schedule (or in real time) and fires when its condition is met. Setting these up _correctly_ is what turns passive detection into **immediate response**:

- **Condition / threshold** — fire on the right trigger (e.g. "> 100 failed logins from one IP in 5 minutes"), not on every single event. Too tight misses threats; too loose buries analysts in noise.
- **Severity** — tag each alert (Critical/High/…) so the urgent ones stand out and are routed appropriately.
- **Action** — a well-configured alert doesn't just land in a queue. For the most urgent cases it can **notify/page an analyst**, open a ticket, or trigger an automated **SOAR** playbook — so a critical event gets an immediate reaction instead of waiting to be noticed.

Badly tuned alerts cause the SOC's two failure modes: **too sensitive** → alert fatigue from false positives; **too loose** → real threats slip through. Getting this balance right is ongoing work (owned largely by **Detection Engineering**).

**What the DevOps/engineering side owns:** deploying and configuring the SIEM/EDR, **onboarding new log sources**, keeping ingestion healthy and scalable, automating deployment (infrastructure-as-code), and building the **SOAR** automations. In the cloud, this includes wiring up **CloudTrail/CloudWatch** log delivery so the SOC even sees that data. If the pipeline breaks or a source stops logging, the SOC goes **blind** to it — which is why this "setup" work is as important as the analysis.

### Types of data analysed (event data & sources)

This raw material is often called **event data** — machine-generated records of things that happened — and it comes in several **forms**:

- **Logs** — the bulk of it: timestamped records of events (a login, a request, a process starting, a firewall allow/deny).
- **Metadata** — "data about data": the who/when/where/how of an activity without its full contents (e.g. a network connection's source, destination, port, and size — not the payload).
- **Sensor / IoT telemetry** — readings and events from devices and physical/embedded sensors (badge readers, industrial equipment, smart devices), increasingly part of the picture.

(These overlap rather than being cleanly separate categories — a log line is itself largely _made of_ metadata. The distinction is one of emphasis: a **log** is the full timestamped record, while **metadata** stresses the summary attributes — the who/when/where — rather than any payload.)

Organised by **source**, each answers a different question:

| Data type                     | What it captures                                                                                   | Useful for spotting                                                   |
| ----------------------------- | -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| **Network logs**              | Firewall, IDS/IPS, proxy, VPN, **NetFlow**                                                         | Suspicious connections, port scans, C2 traffic                        |
| **DNS logs**                  | Domain lookups                                                                                     | Malware calling home, data exfil over DNS                             |
| **Endpoint logs**             | EDR telemetry, process creation, **Sysmon**, OS event logs                                         | Malware, unusual processes, privilege escalation                      |
| **Authentication / identity** | AD / Microsoft Entra ID (formerly Azure AD), IAM, MFA, login events                                | Brute force, impossible travel, account takeover                      |
| **Application logs**          | Web server, app, database logs                                                                     | Injection attacks, abuse, errors                                      |
| **Cloud logs**                | AWS **CloudTrail** (API activity), **CloudWatch** (metrics/logs), GuardDuty; Azure/GCP equivalents | Risky API calls, misconfig, resource abuse (e.g. the EC2 CPU example) |
| **Email / gateway**           | Mail security gateway logs                                                                         | Phishing, malicious attachments                                       |
| **Threat intel feeds**        | Known-bad IPs, domains, hashes (IOCs)                                                              | Matching activity against known threats                               |
| **Vulnerability scan data**   | Scanner output                                                                                     | Exposed, exploitable weaknesses                                       |

The analytical goal is the same across all of them: establish what **normal** looks like, then surface the **anomalies** — ideally by **correlating across sources** (a CloudWatch CPU spike _plus_ a CloudTrail anomaly _plus_ a new outbound connection is far stronger evidence than any one alone).

---

## Common event types to look out for

The recurring signals a SOC watches for — this consolidates the Tier 1 detection red flags into one checklist:

- **Authentication anomalies** — brute-force attempts, bursts of failed logins, logins at odd hours, **impossible travel**, logins from new/foreign/blocklisted IPs.
- **Account misuse** — privilege escalation, new admin accounts, dormant accounts reactivating, MFA/credential changes.
- **Malware & suspicious execution** — EDR/AV detections, unknown or suspicious processes, known-bad file hashes, script/PowerShell abuse.
- **Command-and-control (C2)** — beaconing, connections to known-malicious domains/IPs, unusual DNS activity.
- **Data exfiltration** — large or unusual outbound transfers, uploads to unknown services, data staged for export.
- **Lateral movement** — unusual internal connections, remote-execution tools, credential reuse across hosts.
- **Configuration & integrity changes** — security settings altered, logging/AV disabled, new firewall rules, unexpected system/file changes.
- **Resource anomalies** — CPU/memory spikes (e.g. **cryptojacking** — see the EC2 example), unexpected new services.

In Splunk terms, these map to **saved/correlation searches**, and related events can be grouped with **eventtypes** (Splunk's feature for tagging categories of events) to make them easier to search and alert on.

---

## Common tools

The SOC's work runs on a stack of tooling — worth knowing the categories:

| Category                                                 | What it's for                                                                 | Examples of the category                             |
| -------------------------------------------------------- | ----------------------------------------------------------------------------- | ---------------------------------------------------- |
| **SIEM** (Security Information & Event Management)       | Central log collection, correlation, and alerting — the analyst's main screen | Splunk, Microsoft Sentinel, Elastic, QRadar          |
| **SOAR** (Security Orchestration, Automation & Response) | Automate repetitive response steps via playbooks                              | Cortex XSOAR (standalone); often built into the SIEM |
| **EDR / XDR** (Endpoint / Extended Detection & Response) | Detect and respond to threats on endpoints (and beyond)                       | CrowdStrike, Microsoft Defender, SentinelOne         |
| **IDS / IPS** (Intrusion Detection / Prevention System)  | Detect (IDS) or block (IPS) malicious network traffic                         | Snort, Suricata                                      |
| **Threat Intelligence Platform (TIP)**                   | Aggregate and operationalise threat intel / IOCs                              | MISP, Recorded Future                                |
| **Ticketing / Case management**                          | Track incidents through their lifecycle                                       | ServiceNow, Jira                                     |

---

## SIEM — what it is, an analogy, and 2026 tooling

**What is a SIEM?** A **SIEM** (Security Information & Event Management) is the SOC's core platform: it **ingests machine-generated data** — logs and events from every source — and **parses it into a searchable, correlated, alertable form**, turning raw machine noise into human-readable, actionable signal. It's where analysts monitor feeds, run dashboards, and receive their alerts. (See the data-pipeline section above for how data actually reaches it.)

**Analogy — the control-room monitor wall.** If the org's logs are the security **cameras** (from the analogy at the top of this doc), the **SIEM is the wall of monitors in the control room, plus the smart software behind it.** It takes every camera feed — thousands of them, each in a different format — puts them on one screen, time-syncs and labels them, and automatically **flashes an alert** when something looks off. Without it, a guard would have to squint at a thousand feeds at once; with it, the important frame lights up on its own.

**SIEM software (2026).** The major platforms you'll meet as of 2026:

- **Splunk** — the market heavyweight and the focus of this repo; powerful search language (SPL) and a huge app ecosystem (now a Cisco company).
- **Microsoft Sentinel** — cloud-native, tightly integrated with Azure and Microsoft 365.
- **Google Security Operations (Chronicle)** — cloud-scale, Google-backed.
- **Elastic Security** — built on the open ELK stack.
- **IBM QRadar** — long-established enterprise SIEM.
- **CrowdStrike Falcon Next-Gen SIEM** — SIEM converging with EDR/XDR.
- **Palo Alto Cortex XSIAM** — AI-driven, SOC-automation-focused platform.
- **Exabeam, Sumo Logic, Devo** — notable others, strong on analytics/UEBA and cloud-native ingestion.

A 2026 trend: SIEM is **converging with SOAR and XDR** into unified platforms, AI/ML is increasingly used to cut alert noise, and pricing is shifting away from pure data-volume licensing.

---

## Key frameworks & concepts

- **MITRE ATT&CK** — a knowledge base of real-world attacker **tactics & techniques (TTPs)**. SOCs map detections and hunts to it, so they can reason about _how_ attackers behave and where their coverage has gaps.
- **Cyber Kill Chain** (Lockheed Martin) — a model of the stages of an attack (recon → weaponise → deliver → exploit → install → command-and-control → actions). Useful for thinking about where to break an attack.
- **IOC (Indicator of Compromise)** — an artefact that signals a breach (a malicious IP, file hash, domain).
- **TTPs (Tactics, Techniques & Procedures)** — _how_ an adversary operates (more durable and useful than individual IOCs).
- **False positive / true positive** — the core triage judgement: was the alert wrongly fired, or a real threat?
- **Runbook / Playbook** — step-by-step procedures for handling a given alert type consistently.
- **Escalation** — passing an incident up a tier when it exceeds the current level's scope.

---

## SOC metrics (how a SOC measures itself)

- **MTTD — Mean Time to Detect:** the average time taken to notice a threat after it becomes active.
- **MTTR — Mean Time to Respond/Resolve:** how long from detection to containment/resolution.
- **Dwell time** — how long a _specific_ attacker was actually present before being found; the real-world gap that MTTD aims to shrink (lower is better).
- **False positive rate** — how much of the alert volume is noise (drives Tier 1 workload).
- **Alert/ticket volume**, **escalation rate**, and **SLA adherence**.

Lowering **MTTD** and **MTTR** — detect faster, respond faster — is the SOC's central goal.

---

## SOC challenges

Running a SOC is genuinely hard, for reasons worth knowing:

- **Alert volume & fatigue** — a SOC can face tens of thousands of alerts a day, most of them false positives. Analysts become desensitised (**alert fatigue**) and risk missing the real one.
- **False positives & endless tuning** — noisy detections waste time; keeping them tuned is never finished.
- **Skills shortage & burnout** — cybersecurity talent is scarce, and 24×7 shift work drives high turnover.
- **Blind spots** — you can't detect what you don't collect; a misconfigured or silent log source hides attacks.
- **Data volume & cost** — ingesting and storing huge log volumes is expensive (Splunk has historically been licensed by data volume, so _what you choose to log_ has a direct cost).
- **Evolving threats** — attackers constantly change their TTPs, so detections go stale and must be updated.
- **Speed vs accuracy** — pressure to respond fast without over- or under-reacting.
- **Tool sprawl** — many disconnected tools are hard to integrate and correlate across.

## SOC best practices

- **Automate the routine** — use **SOAR** playbooks for repetitive Tier 1 work, freeing analysts for judgement calls.
- **Tune detections continuously** — cut false positives so real alerts aren't lost in the noise.
- **Use playbooks/runbooks** — consistent, repeatable response instead of ad-hoc reactions.
- **Map coverage to MITRE ATT&CK** — know which attacker techniques you can and can't detect.
- **Be metrics-driven** — track MTTD/MTTR and actively drive them down.
- **Hunt proactively** — don't only wait for alerts; assume breach and go looking.
- **Integrate threat intelligence** — enrich alerts with context on known threats.
- **Defence in depth** — the SOC is one layer; pair it with prevention (patching, hardening, least privilege).
- **Protect & monitor the pipeline** — a source that stops logging is a blind spot, so alert on log-source health.
- **24×7 coverage** — attackers don't keep office hours.
- **Close the loop** — every incident should improve detections and playbooks (lessons learned).

## In-house vs outsourced (MSSP / MDR)

Not every org runs its own SOC:

- **In-house SOC** — built and staffed internally. Most control, highest cost.
- **MSSP (Managed Security Service Provider)** — a third party provides SOC services (often monitoring) for many clients.
- **MDR (Managed Detection & Response)** — a more hands-on managed service focused on detection _and_ active response, not just alerting.
- **Hybrid / co-managed** — internal team plus an external provider (e.g. the provider covers overnight/24×7 monitoring).

SOCs typically aim for **24×7×365** coverage, which is a big driver of both staffing models and the shift/tier structure.

---

## Quick glossary

| Acronym          | Meaning                                                                     |
| ---------------- | --------------------------------------------------------------------------- |
| **SOC**          | Security Operations Center                                                  |
| **SIEM**         | Security Information & Event Management                                     |
| **SOAR**         | Security Orchestration, Automation & Response                               |
| **EDR / XDR**    | Endpoint / Extended Detection & Response                                    |
| **IDS / IPS**    | Intrusion Detection / Prevention System                                     |
| **IR**           | Incident Response                                                           |
| **CSIRT / CERT** | Computer Security Incident Response Team / Computer Emergency Response Team |
| **DFIR**         | Digital Forensics & Incident Response                                       |
| **CTI**          | Cyber Threat Intelligence                                                   |
| **IOC**          | Indicator of Compromise                                                     |
| **TTP**          | Tactics, Techniques & Procedures                                            |
| **MTTD / MTTR**  | Mean Time to Detect / Respond                                               |
| **MSSP / MDR**   | Managed Security Service Provider / Managed Detection & Response            |
