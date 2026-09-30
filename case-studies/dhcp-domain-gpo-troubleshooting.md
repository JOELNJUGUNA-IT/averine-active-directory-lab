

```markdown
# Case Study: Domain Login Failure, DHCP Recovery & GPO Drive Mapping

## Lab Environment

- Domain: `cyberlab.local`
- Domain Controller: `DC01`
- DC01 IP: `192.168.157.10`
- Client: `CLIENT01`
- DHCP Scope: `192.168.157.100–200`
- Test User: `CYBERLAB\testuser`
- Hypervisor: VMware Workstation

---

## Problem

CLIENT01 was unable to authenticate normally with the CYBERLAB domain.

Windows displayed a message indicating that the domain was unavailable.

The objective was to troubleshoot the problem systematically rather than immediately changing DNS, DHCP, or Group Policy settings.

---

## 1. Verify the Logged-In User

After gaining access to CLIENT01, I verified the account with:

```cmd
whoami
```

Example:

```text
cyberlab\david.finance
```

This confirmed the identity being used during troubleshooting.

---

## 2. Check Network Configuration

I ran:

```cmd
ipconfig /all
```

CLIENT01 had an address in the `169.254.x.x` range.

This is an APIPA address.

### Interpretation

APIPA indicated that CLIENT01 had failed to obtain an IPv4 lease from DHCP.

This shifted the investigation toward DHCP and network connectivity before troubleshooting DNS or Group Policy.

---

## 3. Test Domain Controller Connectivity

I tested name-based communication:

```cmd
ping DC01
```

The initial test failed.

I then tested the DC directly:

```cmd
ping 192.168.157.10
```

Communication also failed.

This showed that the problem was not simply DNS name resolution. CLIENT01 had a more fundamental network/DHCP connectivity problem.

---

## 4. Investigate DHCP

On DC01 I verified:

- DHCP service was running.
- DHCP service startup was automatic.
- DHCP scope was active.
- Scope network was `192.168.157.0/24`.
- Address pool was `192.168.157.100–200`.
- Available leases existed.

I also checked VMware networking to make sure the client and server were connected to the appropriate virtual network.

---

## 5. Request a Fresh DHCP Lease

On CLIENT01 I used:

```cmd
ipconfig /release
ipconfig /renew
```

CLIENT01 successfully received an address in the correct network:

```text
192.168.157.x/24
```

The default gateway was:

```text
192.168.157.2
```

The APIPA condition was resolved.

---

## 6. Validate DC Connectivity

I tested:

```cmd
ping DC01
```

Result:

```text
Reply from 192.168.157.10
```

The test completed with successful replies and no packet loss.

This proved CLIENT01 could communicate with DC01 again.

---

## 7. Validate DNS

I ran:

```cmd
nslookup DC01
```

The result resolved:

```text
DC01.cyberlab.local
192.168.157.10
```

This confirmed that CLIENT01 was using the domain DNS infrastructure successfully.

---

## 8. Validate User and Group Membership

I signed in using the IT test account:

```text
CYBERLAB\testuser
```

I verified identity:

```cmd
whoami
```

Then checked the user's security groups:

```cmd
whoami /groups
```

The account showed the expected IT-related group membership.

This was important because:

- The OU determines where the GPO applies.
- Security groups determine authorization to resources.

---

## 9. Validate Group Policy

I ran:

```cmd
gpresult /r
```

Under Applied Group Policy Objects I confirmed:

```text
LAB-IT-Drive-Mapping
```

This proved that the drive-mapping GPO successfully reached the IT test user.

---

## 10. Validate the Mapped Drive

I ran:

```cmd
net use
```

The result showed:

```text
Status    Local    Remote
OK        I:       \\DC01\IT-Shared
```

The drive mapping had succeeded.

---

## 11. Validate Resource Access

Finally:

```cmd
I:
dir
```

The directory displayed:

```text
GPO-Drive-Test.txt
```

This proved more than simply seeing the mapped drive.

It demonstrated that the user could successfully access the underlying shared resource.

---

## Troubleshooting Chain

```text
Domain login problem
        ↓
ipconfig /all
        ↓
169.254.x.x
        ↓
APIPA
        ↓
Investigate DHCP/network
        ↓
DHCP service running
Scope active
Range valid
        ↓
ipconfig /release
ipconfig /renew
        ↓
Valid 192.168.157.x address
        ↓
ping DC01
        ↓
SUCCESS
        ↓
nslookup DC01
        ↓
SUCCESS
        ↓
whoami /groups
        ↓
Validate authorization
        ↓
gpresult /r
        ↓
LAB-IT-Drive-Mapping applied
        ↓
net use
        ↓
I: → \\DC01\IT-Shared
        ↓
dir
        ↓
Resource accessible
```

## Key Lessons

1. APIPA (`169.254.x.x`) is an important clue that DHCP failed.
2. Do not immediately blame DNS when the client does not have valid IP configuration.
3. Test IP connectivity separately from DNS name resolution.
4. A DHCP scope existing does not automatically prove that clients are successfully receiving leases.
5. A GPO existing does not prove that it reached the user; verify with `gpresult /r`.
6. A mapped drive appearing does not prove resource authorization; test actual access.
7. Troubleshooting should move from evidence to evidence instead of guessing fixes.

## Result

CLIENT01 regained network and domain-controller communication, DNS resolution succeeded, the IT drive-mapping GPO applied successfully, and the test user could access `\\DC01\IT-Shared` through the mapped `I:` drive.

**Status: Resolved and validated.**
```

