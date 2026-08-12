# MTU and Path MTU Troubleshooting Playbook

## Overview

Maximum Transmission Unit (MTU) defines the largest packet that an interface can transmit without fragmentation.

MTU mismatches can cause difficult production issues where basic connectivity works, but applications, VPNs, large file transfers, or specific protocols fail.

## Common Symptoms

- Ping works but applications fail
- Large packets are dropped
- HTTPS sessions hang
- VPN traffic fails intermittently
- File transfers stop or perform poorly
- BGP sessions experience unexpected problems
- Fragmentation occurs across WAN links

## Troubleshooting Workflow

### 1. Verify Interface MTU

Cisco IOS/IOS XE:

```bash
show interfaces
show ip interface
```

Look for the configured MTU value.

Example:

```text
MTU 1500 bytes
```

### 2. Test Basic Connectivity

```bash
ping 10.10.10.2
```

If normal ping succeeds, continue testing larger packets.

### 3. Test with Don't Fragment

Cisco example:

```bash
ping 10.10.10.2 size 1472 df-bit
```

For IPv4 Ethernet with a 1500-byte MTU:

```text
1472 bytes ICMP payload
+ 20 bytes IPv4 header
+ 8 bytes ICMP header
= 1500 bytes
```

If this succeeds, the path supports a 1500-byte IP packet.

### 4. Reduce Packet Size

If the test fails, gradually reduce the packet size:

```bash
ping 10.10.10.2 size 1460 df-bit
ping 10.10.10.2 size 1400 df-bit
ping 10.10.10.2 size 1300 df-bit
```

Identify the largest successful packet.

### 5. Check Path MTU

Review every segment between source and destination.

```text
Source
   |
LAN
   |
Router
   |
Firewall
   |
VPN / WAN
   |
Router
   |
Destination
```

Encapsulation can reduce the effective MTU.

Examples include:

- GRE
- IPsec
- VXLAN
- MPLS
- SD-WAN overlays

### 6. Check ICMP Filtering

Path MTU Discovery depends on appropriate ICMP error signaling.

Overly restrictive firewall or ACL policies can interfere with PMTUD and create black-hole behavior.

Review:

- Firewall rules
- ACLs
- Security policies
- ICMP handling

### 7. Check Interface Errors

```bash
show interfaces counters errors
show interfaces
```

Look for:

- Drops
- Giants
- Input errors
- Output errors

### 8. Validate TCP MSS

For TCP applications, review whether MSS adjustment is required when tunnels reduce the effective path MTU.

Example Cisco configuration:

```bash
interface Tunnel10
 ip tcp adjust-mss 1360
```

The correct value must be based on the actual encapsulation and network design.

## Common Root Causes

### MTU Mismatch

Different network segments are configured with incompatible MTU values.

### Tunnel Overhead

GRE, IPsec, VXLAN, or SD-WAN encapsulation increases packet size.

### PMTUD Failure

Required ICMP messages are blocked somewhere in the path.

### Incorrect TCP MSS

TCP endpoints advertise segment sizes that are too large for the effective path.

### Jumbo Frame Mismatch

Some devices support jumbo frames while another device in the forwarding path does not.

## Production Troubleshooting Sequence

```text
Application Failure
        ↓
Verify Basic Connectivity
        ↓
Check Interface MTU
        ↓
DF-Bit Ping Testing
        ↓
Determine Maximum Packet Size
        ↓
Check Tunnel Overhead
        ↓
Check ICMP / PMTUD
        ↓
Validate TCP MSS
        ↓
Review Drops and Errors
        ↓
Retest Application
```

## Validation Checklist

- Source interface MTU verified
- Destination interface MTU verified
- Intermediate path reviewed
- DF-bit testing completed
- Maximum successful packet size identified
- Tunnel overhead considered
- ICMP signaling validated
- TCP MSS reviewed where applicable
- Interface drops checked
- Application retested successfully

## Best Practices

- Maintain consistent MTU settings where possible.
- Account for encapsulation overhead during network design.
- Avoid blindly changing MTU values during incidents.
- Validate PMTUD behavior across security devices.
- Document non-standard MTU configurations.
- Test application traffic after MTU changes.
- Monitor interfaces for fragmentation-related symptoms.
