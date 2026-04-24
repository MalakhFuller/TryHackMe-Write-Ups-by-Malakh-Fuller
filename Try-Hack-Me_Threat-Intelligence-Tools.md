# TryHackMe: Threat Intelligence Tools
- Room: https://tryhackme.com/room/threatinteltools
- Completed: 2026-04-24  
- Author: Malakh Fuller

## Room Objective
This room is all about understanding the foundational concepts of threat intelligence and developing hands-on familiarity with the open-source tools analysts used to investigate IOCs, track malware infrastructure, analyze phishing emails, and gather threat context.

The exercises simulate real SOC workflows: validating indicators, analyzing phishing emails, enriching suspicious infrastructure, and correlating findings across multiple intelligence sources. My goal was to approach each task as if I were triaging active alerts in a production environment- methodically, defensibly, and with an emphasis on repeatable analysis.

From a SOC perspective, these skills matter because analysts rarely investigate alerts in isolation. Threat intelligence helps turn raw indicators into context: who may be behind activity, what infrastructure is involved, what malware family may be present, and what defensive action should follow.

This room builds the analyst muscle of turning raw indicators into actionable intelligence — a core competency for SOC, CTI, and threat‑hunting roles.

## Tools and Technologies
- Tools: Thunderbird, CyberChef, VirusTotal, sha256sum, md5sum
- Technologies: UrlScan.io, Abuse.ch (MalwareBazaar, FeodoTracker, SSL Blacklist, URLhaus, ThreatFox), PhishTool, Cisco Talos Intelligence
- Prior Knowledge: A+/Networking+/Security+ | TryHackMe Rooms: Junior Security Analyst Intro, Intro to Logs, and Threat Hunting: Introduction

## Summary
This room introduced the operational side of threat intelligence — not just the theory, but the specific platforms a SOC analyst reaches for when triaging a suspicious IP, a flagged URL, or a forwarded phishing email. I worked through realistic scenarios involving malware alias identification, botnet C2 infrastructure lookup, JA3 fingerprint analysis, and hands-on phishing email forensics across three separate email samples.

The room reinforced that threat intelligence is not just about collecting indicators. The analyst must evaluate the source, understand the context, connect related artifacts, and translate findings into defensible security decisions.

The room culminates in two email based scenarios where I analyzed suspicious messages, extracted indicators, validated attachments, and identified malware families. The workflow (as I understand everything so far) mirrors what a Tier 1–2 SOC analyst performs daily: extract → enrich → correlate → document.





## Methodology

### 1. Understanding Threat Intelligence Classifications
Before diving into tooling, the room grounded the work in context. Threat intelligence breaks down into four classifications, each serving a different consumer:

- Strategic Intel (executive level): trend analysis, threat landscape mapping, risk decisions.
- Technical Intel (attack artifacts): hashes, IPs, domains, and other IOC-level evidence that incident responders use to build detection logic.
- Tactical Intel: adversary tactics, techniques, and procedures (TTPs) - feeding directly into the defensive control improvement.
- Operational Intel: specific actor intent and the critical assets most likely in their crosshairs.

This distinction matters because not every intelligence product serves the same audience. Knowing which tier you're operating in shapes how you communicate findings. A Tier 2 analyst handing off a phishing report needs technical and tactical detail, not a strategic briefing. Strategic intelligence may inform business risk decisions, while technical and tactical intelligence help analysts validate alerts, tune detections, and investigate active threats.

The most useful framing was that threat intelligence helps answer:

- Who is attacking?
- What is their motivation?
- What are their capabilities?
- What artifacts or indicators should defenders look for?

That is the same investigative structure used in SOC triage: identify the activity, enrich it with context, determine impact, and decide what action is justified.


### 2. UrlScan.io - Domain Intelligence
I used UrlScan.io to review domain scan results and identify important metadata about TryHackMe’s domain.

Key findings included:

- Cisco Umbrella Rank: 345612
- Domains identified: 13
- Main registrar: Namecheap Inc
- Main IP address: 2606:4700:10::ac43:1b0a

UrlScan.io was useful because it showed how a single domain can produce multiple related artifacts, including contacted domains, IP addresses, HTTP activity, redirects, page metadata, and associated indicators.

- Lesson One: indicators from a scan do not automatically prove malicious activity. They provide leads. An analyst still must validate whether the infrastructure, redirects, resources, or linked domains are expected- or suspicious.

- Lesson Two: UrlScan results are a snapshot. Because the internet is dynamic, the same search run on different days can produce different results — a detail that matters when using scan results as evidence in an investigation timeline. Screenshots and other evidence collection efforts as an investigation unfolds then become critical artifacts for later analysis and understanding.


