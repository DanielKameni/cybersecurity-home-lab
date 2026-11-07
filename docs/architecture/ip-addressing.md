# IP Addressing & Routing Configuration

## 1. Overview

This document describes the IP addressing scheme, subnet configuration, and routing behavior of the Cybersecurity Home Lab.

The purpose of the network configuration is to provide controlled connectivity between virtual machines while maintaining network segmentation and limiting the exposure of deliberately vulnerable systems.

The lab consists of three network segments:

- NAT Network – Internet connectivity
- Internal Network – Penetration testing environment
- Host-Only Network – Communication with the Ubuntu host

Each network segment has a dedicated purpose and uses a separate IPv4 subnet.

## 2. IP Addressing Scheme

The following table summarizes the current IPv4 configuration.

| System | Interface | IPv4 Address | Subnet | Network Type |
|---|---|---|---|---|
| Kali Linux | eth0 | 10.0.2.15 | 10.0.2.0/24 | NAT |
| Kali Linux | eth1 | 192.168.56.10 | 192.168.56.0/24 | Internal |
| Kali Linux | eth2 | 192.168.100.10 | 192.168.100.0/24 | Host-Only |
| Metasploitable 2 | eth0 | 192.168.56.20 | 192.168.56.0/24 | Internal |
| Ubuntu Host | vboxnet0 | 192.168.100.1 | 192.168.100.0/24 | Host-Only |

All three networks use a `/24` subnet prefix, corresponding to the subnet mask `255.255.255.0`.

## 3. Network Segments

### 3.1 NAT Network

**Network:** `10.0.2.0/24`

**Kali Interface:** `eth0`

**Kali IPv4 Address:** `10.0.2.15`

**Default Gateway:** `10.0.2.2`

**VirtualBox Mode:** NAT

The NAT adapter provides outbound connectivity for Kali Linux.

It is primarily used for:

- Operating system updates
- Security tool installation
- Package management
- Vulnerability database updates

The NAT network is not intended for penetration testing against vulnerable lab targets.

### 3.2 Internal Network – LAB-PENTEST

**Network:** `192.168.56.0/24`

**VirtualBox Network Name:** `LAB-PENTEST`

**Kali Interface:** `eth1`

**Kali IPv4 Address:** `192.168.56.10`

**Metasploitable IPv4 Address:** `192.168.56.20`

The internal network provides communication between the Kali Linux attacker machine and the Metasploitable 2 target.

Metasploitable is configured with a single VirtualBox network adapter attached to `LAB-PENTEST`.

All additional network adapters are disabled.

This configuration limits direct network connectivity to virtual machines attached to the same internal network.

### 3.3 Host-Only Network

**Network:** `192.168.100.0/24`

**VirtualBox Interface:** `vboxnet0`

**Kali Interface:** `eth2`

**Kali IPv4 Address:** `192.168.100.10`

**Ubuntu Host IPv4 Address:** `192.168.100.1`

The Host-Only network enables communication between the Kali Linux virtual machine and the Ubuntu host.

Typical use cases include:

- File transfers
- SSH connections
- Exchange of scan results
- Security report preparation
- Project documentation

The Host-Only network is separate from the internal penetration testing network.

## 4. Routing Configuration

### 4.1 Kali Linux Routing Table

The following routing table was obtained using:

```bash
ip route
```

Observed output:

```text
default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.10 metric 101
192.168.100.0/24 dev eth2 proto kernel scope link src 192.168.100.10 metric 102
```

The routing table contains three directly connected networks and one default route.

The default route uses the NAT gateway `10.0.2.2`.

Traffic destined for `192.168.56.0/24` is transmitted through `eth1`.

Traffic destined for `192.168.100.0/24` is transmitted through `eth2`.

Traffic without a more specific matching route is sent through the default gateway via `eth0`.

### 4.2 Metasploitable 2 Routing Table

The routing configuration was inspected using:

```bash
route -n
```

Observed output:

```text
Destination     Gateway         Genmask         Flags   Iface
192.168.56.0    0.0.0.0         255.255.255.0   U       eth0
```

Metasploitable has a directly connected route to the internal penetration testing network.

No default gateway is configured.

As a result, Metasploitable cannot directly route traffic to destinations outside its local subnet using its current routing configuration.

## 5. Network Security Considerations

### 5.1 Network Segmentation

The lab separates network traffic according to its intended purpose.

The internal penetration testing network contains the deliberately vulnerable target system.

The Host-Only network supports communication between Kali Linux and the Ubuntu host.

The NAT adapter provides internet connectivity to Kali Linux without requiring a direct internet connection for Metasploitable.

### 5.2 IP Forwarding

IPv4 forwarding on Kali Linux was checked using:

```bash
sysctl net.ipv4.ip_forward
```

Observed result:

```text
net.ipv4.ip_forward = 0
```

This indicates that IPv4 forwarding was disabled at the time of verification.

Consequently, Kali Linux was not configured to forward IPv4 packets between its network interfaces.

### 5.3 Internet Isolation

Metasploitable has no NAT or bridged network adapter.

Its only active VirtualBox network adapter is connected to the internal network `LAB-PENTEST`.

Furthermore, the target has no default route.

These controls limit direct network exposure and prevent ordinary IPv4 routing to external networks under the verified configuration.

### 5.4 Security Limitations

The current verification focuses primarily on IPv4 connectivity and VirtualBox network configuration.

Additional security checks may include:

- IPv6 configuration and forwarding
- Host firewall rules
- Network service exposure
- Changes to VirtualBox network settings
- Verification of network isolation after configuration changes

## 6. Network Verification

The following verification activities have been completed.

| Test ID | Test Description | Expected Result | Actual Result | Status |
|---|---|---|---|---|
| NET-001 | Kali to Metasploitable connectivity | Reachable | Reachable | PASS |
| NET-002 | Metasploitable to Kali connectivity | Reachable | Reachable | PASS |
| NET-003 | Metasploitable to Host-Only network | Unreachable | Network unreachable | PASS |
| NET-004 | Kali IPv4 forwarding | Disabled | Disabled | PASS |
| NET-005 | Metasploitable default gateway | Not configured | Not configured | PASS |
| NET-006 | Metasploitable VirtualBox adapter | Internal Network only | Internal Network only | PASS |
| NET-007 | Ubuntu Host-Only IP configuration | Address in 192.168.100.0/24 | 192.168.100.1/24 | PASS |


The results support the intended IPv4 network configuration and segmentation design.

However, successful verification of these controls does not constitute a comprehensive network security assessment.

## 7. Conclusion

The Cybersecurity Home Lab uses a structured IPv4 addressing scheme with three dedicated network segments.

The routing configuration supports communication between the attacker and target machines while limiting the target system's direct connectivity to external networks.

The documented configuration provides a foundation for future penetration testing, vulnerability assessment, and security monitoring projects.
