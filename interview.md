# SOC, SIEM & Splunk - Interview Questions & Answers

Interview-ready answers for SOC analyst / Splunk roles. Kept punchy and memorable; the full detail behind each lives in [README.md](README.md).

## Contents

**Core concepts**
- [What is a SOC?](#what-is-a-soc)
- [What are the three pillars of a SOC?](#what-are-the-three-pillars-of-a-soc)
- [What is event data?](#what-is-event-data)
- [Event vs alert vs incident?](#event-vs-alert-vs-incident)
- [Logs vs events?](#logs-vs-events)
- [What is a SIEM?](#what-is-a-siem)
- [SIEM vs SOAR?](#siem-vs-soar)
- [SOC vs NOC?](#soc-vs-noc)

**Security fundamentals**
- [What is the CIA triad?](#what-is-the-cia-triad)
- [Vulnerability vs threat vs risk?](#vulnerability-vs-threat-vs-risk)
- [IDS vs IPS?](#ids-vs-ips)
- [Authentication vs authorisation?](#authentication-vs-authorisation)
- [What is EDR / XDR?](#what-is-edr--xdr)
- [What is defence in depth?](#what-is-defence-in-depth)
- [Common attacker techniques to know](#common-attacker-techniques-to-know)
- [What is phishing?](#what-is-phishing)
- [What is malware, and its main types?](#what-is-malware-and-its-main-types)
- [Firewalls and common ports?](#firewalls-and-common-ports)

**SOC operations**
- [What are the SOC tiers / roles?](#what-are-the-soc-tiers--roles)
- [What is threat hunting?](#what-is-threat-hunting)
- [What is the incident response lifecycle?](#what-is-the-incident-response-lifecycle)
- [What are MTTD and MTTR?](#what-are-mttd-and-mttr)
- [True positive vs false positive?](#true-positive-vs-false-positive)
- [What is correlation?](#what-is-correlation)

**Frameworks**
- [What is MITRE ATT&CK?](#what-is-mitre-attck)
- [What is the Cyber Kill Chain?](#what-is-the-cyber-kill-chain)
- [IOC vs TTP?](#ioc-vs-ttp)

**Splunk**
- [What is Splunk?](#what-is-splunk)
- [Why use Splunk?](#why-use-splunk)
- [What is SPL?](#what-is-spl)
- [What are the components of Splunk architecture?](#what-are-the-components-of-splunk-architecture)
- [How does data get into Splunk?](#how-does-data-get-into-splunk)
- [What are the basic terms in Splunk?](#what-are-the-basic-terms-in-splunk)
- [Why is the source type important?](#why-is-the-source-type-important)
- [Host vs src?](#host-vs-src)
- [What is schema-on-read?](#what-is-schema-on-read)
- [Splunk apps vs add-ons?](#splunk-apps-vs-add-ons)
- [What can you build in Splunk?](#what-can-you-build-in-splunk)
- [What is Splunk Enterprise Security?](#what-is-splunk-enterprise-security)

**Scenario questions**
- [How would you triage a brute-force alert?](#how-would-you-triage-a-brute-force-alert)
- [A user reports a phishing email - what do you do?](#a-user-reports-a-phishing-email---what-do-you-do)
- [An EDR/antivirus alert fires on a workstation - how do you handle it?](#an-edrantivirus-alert-fires-on-a-workstation---how-do-you-handle-it)
- [A log source stops reporting - what do you do?](#a-log-source-stops-reporting---what-do-you-do)
- [Walk me through investigating a suspicious login.](#walk-me-through-investigating-a-suspicious-login)

**Behavioural**
- [Why do you want to work in a SOC?](#why-do-you-want-to-work-in-a-soc)
- [How do you stay up to date with security?](#how-do-you-stay-up-to-date-with-security)
- [Do you have a home lab or project?](#do-you-have-a-home-lab-or-project)

---

## Core concepts

### What is a SOC?

A **SOC (Security Operations Centre)** is a centralised team and facility responsible for **continuously monitoring, detecting, analysing, and responding** to cybersecurity threats across an organisation. It typically operates **24/7**, and its core goal is to reduce the time between a threat emerging and it being contained - measured through **MTTD** (mean time to detect) and **MTTR** (mean time to respond).

A SOC is built on three pillars: **People, Processes, and Technology** (see below).

> **Key point:** a SOC is primarily about **detection and response**, not prevention - keeping attackers out is mostly the job of firewalls, patching, and access controls.

### What are the three pillars of a SOC?

**People, Processes, and Technology** - all three must work together; strong tools are useless without skilled people and solid processes, and vice versa.

- **People** - the analysts and specialists who run the SOC:
  - Tier 1 - alert triage and monitoring
  - Tier 2 - investigation and incident response
  - Tier 3 - threat hunting, forensics, advanced analysis
  - SOC Manager / Lead - strategy, oversight, reporting
  - (Requires ongoing training to keep pace with evolving threats.)
- **Processes** - documented procedures that guide consistent action:
  - Playbooks / runbooks - step-by-step guides for specific incident types
  - Escalation paths - when and how to hand off between tiers
  - Incident response lifecycle (maps to NIST / SANS frameworks)
  - Compliance & reporting standards (GDPR, PCI-DSS, ISO 27001)
  - (Ensures repeatability, accountability, and speed under pressure.)
- **Technology** - the tools that enable monitoring and defence:
  - **SIEM** (e.g. Splunk) - central log aggregation, correlation, alerting
  - IDS/IPS, firewalls, **EDR** (endpoint detection & response)
  - **SOAR** - automation and orchestration of response
  - Threat intelligence feeds
  - (Powers the people and processes - without it, analysts are blind.)

### What is event data?

**Event data** is a record of any single occurrence or activity within a system, device, or network - captured with details like *what* happened, *when*, *where*, and *who/what* was involved. Each event is essentially a **timestamped log entry**.

In a SOC, event data is the raw material analysts work with. Sources include:

- **Operating systems** - logins, logouts, file access, privilege changes
- **Network devices** - firewall allow/deny, router/switch traffic
- **Applications** - errors, transactions, user actions
- **Security tools** - antivirus/EDR detections, IDS/IPS alerts
- **Authentication systems** - successful and failed logins

A typical event contains a **timestamp, source, event type, severity, and description** - e.g. *"User X failed login from IP 10.0.0.5 at 14:32."*

> **Why it matters:** one event is just noise - thousands are generated every second. The value comes from **collecting, correlating, and analysing** event data (usually in a SIEM) to spot patterns. One failed login is nothing; 200 failed logins across 50 accounts in a minute is a likely brute-force attack.

### Event vs alert vs incident?

A funnel - millions of events get filtered down to a handful of real incidents:

- **Event** - any observed occurrence (a login, a request, a process start).
- **Alert** - an event (or correlated set) that matches a detection rule and warrants attention.
- **Incident** - a confirmed threat that requires a response.

> **events → alerts → incidents.** Millions → thousands → a handful.

### Logs vs events?

- A **log** is the raw, machine-generated record a system writes - typically a line in a file or stream.
- An **event** is a single occurrence represented by that record - in practice, one parsed log entry.

Loosely: the **log** is the file/stream; the **event** is one meaningful entry within it. In Splunk, each **event** is one indexed record with a timestamp.

### What is a SIEM?

A **SIEM (Security Information and Event Management)** is a platform that **collects and aggregates log data** from across an organisation's IT systems into one central, searchable place.

The foundation is **logs** - machine-generated records every server, firewall, endpoint, app, and auth system produces constantly. Individually they're scattered lines of text in thousands of places; the SIEM's first job is to **pull them together** so they can actually be used. From there it does two things, reflected in the name:

- **SIM (Information Management)** - collecting, storing, and analysing log data over time.
- **SEM (Event Management)** - real-time monitoring, correlation, and alerting.

Its main value is **correlation**: linking events from different logs to detect threats no single source would reveal - e.g. a failed login on one system, a successful login on another, then a large data transfer.

> **Key point:** a SIEM **detects and alerts** in real time - it's a visibility/detection tool, **not** a prevention tool. Blocking and response come from firewalls, EDR, and SOAR. Examples: **Splunk, Microsoft Sentinel, IBM QRadar**.

### SIEM vs SOAR?

- A **SIEM detects** - aggregates logs, correlates them, and raises alerts (visibility).
- A **SOAR responds** - takes those alerts and automates the reaction via **playbooks** (disable an account, open a ticket, block an IP).

The SIEM is the **brain** that spots the problem; the SOAR is the **hands** that act on it. Modern platforms increasingly bundle both.

### SOC vs NOC?

- A **SOC (Security Operations Centre)** focuses on **security** - detecting and responding to threats. It asks *"is this an attack?"*
- A **NOC (Network Operations Centre)** focuses on **availability and performance** - keeping systems up, fast, and healthy. It asks *"is this system working?"*

They watch similar telemetry for different goals.

---

## Security fundamentals

### What is the CIA triad?

The three core goals of security: **Confidentiality** (only authorised people can access data), **Integrity** (data isn't altered or tampered with), and **Availability** (systems and data are accessible when needed). Most attacks target one of these - a data breach hits confidentiality, ransomware hits availability, tampering hits integrity.

### Vulnerability vs threat vs risk?

- **Vulnerability** - a weakness (e.g. an unpatched server).
- **Threat** - something that could exploit that weakness (a hacker, malware).
- **Risk** - the likelihood *and* impact of it actually happening.

Simple version: **Risk = Threat × Vulnerability × Impact.** A vulnerability with no threat, or a threat with no vulnerability, is low risk.

### IDS vs IPS?

Both inspect network traffic for malicious activity - the difference is what they do about it:

- **IDS (Intrusion Detection System)** - **detects and alerts** only. Passive.
- **IPS (Intrusion Prevention System)** - sits **inline** and can **block** the traffic. Active.

Think of it as: **IDS = alarm; IPS = alarm + locked door.**

### Authentication vs authorisation?

- **Authentication (AuthN)** - proving *who you are* (password, MFA, biometrics).
- **Authorisation (AuthZ)** - what you're *allowed to do* once you're authenticated (permissions, access level).

One-liner: **AuthN = identity; AuthZ = access.**

### What is EDR / XDR?

- **EDR (Endpoint Detection & Response)** - security software on endpoints (laptops, servers) that detects, investigates, and responds to threats on that device (e.g. CrowdStrike, Microsoft Defender).
- **XDR (Extended Detection & Response)** - extends that idea across **endpoints, network, email, and cloud** into one correlated view.

### What is defence in depth?

Layering **multiple security controls** so that if one fails, others still protect you - e.g. firewall + EDR + MFA + patching + user training. No single control is enough, so an attacker has to beat several. The SOC is one layer in this stack.

### Common attacker techniques to know

Terms you'll be expected to recognise (they map to the [Cyber Kill Chain](#what-is-the-cyber-kill-chain) and [MITRE ATT&CK](#what-is-mitre-attck)):

- **Phishing** - tricking a user into clicking a link or giving up credentials (a common entry point).
- **Privilege escalation** - gaining higher access than you were granted (user → admin).
- **Lateral movement** - hopping from one compromised host to others inside the network.
- **Persistence** - keeping access after a reboot (scheduled tasks, new accounts, registry keys).
- **Exfiltration** - stealing data out of the network.
- **Command-and-control (C2)** - malware "phoning home" to the attacker for instructions.

### What is phishing?

A **social-engineering attack** that tricks a user into revealing credentials or running malware, usually via a deceptive email or link. It's one of the most common initial attack vectors - which is why **user-reported phishing** is a routine Tier 1 workflow.

### What is malware, and its main types?

**Malware** = malicious software. Main types:

- **Virus** - attaches to a file and spreads when it's run.
- **Worm** - self-spreading, no user action needed.
- **Trojan** - disguised as something legitimate.
- **Ransomware** - encrypts data and demands payment.
- **Spyware** - secretly steals information.
- **Rootkit** - hides deep in the OS to avoid detection.

### Firewalls and common ports?

A **firewall** controls network traffic against a set of rules, allowing or blocking by IP, port, and protocol. Ports worth memorising:

| Port | Service | | Port | Service |
| --- | --- | --- | --- | --- |
| **80** | HTTP | | **53** | DNS |
| **443** | HTTPS | | **25** | SMTP (email) |
| **22** | SSH | | **3389** | RDP (remote desktop) |

Also know: **TCP** is reliable and connection-based; **UDP** is faster and connectionless.

---

## SOC operations

### What are the SOC tiers / roles?

A triage-and-escalation ladder - alerts flow **up**, and only what a tier can't resolve is escalated:

- **Tier 1 - Monitoring / Triage** - "eyes on glass"; watch the alert queue, decide real vs false positive, escalate the genuine ones.
- **Tier 2 - Incident Responder** - deeper investigation, determine scope/impact, contain and remediate.
- **Tier 3 - Threat Hunter / SME** - proactive hunting, forensics (DFIR), malware analysis, detection engineering.
- **Tier 4 / SOC Manager** - strategy, staffing, metrics, and major-incident coordination.

> Many modern SOCs are flattening these tiers as **SOAR** automates repetitive Tier 1 work, but the vocabulary is still the standard way to describe "who does what."

### What is threat hunting?

**Proactively searching for threats that evaded automated detection** - hypothesis-driven ("assume breach and go looking") rather than waiting for an alert to fire. A hunt: form a hypothesis (often around a **MITRE ATT&CK** technique) → gather the data → analyse → confirm or refute → turn any finding into a new automated detection.

### What is the incident response lifecycle?

Two standard models:

- **NIST (4 phases):** Preparation → Detection & Analysis → Containment, Eradication & Recovery → Post-Incident Activity (lessons learned).
- **SANS (6 steps):** Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned.

The loop is the key idea: every incident should feed improvements back into detections and playbooks.

### What are MTTD and MTTR?

- **MTTD - Mean Time to Detect:** average time to notice a threat after it becomes active.
- **MTTR - Mean Time to Respond/Resolve:** time from detection to containment/resolution.

Lower is better; driving both down is the SOC's central goal. Related: **dwell time** - how long a specific attacker was actually present before being found.

### True positive vs false positive?

- **True positive** - an alert correctly flags a real threat.
- **False positive** - an alert fires on benign activity that only *looks* suspicious.
- **False negative** - a real threat that was **not** caught (the dangerous one).

Most alerts are false positives; continuous **tuning** reduces the noise so real alerts aren't lost (and analysts avoid **alert fatigue**).

### What is correlation?

**Linking events from different sources to reveal a threat no single log shows.** Example: a failed login on one system + a successful login on another + a large outbound transfer = possible account takeover. No single log tells that story; the SIEM stitches them together. Correlation is the core value of a SIEM.

---

## Frameworks

### What is MITRE ATT&CK?

A **knowledge base of real-world attacker tactics and techniques (TTPs)**. SOCs map their detections and hunts to it, so they can reason about *how* attackers behave and find gaps in their coverage.

### What is the Cyber Kill Chain?

A **Lockheed Martin model of the stages of an attack**: reconnaissance → weaponise → deliver → exploit → install → command-and-control (C2) → actions on objectives. Useful for thinking about *where* you can break an attack.

### IOC vs TTP?

- **IOC (Indicator of Compromise)** - a specific artefact signalling a breach (a malicious IP, file hash, or domain). Precise but brittle - attackers change them easily.
- **TTP (Tactics, Techniques & Procedures)** - *how* an adversary operates. More durable and harder to change, so more valuable for detection.

---

## Splunk

### What is Splunk?

**Splunk is a data platform most commonly used as a SIEM** - it collects log and event data from across an organisation's IT systems, pulls it into one central place, and lets analysts **search, correlate, and visualise** it to detect threats. (Strictly, Splunk Enterprise is the platform and **Enterprise Security** is the SIEM app that runs on top - but in an interview it's fine to call Splunk a SIEM.)

> **The security-camera analogy.** Imagine a building with hundreds of cameras - one on every door, corridor, and server room. Each camera is a system generating logs (firewalls, servers, endpoints, apps, login systems). Individually each sees only its own patch, and no one could watch hundreds of screens at once.
>
> **Splunk is the control room** that wires every camera into one wall of monitors watched from a single desk. Now one analyst can:
> - **See everything in one place** - no running between rooms (Splunk **aggregates** all the logs centrally).
> - **Search instantly** - *"show me everyone who entered the server room after 2am"* instead of scrubbing hours of footage (**SPL**, Splunk's search language).
> - **Get automatic alerts** - the system flags *"someone's rattled the back door 200 times"* so the analyst doesn't have to stare and hope (**correlation and alerting**).
> - **Spot patterns across cameras** - the same suspicious person at the front door, then the stairwell, then the vault; no single camera tells that story, but the control room connects them (**correlation**).
>
> Without the control room you'd need hundreds of people each watching one screen - impossible. Splunk lets one analyst effectively monitor the whole building by centralising every feed and doing the heavy watching for them.

### Why use Splunk?

- **Ingests almost anything** - schema-on-read, so messy/varied log formats go in without modelling them first.
- **Powerful search language (SPL)** - slices data in ways simple keyword search can't.
- **Real-time** - search and alert on data as it arrives.
- **Scales massively** - from a laptop to petabyte clusters.
- **Huge ecosystem** - thousands of ready-made apps and add-ons.

The main trade-off is **cost** - historically licensed by **data volume ingested per day**, so what you choose to log has a direct price tag.

### What is SPL?

**SPL (Search Processing Language)** is Splunk's query language. Its defining feature is the **pipe (`|`)**: like a Unix shell, each command's output feeds the next, so a search is a **chain** - retrieve → transform → present.

```spl
index=web status=404
| stats count by clientip
| sort -count
```
> Retrieve all 404 events, count them per client IP, and rank most-frequent first - a simple noisy-client detector.

### What are the components of Splunk architecture?

Three core components; data flows one way, **Forwarder → Indexer → Search Head**:

- **Universal Forwarder (UF)** - a lightweight agent installed on the source machines that **collects and ships** logs. No UI, minimal footprint.
- **Indexer** - the engine room: **parses, indexes, and stores** the data, and **runs the searches** over its slice of it.
- **Search Head** - the analyst's UI: you type **SPL** here, it dispatches the search to the indexers, then merges and presents the results. Dashboards, reports, and alerts live here.

### How does data get into Splunk?

Several routes, picked per source:

- **Universal Forwarder** - agent on the source (standard for servers/endpoints at scale).
- **Monitor input** - watch a file/directory or network port locally.
- **Upload / one-shot** - manually upload a file via the UI (great for testing).
- **HTTP Event Collector (HEC)** - apps/cloud services POST events to an HTTP endpoint (no forwarder needed).
- **Scripted / modular inputs** - pull from an API on a schedule (e.g. cloud logs).
- **Syslog** - receive straight from network devices.

Whichever route, data is parsed into events and stored in an **index**, ready to search.

### What are the basic terms in Splunk?

- **Index** - where Splunk stores processed data (think "database"); default is `main`.
- **Sourcetype** - the data's format (e.g. `access_combined`), which tells Splunk how to parse it.
- **Source** - the file/input an event came from.
- **Host** - the machine an event originated on.
- **Field** - a name-value pair extracted from an event (`status=404`, `user=alice`).
- **Event** - a single record; usually one log line, with `_time` (timestamp) and `_raw` (original text).
- **Eventtype / knowledge object** - user-created content that tags and enriches data for reuse.

### Why is the source type important?

The **sourcetype tells Splunk how to parse the data**, so getting it right (or wrong) affects everything downstream:

- **Parsing** - it drives **timestamp recognition**, **line breaking** (where one event ends and the next begins), and **field extraction**. The wrong sourcetype means broken timestamps, mis-split events, and missing fields.
- **Knowledge & add-ons** - field extractions, CIM mappings, and an add-on's parsing are all **keyed to the sourcetype**, so the right one applies the right logic automatically (e.g. `access_combined` extracts `clientip`, `status`, `bytes` for you).
- **Searchability** - consistent sourcetypes let you filter and **correlate across similar data** (`sourcetype=linux_secure`), and it's an efficient filter alongside index and time.

In short: sourcetype is the key that maps raw data to the correct parsing and field logic - get it wrong and your searches, dashboards, and detections are all built on mis-parsed data.

### Host vs src?

Easy to confuse, but they sit at different levels:

- **`host`** - Splunk **metadata**: the machine that **generated or forwarded the log** (i.e. *which system logged the event*).
- **`src`** - a **field in the event data**: the **source of the activity**, e.g. the source IP in a firewall/auth log (a CIM field - *who/what initiated the action*).
- (**`source`** - also metadata: the **file/input** the event came from, e.g. `/var/log/auth.log`. People mix this up with `src` too - it isn't the same.)

*Example:* firewall `fw01` logs a connection from `10.0.0.5` → `8.8.8.8`. Here **`host` = `fw01`** (the box that produced the log) and **`src` = `10.0.0.5`** (the source IP *inside* the log). One says *which system logged it*, the other says *who initiated the action*.

### What is schema-on-read?

Splunk **stores raw data as-is and extracts fields at search time**, not at ingest. Contrast **schema-on-write** (traditional databases), which forces you to define the structure *before* loading data. Schema-on-read is why you can onboard unfamiliar or changing log formats instantly, without redesigning anything.

### Splunk apps vs add-ons?

- **Add-on (TA - Technology Add-on)** - gets data **in** and makes it usable: inputs, parsing, field extractions, CIM mapping. Usually **no UI** (the plumbing).
- **App** - the **experience on top**: dashboards, reports, UI, workflow (the rooms). e.g. **Splunk Enterprise Security**.

Apps often **depend on** add-ons - the add-on normalises the data, the app analyses it. Both come from **Splunkbase**.

### What can you build in Splunk?

- **Searches** - ad-hoc investigation and hunting.
- **Reports** - saved/scheduled searches (e.g. a daily failed-login summary).
- **Alerts** - a saved search that fires an **action** (notify, ticket, SOAR playbook) when a condition is met - the core of automated detection.
- **Dashboards** - panels of visualisations giving a live view.
- **Knowledge objects** - reusable eventtypes, tags, lookups, and data models.

### What is Splunk Enterprise Security?

A **premium SIEM app** that runs on top of Splunk Enterprise/Cloud - correlation searches, **notable events**, risk-based alerting, and dashboards mapped to frameworks. It turns raw Splunk into turnkey SOC content.

---

## Scenario questions

### How would you triage a brute-force alert?

Start with **context - is it real or a false positive?** An account logging in 10 times then succeeding, from its usual IP, is probably a user who forgot their password. A **real** brute force looks different: **hundreds of attempts**, from an **unfamiliar IP**, against **many accounts**, at machine speed. Check the baseline, the source IP, whether it's all failures, the time of day, and what happened next. If it's genuine, weigh **impact and scope**, then escalate to Tier 2 with the context gathered.

### A user reports a phishing email - what do you do?

**Don't click anything.** Examine the email safely: check the **real sender address** (not just the display name), **hover the links** to see where they actually go, and check attachments via a **hash lookup or sandbox** rather than opening them. See **who else received it**. If it's malicious: quarantine/report it, block the sender, domain, and URLs, warn users, and crucially **check whether anyone already clicked or entered credentials** - if so, reset those accounts. Then document it.

### An EDR/antivirus alert fires on a workstation - how do you handle it?

**Triage first:** what did it flag (file, process, hash), on which host and user, and is it a known false positive? Check the **file hash against threat intel** (e.g. VirusTotal). If it looks like a real detection, **contain it** - isolate the host from the network to stop any spread - then **escalate to Tier 2** for deeper investigation and clean-up. Preserve the evidence rather than just deleting the file.

### A log source stops reporting - what do you do?

A silent source is a **blind spot** - you can't detect what you don't collect. First treat it as an operational issue: is a forwarder or pipeline broken? But also treat it as a **potential security signal** - attackers deliberately disable logging to hide. Investigate the cause, restore the feed, and ideally have an alert on **log-source health** so silences are caught automatically.

### Walk me through investigating a suspicious login.

Frame it with three questions:

1. **When did it happen?** Build a timeline - first occurrence, frequency, still ongoing? (3am activity on a 9-5 account is a flag.)
2. **What happened?** Which user, host, IP, and action fired the alert? Pull the surrounding logs to see what came before and after.
3. **Is it normal?** Compare against the **baseline** for that user/system - same IP as always, or a new country? Followed by anything unusual (privilege change, data access)?

Then **correlate** across sources (auth logs + endpoint + network) to confirm scope, and escalate with the evidence if it's real.

---

## Behavioural

> These matter as much as the technical questions at junior level - interviewers want **genuine interest and initiative**, not perfect answers.

### Why do you want to work in a SOC?

Show real interest in **defence, investigation, and continuous learning** - you enjoy piecing together what happened and staying ahead of attackers. Back it with something hands-on: a **home lab** (like this Splunk/Docker project), CTFs, or TryHackMe/Splunk BOTS. Avoid "for the money" or "to become a pentester" (signals you don't actually want the blue-team role).

### How do you stay up to date with security?

Name concrete sources: security news (**The Hacker News, BleepingComputer, r/netsec**), hands-on platforms (**TryHackMe, Hack The Box, Splunk BOTS**), following researchers on LinkedIn/X, and doing **home-lab projects**. This shows initiative, which is exactly what's valued when you don't yet have experience.

### Do you have a home lab or project?

**Yes - and lead with it.** Describe this project: a **Splunk SOC lab running in Docker**, ingesting logs and writing SPL detections against them. It demonstrates you can set up tooling, get data in, and actually hunt for threats - hands-on initiative is the single biggest thing that separates junior candidates who get hired.

---

*A study companion to [README.md](README.md) (full detail) and [docker.md](docker.md) (the hands-on Splunk lab).*