### 3. Abuse.ch — Mapping Malware Infrastructure & Botnet Indicators
Abuse.ch is a research project out of the Bern University of Applied Sciences that operates five distinct intelligence platforms under one umbrella. I reviewed multiple Abuse.ch projects and used them to connect indicators to malware infrastructure.

- MalwareBazaar | malware samples | sample upload and analysis, searchable by tag, YARA rule, or vendor signature
- FeodoTracker | botnet command-and-control | C2 infrastructure for Emotet, Dridex, TrickBot, QakBot, and BazarLoader
- SSL Blacklist | malicious SSL certificates | JA3/JA3s fingerprints (for TCP-layer blocking)
- URLhaus | malware distribution URLs | filterable by country, ASN, and TLD
- ThreatFox | IOC sharing and export | export to MISP, Suricata, DNS RPZ, JSON, and CSV formats

This section showed how threat intelligence platforms can enrich a single suspicious artifact into a broader picture. An IP address, JA3 fingerprint, URL, or hash all become much more useful when tied to a malware family, hosting provider, country, infrastructure type, or known command-and-control activity.

### 4. PhishTool — Email Forensics on Email1.eml
PhishTool is designed specifically for email analysis, combining header parsing, metadata extraction, OSINT integration, and classification reporting into a single workflow. The task involved analyzing Email1.eml using Thunderbird on the provided VM. 

Phishing investigations require more than reading the body of the email. The analyst must inspect headers, received lines, originating infrastructure, authentication results, attachments, message URLs, and any other potentially suspicious artifacts.

To find the originating IP, I navigated to View > Message Source (Ctrl+U) and read the raw SMTP headers. The originating IP was 204.93.183.11, which was then Defanged using CyberChef's Defang IP Addresses recipe. The originating IP was especially important because it became the pivot point for later reputation enrichment in Cisco Talos.

Defanging is a standard practice when sharing IOCs in reports or tickets. It prevents accidental clicks and keeps threat indicators from being parsed or executed in environments that auto-link IP addresses.

### 5. Cisco Talos Intelligence - Enriching IP Reputation with Attribution and Context
Talos is Cisco's threat intelligence arm, drawing on telemetry from across their product portfolio. For the purposes of this task, the Reputation Center was the primary interface — specifically, reputation lookup by IP.

The IP alone is just a data point. Reputation and ownership details help determine whether the activity aligns with legitimate infrastructure, abused hosting, suspicious mail flow, or known threat activity.

The customer’s name required a WHOIS lookup via ICANN's lookup tool, since Talos did not surface it directly. In practice, this kind of pivot — from IP to WHOIS to customer context — is exactly the kind of multi-source correlation that makes the difference between a thorough investigation and a shallow one.

### 6. Scenario 1 — Email2.eml
For the first scenario, I analyzed another suspicious email and used external intelligence to identify the recipient and investigate the attachment.

This showed the value of historical intelligence. If a file has been seen before, its submission history can help determine whether the artifact is newly emerging, previously known, or connected to older campaigns.

### 6. Scenario 2 — Email2.eml
For the second scenario, I analyzed an email attachment (an Excel file) and connected it to a malware family. This was one of the strongest SOC-relevant sections of the room. Malicious Excel attachments are common in phishing and malware delivery workflows. Identifying the attachment name is useful but associating it with Dridex gives the investigation more meaning because that connection links the artifact to credential theft and banking malware activity.

Dridex has been active for over a decade and continues to be distributed via weaponized Office documents with malicious macros, exactly as modeled in this task.

## Key Findings

### Threat Intelligence Adds Context
A domain, IP address, hash, JA3 fingerprint, or attachment name becomes valuable when it is enriched with reputation, malware association, infrastructure ownership, or historical sightings.

### IOC Lookup Is Only Step One
Every IOC query in this room required at least one follow-on step: drilling into an alias, cross-referencing a customer name, or computing a hash before the VirusTotal search was even possible. The tool surfaces data; to date, the analyst provides the judgment about what that data means. AI may end up automating some of this decision making. Time will tell.

### Email Headers Are Evidence
The sender display, email body, and branding can be misleading. Header analysis provides a more reliable path for identifying routing, originating IPs, authentication results, and infrastructure used to deliver the message.

Reading SMTP headers manually to count hops and identify originating IPs can tell a story. Each Received: line represents a trust handoff, and anomalies in those handoffs can be where the investigation gets some traction.

### OSINT Tools Support Triage
UrlScan.io, Abuse.ch, Cisco Talos, PhishTool, and VirusTotal each provide a different layer of visibility. Used together, they help an analyst validate suspicious activity, identify related indicators, and make better escalation decisions.

