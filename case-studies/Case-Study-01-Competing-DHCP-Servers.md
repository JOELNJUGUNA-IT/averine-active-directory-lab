# Case Study 01: CLIENT01 Receiving DHCP and DNS From the Wrong Server

## Overview

While validating my `cyberlab.local` Windows Server home lab, I discovered that CLIENT01 was receiving network configuration from VMware’s built-in DHCP service instead of the DHCP service running on DC01.

This created a risk of unreliable domain authentication, DNS resolution and Group Policy processing.

## Lab Environment

* Hypervisor: VMware Workstation Pro
* Domain controller: DC01
* Server operating system: Windows Server 2025 Standard Evaluation
* Domain: `cyberlab.local`
* DC01 address: `192.168.157.10`
* Client: CLIENT01 running Windows 11
* Network: VMware NAT on `VMnet8`
* Domain user used for testing: `cyberlab\david.finance`

## Reported Configuration

I ran:

```cmd
ipconfig /all
```

CLIENT01 initially displayed:

```text
IPv4 Address:    192.168.157.131
Default Gateway: 192.168.157.2
DHCP Server:     192.168.157.254
DNS Server:      192.168.157.2
```

The gateway was correct, but the DHCP and DNS servers were not consistent with the intended domain design.

## Expected Configuration

CLIENT01 should receive the following settings:

```text
DHCP Server: 192.168.157.10
DNS Server:  192.168.157.10
Gateway:     192.168.157.2
```

DC01 must provide DHCP and internal DNS services, while the VMware NAT gateway provides external network access.

## Investigation

I opened the DHCP management console on DC01 and verified that:

* The `192.168.157.0` DHCP scope was active.
* DHCP option 003 Router was set to `192.168.157.2`.
* DHCP option 006 DNS Servers was set to `192.168.157.10`.

Because the Windows DHCP scope and options were correct, I inspected VMware’s Virtual Network Editor.

VMnet8 was configured as a NAT network, but VMware’s local DHCP service was also enabled. Therefore, both VMware and DC01 were capable of answering DHCP requests.

## Root Cause

The network had two active DHCP servers:

1. VMware DHCP at `192.168.157.254`
2. Windows Server DHCP on DC01 at `192.168.157.10`

VMware DHCP answered CLIENT01 and supplied the VMware NAT address as DNS. This prevented CLIENT01 from consistently using DC01 for domain DNS resolution.

## Resolution

In VMware Virtual Network Editor, I selected VMnet8 and disabled:

```text
Use local DHCP service to distribute IP addresses to VMs
```

I kept VMnet8 configured as NAT and did not change its subnet or gateway.

On CLIENT01, I then ran:

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

## Validation

After renewing the lease, CLIENT01 displayed:

```text
IPv4 Address:    192.168.157.131
Default Gateway: 192.168.157.2
DHCP Server:     192.168.157.10
DNS Server:      192.168.157.10
```

I tested internal name resolution with:

```cmd
nslookup dc01.cyberlab.local
```

The result confirmed:

```text
Server:  dc01.cyberlab.local
Address: 192.168.157.10

Name:    dc01.cyberlab.local
Address: 192.168.157.10
```

![CLIENT01 DHCP and DNS validation](../Screenshots/04-CLIENT01-DHCP-DNS-Validation.png)

## Result

CLIENT01 now receives its DHCP lease and DNS configuration from DC01 while continuing to use the VMware NAT gateway for external network access.

This provides the correct foundation for:

* Active Directory authentication
* Internal DNS resolution
* Group Policy processing
* Domain-controller discovery
* Access to domain resources

## Skills Demonstrated

* Windows Server DHCP administration
* DHCP scope-option verification
* DNS configuration validation
* VMware NAT networking
* Identification of competing DHCP services
* Windows command-line troubleshooting
* Root-cause analysis
* Post-fix verification

## Lesson Learned

An IP address within the expected subnet does not prove that the correct DHCP server supplied it. The `DHCP Server` and `DNS Servers` fields in `ipconfig /all` must also be checked.

In an Active Directory environment, domain clients should use the domain DNS server rather than a public DNS server or the NAT gateway.
