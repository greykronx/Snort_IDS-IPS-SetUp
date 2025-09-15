# SNORT IDS/IPS - Implementation Guide

This repository contains step-by-step instructions and commands to install, configure, and test Snort (IDS/IPS) on a Linux system. The instructions are written for an administrator who wants to build Snort from source (Snort 2.x style with DAQ) and deploy community rules for testing. The original notes were collected and consolidated from a working lab document. :contentReference[oaicite:0]{index=0}

> **Note:** Snort evolves (Snort 2.x vs Snort 3). Check the official Snort site for the latest downloads and instructions; some filenames and ruleset packages have changed over time. Official Snort resources: https://www.snort.org. :contentReference[oaicite:1]{index=1}

---

## What’s included

- `INSTALL_STEPS.md` — Complete sequence of commands with explanations (copy/paste friendly).
- `README.md` — This file.
- Example local.rules and instructions to test alerts.

---

## Supported distributions

- Debian / Ubuntu / Kali (apt-based)
- CentOS / RedHat / RHEL / Alma / Rocky (yum/dnf-based)

Commands vary slightly between distros — the step file has both apt/yum variants where needed.

---

## Requirements / Prerequisites

- A Linux machine (VM or bare metal) with a working internet connection.
- `sudo` or root access.
- Recommended: a snapshot / VM snapshot before you begin so you can revert if needed.
- If you plan to download subscriber rules you require a Snort.org account and Oinkcode; community rules are freely available. :contentReference[oaicite:2]{index=2}

---

## High-level flow

1. Update system packages
2. Install build dependencies and packet libraries
3. Download, compile and install DAQ (Data Acquisition library)
4. Download, compile and install Snort
5. Create snort user/groups and directories, set permissions
6. Install community rules and configure `snort.conf`
7. Test Snort in test mode and live monitoring mode
8. Optional: timezone, NTP, swapfile, OpenVPN & firewall steps (extra items in notes)

---

## Important notes

- The example commands in `INSTALL_STEPS.md` reference `daq-2.0.7` and Snort 2.x style files — verify the exact version you want to install from snort.org before running commands. :contentReference[oaicite:3]{index=3}
- Community rules archive names and top-level directories have changed over time (for example `snort3-community-rules.tar.gz` vs older names). If rule copying fails, inspect the extracted directory name. :contentReference[oaicite:4]{index=4}
- To download certain rule sets or subscriber rules you may need to register at Snort.org and use an Oinkcode. :contentReference[oaicite:5]{index=5}
- Use the `-T` test option for `snort` to verify configuration before running in production.

---

## Testing

- Start Snort in console alert mode on your interface and generate traffic (e.g., `ping`) to see ICMP alerts.
- View binary logged alerts (unified2) with tools such as `barnyard2` or `snort -r` depending on the log format. See `INSTALL_STEPS.md` for examples.
---

