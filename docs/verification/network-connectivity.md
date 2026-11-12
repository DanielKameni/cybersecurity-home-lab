# Network Connectivity Verification Report

## 1. Overview

This document describes the network connectivity verification activities performed in the Cybersecurity Home Lab.

The objective is to validate the intended communication paths between the Ubuntu host, Kali Linux attacker machine, and Metasploitable 2 target system.

The verification focuses on IPv4 addressing, routing, ICMP connectivity, internet access, and DNS resolution.

## 2. Test Environment

| System | Role | IPv4 Address |
|---|---|---|
| Ubuntu Host | Virtualization Host | 192.168.100.1 |
| Kali Linux | Attacker | 192.168.56.10 / 192.168.100.10 / 10.0.2.15 |
| Metasploitable 2 | Vulnerable Target | 192.168.56.20 |

The environment uses Oracle VirtualBox with NAT, Internal Network, and Host-Only network configurations.

## 3. Test Methodology

The verification process follows these steps:

1. Confirm that the required virtual machines are running.
2. Inspect the network interface configuration.
3. Verify routing information.
4. Test communication between authorized lab systems.
5. Verify external connectivity through the Kali NAT interface.
6. Verify DNS resolution.
7. Record the results and supporting evidence.

A test is marked as PASS when the observed result meets the defined acceptance criteria.

A test is marked as FAIL when the observed result contradicts the acceptance criteria.

A test is marked as PARTIAL when only some of its acceptance criteria are satisfied.

Tests that have not yet been executed are marked as NOT TESTED.

## 4. Test Results Summary

| Test ID | Source | Destination | Test Method | Observed Result | Status |
|---|---|---|---|---|---|
| NET-CON-001 | Kali | Metasploitable | ICMP Echo | 4/4 packets received | PASS |
| NET-CON-002 | Metasploitable | Kali | ICMP Echo | 4/4 packets received | PASS |
| NET-CON-003 | Kali | Ubuntu Host | ICMP Echo | 4/4 packets received | PASS |
| NET-CON-004 | Ubuntu Host | Kali | ICMP Echo | 4/4 packets received | PASS |
| NET-CON-005 | Kali | NAT Gateway | ICMP / Route Inspection | Gateway reachable; route verified | PASS |
| NET-CON-006 | Kali | Internet | ICMP / HTTPS / TLS | successful | PASS |
| NET-CON-007 | Kali | DNS Resolver | DNS Lookup | IPv4 resolution successful | PASS |

## 5. Detailed Verification Results

### NET-CON-001 – Kali to Metasploitable

**Objective:** Verify connectivity from Kali Linux to Metasploitable.

**Command:**

`ping -c 4 192.168.56.20`

**Expected Result:** Metasploitable responds to ICMP Echo Requests.

**Observed Result:** Four packets transmitted and four packets received, with 0% packet loss.

**Average RTT:** 0.583 ms.

**Status:** PASS.

### NET-CON-002 – Metasploitable to Kali

**Objective:** Verify connectivity from Metasploitable to Kali Linux.

**Command:**

`ping -c 4 192.168.56.10`

**Expected Result:** Kali responds to ICMP Echo Requests.

**Observed Result:** Four packets transmitted and four packets received, with 0% packet loss.

**Average RTT:** 0.864 ms.

**Status:** PASS.

### NET-CON-003 – Kali to Ubuntu Host

**Objective:** Verify communication through the Host-Only network.

**Command:**

`ping -c 4 192.168.100.1`

**Expected Result:** Ubuntu responds to ICMP Echo Requests.

**Observed Result:** Four packets transmitted and four packets received, with 0% packet loss.

**Average RTT:** 0.481 ms.

**Status:** PASS.

### NET-CON-004 – Ubuntu Host to Kali

**Objective:** Verify communication from the Ubuntu host to Kali.

**Command:**

`ping -c 4 192.168.100.10`

**Expected Result:** Kali responds to ICMP Echo Requests.

**Observed Result:** 4/4 packets received.

**Status:** PASS.

### NET-CON-005 – NAT Gateway Connectivity

**Objective:** Verify the configured route to the NAT gateway.

**Commands:**

`ping -c 4 10.0.2.2`

`ip route get 1.1.1.1`

**Expected Result:** Kali selects the NAT interface and gateway for external destinations.

**Observed Result:** The gateway responded to all four ICMP Echo Requests. Route inspection confirmed that external IPv4 traffic uses gateway `10.0.2.2` through interface `eth0`.

**Status:** PASS.

### NET-CON-006 – Internet Connectivity

**Objective:** Verify external connectivity through the NAT interface.

**Commands:**

`ping -c 4 1.1.1.1`

`curl -I --connect-timeout 5 https://example.com`

**Expected Result:** Successful external network communication through ICMP and HTTPS.

**Observed Result:** ICMP connectivity to `1.1.1.1` was successful with 0% packet loss. The HTTPS request succeed with
HTTP/2 200
SSL certificate verified via OpenSSL.
SSL connection using TLSv1.3
Established connection to example.com

**Status:** PASS.

**Follow-up:** Investigate DNS resolver behavior and repeat HTTPS connectivity testing.

### NET-CON-007 – DNS Resolution

**Objective:** Verify domain name resolution.

**Command:**

`getent ahostsv4 example.com`

**Expected Result:** The domain name resolves to an IPv4 address.

**Observed Result:** Multiple IPv4 addresses were successfully returned.

**Status:** PASS.

**Note:** Successful resolution during this test does not exclude intermittent DNS failures.

HTTPS Connectivity Investigation

During the initial verification, an HTTPS request to example.com failed with a DNS resolution timeout.

Further troubleshooting included:

Inspection of the DNS resolver configuration

Repeated HTTPS connectivity testing

IPv4-only connection testing

Explicit DNS resolution using curl --resolve

TLS certificate verification

The subsequent tests completed successfully.

The HTTPS connection negotiated TLS 1.3, the server certificate was successfully validated, and the server returned HTTP status code 200.

The initial DNS timeout could not be reproduced during the follow-up investigation. Its root cause remains undetermined.

Final Status: PASS


## 6. Evidence

Supporting command outputs and screenshots are stored in:

`evidence/command-outputs/`

`evidence/screenshots/`

Each evidence file should be associated with a specific test ID.

## 7. Assessment

All seven planned network connectivity tests were completed successfully.

The results confirm:

Bidirectional ICMP connectivity between Kali Linux and Metasploitable 2.

Bidirectional ICMP connectivity between Kali Linux and the Ubuntu host.

Successful connectivity to the VirtualBox NAT gateway.

Successful external IPv4 connectivity.

Successful DNS resolution.

Successful HTTPS communication with TLS certificate verification.

The observed results demonstrate that the configured network communication paths are operational.

These tests validate connectivity but do not independently establish complete network isolation or overall system security.

## 8. Conclusion

The Cybersecurity Home Lab successfully passed all seven defined network connectivity verification tests.

The environment is ready for the next verification phase, focusing on network segmentation, restricted communication paths, 
and security controls.
