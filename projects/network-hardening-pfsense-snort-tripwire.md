---
layout: default
title: GNS3 Network Hardening Lab
---

# GNS3 Network Hardening Lab: pfSense, Snort, and Tripwire

## Overview

This project highlights a security-focused GNS3 lab built to practice network and system hardening using pfSense, Snort, and Tripwire. The lab was designed to simulate an enterprise-style environment in which firewall rules, network segmentation, intrusion detection, and host-based integrity monitoring could be configured and tested in a controlled setting. Through this project, I gained hands-on experience with defensive security concepts including traffic filtering, DMZ design, NAT, intrusion detection and prevention, and host-based file integrity monitoring.

## Objectives

- Build a virtual enterprise security lab in GNS3
- Configure pfSense to segment and secure internal network traffic
- Implement DMZ and NAT rules for controlled access
- Deploy Snort for network intrusion detection and prevention
- Use Tripwire as a host-based intrusion detection system
- Practice validating security controls through testing and monitoring

## Lab Environment

This lab was built in GNS3 and included multiple systems to simulate an enterprise-style network. Key components included:

- **GNS3** for topology design and lab simulation
- **pfSense** as the firewall and routing platform
- **Snort** for network intrusion detection and prevention
- **Tripwire** for host-based integrity monitoring
- Internal LAN, management, DMZ, and internet-connected segments
- Windows and Linux systems used for testing connectivity and security controls

## Key Tasks Completed

Across this lab, I completed security-focused tasks such as:

- Configured pfSense interfaces and services
- Set up DHCP services within the pfSense environment
- Verified connectivity between hosts in the LAN and management networks
- Created and tested firewall and DMZ rules
- Configured NAT rules to manage traffic flow between internal and external networks
- Verified access to internal services from the internet test host where appropriate
- Implemented Snort to generate alerts and block suspicious traffic
- Observed Snort alert notifications and verified blocking behavior against test traffic
- Used Tripwire to establish a baseline database and detect file integrity changes
- Verified Tripwire scans under both normal conditions and policy violation scenarios
- Configured a scheduled Tripwire job through crontab

## Skills Demonstrated

This project helped me build and demonstrate skills in:

- Network hardening
- Firewall administration
- pfSense configuration
- Network segmentation and DMZ design
- NAT and access control
- Intrusion detection and prevention
- Host-based intrusion detection
- Security monitoring and validation
- Troubleshooting and documentation

## Project Walkthrough

### 1. GNS3 Workspace and Lab Topology

The lab began with building the security environment in GNS3. This provided the overall layout for the protected network and made it possible to visualize how pfSense, internal hosts, management systems, and internet-facing resources interacted.

![GNS3 Workspace](../assets/photo_assets/network-hardening-topology.png)

This topology formed the foundation for later phases of the project, including internal hardening, internet access control, intrusion detection, and host integrity monitoring.

### 2. pfSense Internal Network Configuration

After the topology was in place, pfSense was configured to support the internal environment. This included enabling services such as DHCP so that systems inside the lab could communicate using correct addressing and network settings.

You can add your screenshot here showing the pfSense services dashboard after DHCP setup.

```md
![pfSense Services Dashboard](../assets/photo_assets/pfsense-services-dashboard.png)