### Hash-Based Lookups Reduce Exposure
Computing a hash and querying it is generally safer than uploading a live sample to a public sandbox, because it reduces the chance of exposing the sample itself. This distinction matters when an adversary may have monitoring in place.

### Malware Family Identification Matters
Connecting the attachment to Dridex changed the investigation from a generic suspicious attachment to a known malware association linked to credential theft and banking issues.

### Defanging Is a Communication Discipline
Defanged IOCs (XX[.]XX[.]XXX[.]XX) are a standard hygiene practice across the threat intelligence spectrum. It is a small thing that prevents real-world harm and signals professional rigor.


## Key Competencies Demonstrated
- Threat intelligence fundamentals and framework application (Strategic, Technical, Tactical, Operational)
- OSINT-based investigation
- URL and domain profiling via UrlScan.io
- Malware and botnet IOC lookup across MalwareBazaar, FeodoTracker, SSL Blacklist, URLhaus, and ThreatFox
- Botnet C2 investigation
- JA3 fingerprint analysis for SSL-layer threat detection
- Phishing email header analysis using Thunderbird and raw message source
- IP defanging using CyberChef
- Multi-hop SMTP relay chain tracing
- SHA-256 and MD5 hash computation for file attribution
- VirusTotal file hash lookups and detection alias identification
- IP reputation and ownership attribution via Cisco Talos and ICANN WHOIS
- Multi-source intelligence interpretation, correlation and cross-referencing
- Analytical documentation

## Employer-Relevant Skills:
- Triaging suspicious emails end-to-end: header inspection, IOC extraction, hash lookup, and malware family attribution
- Using OSINT threat intelligence platforms efficiently under time pressure
- Identifying malware family associations from hashes, files, or infrastructure
- Reviewing email routing details and originating IP addresses
- Validating suspicious URLs, domains, and attachments
- Correlating IOC findings across multiple sources to build a coherent picture
- Recognizing when a tool's output requires a follow-on query to produce an actionable answer
- Documenting findings in a format that supports escalation and handoff
- Applying standard reporting hygiene (defanging, hash-based lookup, source attribution)
- Translating OSINT findings into SOC-ready conclusions
- Supporting escalation decisions with evidence (instead of assumptions)

## SOC Relevance: 
This lab maps directly to SOC analyst responsibilities because threat intelligence is part of everyday alert triage. Analysts frequently receive suspicious emails, domains, IPs, file hashes, or attachment names and must determine whether they represent real risk.

The room practiced the same workflow used in security operations:
- Extract the indicator.
- Enrich it with reliable sources.
- Compare the results across tools.
- Identify relationships between artifacts.
- Determine whether the activity is suspicious or malicious.
- Document the evidence clearly.

In a real SOC environment, this process supports faster triage, better escalation quality, and stronger incident response. It also helps reduce false positives because analysts are not relying on one tool or one artifact alone.

This room operationalized that capability across five platforms and three live email scenarios. The scenarios were not abstract; they involved realistic sender addresses, plausible pretexts (LinkedIn, a supercar dealership, a sales receipt), and actual malware families with documented histories. Working through them in sequence built the kind of pattern recognition that accelerates real-world triage.

The cumulative lesson: threat intelligence tools do not produce conclusions. They produce data points. The analyst assembles the data points into a defensible narrative that supports a decision — whether to escalate, contain, or close.

## HUMINT to SOC Translation: 
Threat intelligence works much like human-source reporting and is structurally identical to the source-assessment and collection-planning disciplines in competitive intelligence. A single report, contact, or observation may be useful, but it rarely tells the full story by itself. The real value comes from corroboration, source evaluation, pattern recognition, and context.

In this room, each indicator functioned like a fragment of reporting:

- A URL scan provided environmental context.
- A JA3 fingerprint connected traffic behavior to malware.
- An IP lookup provided ownership and reputation details.
- Email headers reconstructed delivery path.

This is precisely the multi-source correlation model used in professional intelligence collection, where the reliability of a single source is always subordinate to the convergence of independent lines of evidence. The same discipline that makes a CI analyst rigorous about source validation - never accepting a single-source finding as definitive, makes a SOC analyst effective at building attribution cases that hold up under scrutiny.

In both fields, the deliverable is not the data. It is the defensible assessment, supported by named sources, with explicit confidence levels and a clear chain of reasoning from evidence to conclusion.

Security teams need defensible narratives- not just tool output. A strong investigation explains what was found, where it was found, why it matters, the confidence level, and what action should be taken next.

## Author: 
[Malakh Fuller](https://www.linkedin.com/in/malakhfuller/)
