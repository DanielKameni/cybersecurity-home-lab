# VirtualBox Configuration & Virtual Machine Setup

## 1. Overview

This document describes the virtualization platform, virtual machine configurations, and network adapter settings used in the Cybersecurity Home Lab.

The lab is designed to provide a controlled environment for penetration testing, vulnerability assessment, and network security experiments.

The infrastructure is hosted on Ubuntu Linux using Oracle VirtualBox.

Two virtual machines are currently deployed:

- Kali Linux – Attacker machine
- Metasploitable 2 – Deliberately vulnerable target machine

The configuration separates internet access, penetration testing traffic, and communication with the Ubuntu host.

## 2. Virtualization Platform

| Property | Configuration |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Version | 7.2.22 r175245 |
| Host Operating System | Ubuntu Linux |
| Virtualization Type | Type-2 Hypervisor |
| Virtual Machines | Kali Linux, Metasploitable 2 |

Oracle VirtualBox is used to manage the virtual machines, allocate hardware resources, and configure virtual network adapters.

## 3. Virtual Machine Configuration

### 3.1 Kali Linux – Attacker Machine

**VirtualBox VM Name:** `kali-linux-2026.2-virtualbox-amd64`

Kali Linux serves as the primary attacker and security assessment workstation.

It provides tools for network reconnaissance, vulnerability assessment, penetration testing, and security analysis.

#### Hardware Configuration

| Property | Value |
|---|---|
| Guest OS Type | Debian (64-bit) |
| Distribution | Kali Linux |
| RAM | 3072 MB |
| Virtual CPUs | 2 |
| Firmware | BIOS |
| Active Network Adapters | 3 |

#### Network Adapter Configuration

| Adapter | Attachment Type | Network | IPv4 Address |
|---|---|---|---|
| NIC 1 | NAT | VirtualBox NAT | 10.0.2.15 |
| NIC 2 | Internal Network | LAB-PENTEST | 192.168.56.10 |
| NIC 3 | Host-Only Adapter | vboxnet0 | 192.168.100.10 |
| NIC 4–8 | Disabled | — | — |

All three active adapters use the Intel PRO/1000 MT Desktop (`82540EM`) virtual network adapter model.

#### Network Functions

**NIC 1 – NAT**

Provides outbound internet connectivity for operating system updates, software installation, and security tool maintenance.

**NIC 2 – Internal Network**

Connects Kali Linux to the isolated penetration testing network.

This interface is used for communication with deliberately vulnerable target machines.

**NIC 3 – Host-Only Network**

Enables communication between Kali Linux and the Ubuntu host.

It supports file transfers, report preparation, and the exchange of security assessment results.

### 3.2 Metasploitable 2 – Vulnerable Target

**VirtualBox VM Name:** `Metasploitable2`

Metasploitable 2 is an intentionally vulnerable Linux virtual machine used for controlled security testing.

It provides a target environment for network enumeration, vulnerability identification, and penetration testing exercises.

#### Hardware Configuration

| Property | Value |
|---|---|
| Guest OS Type | Ubuntu (32-bit) |
| RAM | 1024 MB |
| Virtual CPUs | 1 |
| Firmware | BIOS |
| Active Network Adapters | 1 |

#### Network Adapter Configuration

| Adapter | Attachment Type | Network | IPv4 Address |
|---|---|---|---|
| NIC 1 | Internal Network | LAB-PENTEST | 192.168.56.20 |
| NIC 2–8 | Disabled | — | — |

The active adapter uses the Intel PRO/1000 MT Desktop (`82540EM`) virtual network adapter model.

#### Security Configuration

Metasploitable is configured with only one active network adapter.

The adapter is connected to the VirtualBox Internal Network named `LAB-PENTEST`.

No NAT, bridged, or Host-Only adapter is attached to the target.

The operating system has no default route.

This configuration limits direct connectivity to the internal penetration testing network.

## 4. Virtual Network Architecture

The environment uses three network configurations.

| Network | VirtualBox Mode | Purpose |
|---|---|---|
| 10.0.2.0/24 | NAT | Kali internet connectivity |
| 192.168.56.0/24 | Internal Network | Penetration testing |
| 192.168.100.0/24 | Host-Only | Ubuntu–Kali communication |

