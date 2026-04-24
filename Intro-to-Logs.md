# TryHackMe: Intro to Logs
- Room: https://tryhackme.com/room/introtologs
- Completed: 2026-04-15  
- Author: Malakh Fuller

## Room Objective
Understand foundational Linux logging concepts, including log sources, log formats, log collection, and log rotation. From a SOC perspective, these skills matter because logs are primary evidence for detection, alert validation, and incident reconstruction.

## Tools and Technologies
- Tools: cat, grep, nano
- Technologies: Linux, NGINX, rsyslog, logrotate
- Prerequisites: Security+, TryHackMe Junior Security Analyst Intro

## Summary
This hands-on lab introduced core Linux logging concepts with direct relevance to SOC operations. I examined web server and system logs, identified log formats, investigated suspicious browser activity, configured rsyslog for targeted log collection, and implemented log rotation with tamper-evident hashing.

The room reinforced how logs support detection, triage, and incident investigation, and how configuration accuracy directly affects visibility.

## Methodology

### 1. Identifying the Correct Log Source
I began by locating the GitLab NGINX access log. After confirming the file path (/var/log/gitlab/nginx/access.log), I identified the format as NCSA Combined Log Format, which includes:

- HTTP request  
- Status code  
- Referrer  
- User-agent  

Understanding the format was essential for interpreting suspicious browser activity.

### 2. Configuring rsyslog for SSH Logging

My initial configuration failed:

    :programname, isequal, "sshd" /var/log/websrv-02/rsyslog_sshd.log

Switching to a prefix match resolved the issue:

    :programname, startswith, "sshd" /var/log/websrv-02/rsyslog_sshd.log

This highlighted how small syntax differences can determine whether critical security events are captured.

### 3. Implementing Log Rotation With Integrity Hashing

I created a logrotate configuration:

    /var/log/websrv-02/rsyslog_sshd.log {
        daily
        rotate 30
        compress
        lastaction
            DATE=$(date +"%Y-%m-%d")
            echo "$(date)" >> "/var/log/websrv-02/hashes_${DATE}_rsyslog_sshd.txt"
            for i in $(seq 1 30); do
                FILE="/var/log/websrv-02/rsyslog_sshd.log.$i.gz"
                if [ -f "$FILE" ]; then
                    HASH=$(/usr/bin/sha256sum "$FILE" | awk '{ print $1 }')
                    echo "rsyslog_sshd.log.$i.gz $HASH" >> "/var/log/websrv-02/hashes_${DATE}_rsyslog_sshd.txt"
                fi
            done
            systemctl restart rsyslog
        endscript
    }

This produced a tamper-evident SHA-256 hash file for each rotation, providing a practical defense against log manipulation.

### 4. Confusion Around the 99-websrv-02 File
The room unexpectedly referenced /etc/logrotate.d/99-websrv-02_cron.conf.

The correct answers were:

- Retention: 24 compressed copies  
- Frequency: hourly  

This configuration rotates once per hour, keeping 24 hours of history. This was separate from the SSH log rotation I configured and reinforced the importance of validating file paths and sources before answering.

## Key Findings

### Log Format Matters
Correct interpretation depends on understanding structure. Recognizing Combined format clarified what fields were available for analysis.

### Common Log Types
- Application Logs  
- Audit Logs  
- Security Logs  
- Server Logs  
- System Logs  
- Network Logs  
- Database Logs  
- Web Server Logs  

### Log Collection Principles
- Identify all relevant sources  
- Maintain NTP synchronization  
- Centralize logs  
- Monitor high-priority events  
- Ensure retention supports investigations  

### Source Validation Is Critical
Switching between 98- and 99- config files demonstrated how easy it is to misinterpret a question if the referenced artifact isn’t confirmed.

### Small Syntax Issues Can Break Logging
The isequal vs startswith issue was a real-world example of how fragile log pipelines can be.

## Key Competencies Demonstrated
- Linux log analysis  
- Web server log review  
- Log source identification  
- Log format recognition
- Basic detection-minded investigation
- rsyslog configuration
- logrotate configuration and validation
- Command-line troubleshooting
- Analytical documentation
- Security operations fundamentals

## Employer-Relevant Skills:
- Reviewing logs for suspicious activity
- Understanding how authentication and web logs support investigations
- Using Linux command-line tools to inspect and filter data
- Troubleshooting configuration issues when expected logging behavior fails
- Understanding how log retention affects visibility and investigative depth
- Connecting technical findings to incident response and security monitoring workflows

## SOC Relevance: 
This lab maps well to real-world SOC responsibilities because logs are central to detection and investigation. They help answer the questions analysts ask during triage and incident response: what happened, when it happened, where it happened, who initiated it, whether it succeeded, and what the impact was.

In practice, a SOC analyst must be able to identify relevant logs, interpret them correctly, understand how they are collected, and know whether retention settings support a meaningful investigation. This lab gave me more hands-on exposure to all of those areas.

## HUMINT to SOC Translation: 
In cybersecurity, logs function much like human‑source reporting in competitive intelligence: a single data point rarely tells the full story. The real value emerges when individual log entries are aggregated, correlated, and interpreted within a broader operational context. When cross‑referenced with additional data or intelligence sources, logs become a high‑fidelity reconstruction tool capable of revealing intent, sequence, and impact.

Through contextual correlation, log data can answer the same foundational questions that drive any structured investigation:

- What occurred? (the observable activity or event)
- When did it occur? (temporal sequencing and dwell time)
- Where did it occur? (systems, endpoints, or network segments affected)
- Who initiated the activity? (user, process, or external actor attribution)
- Were their actions successful? (validation of outcomes and access attempts)
- What was the impact? (resulting changes, failures, or follow‑on activity)

Just as in CI work, the strength of log analysis lies not in isolated facts, but in the synthesis of multiple signals into a coherent, defensible narrative that supports decision‑making.

## Author: 
[Malakh Fuller]([https://example.com](https://www.linkedin.com/in/malakhfuller/))
