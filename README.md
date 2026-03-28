# IP Blocker

A lightweight **host-based intrusion detection and mitigation system** designed to detect SSH brute-force attacks and automatically block malicious IP addresses using firewall rules.

The system analyzes authentication failures from the systemd journal, applies multiple detection rules to identify attack patterns, and mitigates threats by blocking attacker IP addresses while preserving access for trusted users through whitelist protection.

---

## Table of Contents

* Features
* Project Structure
* System Flowchart
* How It Works
* Installation
* Usage
* Configuration
* Whitelist
* Uninstallation
* Security Benefits
* Future Improvements
* License
* Authors

---

# Features

* SSH brute-force attack detection
* Multi-rule attack detection engine
* Automatic IP blocking using **UFW firewall rules**
* Whitelist protection for trusted IPs
* Journal-based log analysis using `journalctl`
* Modular architecture using helper scripts
* Automated reporting and logging
* Easy installation and uninstallation scripts

---

# Project Structure

```id="3f27hi"
ipblocker/
│
├── helpers/
│   ├── attack_detector.sh    # Multi-rule attack detection logic
│   ├── backup.sh             # Backup of configuration and data
│   ├── logger.sh             # Structured logging system
│   ├── notifier.sh           # Notification handling
│   └── reporter.sh           # Attack reports and summaries
│
├── docs/
│   └── ip_blocker_flowchart.png   # System workflow diagram
│
├── config.conf               # Main configuration file
├── whitelist.conf            # Trusted IP whitelist
│
├── ip_blocker.sh             # Main execution script
├── setup.sh                  # Installation script
└── uninstall.sh              # Removal script
```

---

# System Flowchart

The following diagram illustrates the operational workflow of the **IP Blocker system**, including log analysis, rule-based attack detection, whitelist verification, and automated mitigation.

![IP Blocker Flowchart](docs/ip_blocker_flowchart.png)

---

# How It Works

## 1. Log Monitoring

The system reads SSH authentication events from the **systemd journal** using `journalctl`. It performs incremental analysis by processing only new log entries generated since the last execution.

Each failed authentication event produces a structured record:

```id="p6bxcl"
(ip_address, username, timestamp)
```

---

## 2. Attack Detection

The detection engine applies **four rule-based techniques** to identify suspicious activity.

### Global Failure Monitor

Detects large-scale brute-force attacks by monitoring the total number of authentication failures within a short time window.

Default threshold:

```id="ovgk7t"
100 failures within 5 minutes
```

---

### Username Correlation Detector

Detects **username spraying attacks**, where multiple IP addresses attempt authentication using the same username.

Default detection rule:

```id="ld5wta"
More than 5 IPs targeting the same username
```

---

### Connection Failure Ratio Analyzer

Evaluates the ratio between failed and successful authentication attempts.

Default threshold:

```id="tfh7o5"
Failure ratio > 90%
```

This helps detect automated password-guessing tools.

---

### Extended Time Window Detector

Identifies **slow and persistent attacks** that attempt to bypass burst-based detection by spreading login attempts over longer periods.

Default threshold:

```id="z51baw"
More than 300 failures per hour
```

---

## 3. Whitelist Verification

Before blocking any IP address, the system checks the `whitelist.conf` file.

Whitelisted IP addresses are ignored to prevent disruption of legitimate administrative access.

---

## 4. Automated Mitigation

Malicious IP addresses are automatically blocked using **UFW firewall rules**.

All mitigation actions are recorded in logs for auditing and monitoring purposes.

---

# Installation

Run the setup script:

```bash id="hq6hvh"
sudo bash setup.sh
```

The installer will:

* Configure the system
* Create required directories
* Initialize configuration files
* Prepare firewall rules

---

# Usage

Run the system manually:

```bash id="6nmbgg"
sudo bash ip_blocker.sh
```

For automated protection, schedule execution using **cron**.

Example cron job (runs every 5 minutes):

```bash id="t7zfa3"
*/5 * * * * root /opt/ipblocker/ip_blocker.sh
```

---

# Configuration

Main configuration settings are stored in `config.conf`.

| Setting                    | Default | Description                                      |
| -------------------------- | ------- | ------------------------------------------------ |
| THRESHOLD                  | 3       | Failed attempts before blocking IP               |
| GLOBAL_FAILURE_THRESHOLD   | 100     | Failures in 5 minutes to trigger alert           |
| USER_CORRELATION_THRESHOLD | 5       | Unique IPs per username to trigger               |
| CONNECTION_RATIO_THRESHOLD | 90      | Failure percentage to flag abnormal activity     |
| EXTENDED_WINDOW_THRESHOLD  | 300     | Failures per hour for sustained attack detection |

---

# Whitelist

Trusted IP addresses can be added to `whitelist.conf`.

Example:

```id="xwce9s"
192.168.1.10
203.0.113.5
10.0.0.0/8
```

These addresses will **never be blocked** by the system.

---

# Uninstallation

To remove the system:

```bash id="sm9iw9"
sudo bash uninstall.sh
```

This will remove installed files and firewall rules created by **IP Blocker**.

---

# Security Benefits

* Prevents automated SSH brute-force attacks
* Detects distributed attack campaigns
* Protects servers with minimal resource usage
* Provides automated mitigation without manual firewall management

---