### 4.1 Network Isolation

The Internal Network named `LAB-PENTEST` is used to isolate vulnerable targets from the physical network.

Metasploitable is attached exclusively to this network.

Kali Linux has access to the internal network through its second network adapter.

IPv4 forwarding on Kali Linux was verified to be disabled.

### 4.2 Host Communication

The Host-Only network `vboxnet0` connects Kali Linux to the Ubuntu host.

The host uses the IPv4 address `192.168.100.1`.

Kali Linux uses `192.168.100.10`.

Both addresses belong to the subnet `192.168.100.0/24`.

### 4.3 Internet Connectivity

Kali Linux uses VirtualBox NAT for outbound internet connectivity.

Metasploitable does not have a NAT adapter or a default gateway.

This configuration prevents ordinary direct IPv4 routing from Metasploitable to external networks.

## 5. Configuration Verification

Virtual machine settings were inspected from the Ubuntu host using the VirtualBox command-line management tool.

### 5.1 VirtualBox Version

```bash
VBoxManage --version
```

Observed result:

```text
7.2.22r175245
```

### 5.2 Kali Linux Configuration

```bash
VBoxManage showvminfo "kali-linux-2026.2-virtualbox-amd64" --details
```

The configuration confirms:

- 3072 MB RAM
- 2 virtual CPUs
- BIOS firmware
- Three active network adapters
- NAT, Internal Network, and Host-Only attachments

### 5.3 Metasploitable Configuration

```bash
VBoxManage showvminfo "Metasploitable2" --details
```

The configuration confirms:

- 1024 MB RAM
- 1 virtual CPU
- BIOS firmware
- One active network adapter
- Internal Network attachment
- All remaining network adapters disabled

## 6. Security Considerations

The following security principles are applied to the lab:

**Network Segmentation**

Vulnerable target systems are separated from the physical network through a dedicated VirtualBox Internal Network.

**Controlled Internet Access**

Internet connectivity is provided to Kali Linux through NAT.

Metasploitable has no direct internet connection.

**Reduced Network Exposure**

Unused virtual network adapters are disabled.

**Restricted Packet Forwarding**

IPv4 forwarding on Kali Linux was verified to be disabled.

**Controlled Testing**

Penetration testing activities are restricted to authorized laboratory systems.

## 7. Reproduction Guidelines

To reproduce the current environment:

1. Install Oracle VirtualBox on a compatible Linux host.
2. Import or create a Kali Linux virtual machine.
3. Configure Kali Linux with 3072 MB RAM and 2 virtual CPUs.
4. Attach three network adapters: NAT, Internal Network, and Host-Only.
5. Create a VirtualBox Internal Network named `LAB-PENTEST`.
6. Configure the Kali Linux internal interface with `192.168.56.10/24`.
7. Import a Metasploitable 2 virtual machine.
8. Configure Metasploitable with 1024 MB RAM and 1 virtual CPU.
9. Attach Metasploitable exclusively to `LAB-PENTEST`.
10. Configure Metasploitable with `192.168.56.20/24` and no default gateway.
11. Configure the Host-Only network `vboxnet0` using the subnet `192.168.100.0/24`.
12. Assign `192.168.100.1/24` to the Ubuntu host and `192.168.100.10/24` to Kali.
13. Verify network connectivity and routing.
14. Verify that IPv4 forwarding is disabled on Kali Linux.

The detailed installation and operating system configuration procedures are maintained in the dedicated setup documentation.

## 8. Future Improvements

Planned improvements include:

- Baseline VM snapshots
- Automated VM configuration verification
- Additional isolated target networks
- Windows and Active Directory laboratory environments
- Centralized security monitoring
- Network traffic capture and analysis

## 9. Conclusion

The VirtualBox configuration provides a controlled virtualization environment for cybersecurity training and security assessment activities.

The separation of network functions, restricted connectivity of vulnerable targets, and documented VM settings establish a foundation for reproducible security experiments.

Further improvements will focus on automation, monitoring, and additional security testing scenarios.
