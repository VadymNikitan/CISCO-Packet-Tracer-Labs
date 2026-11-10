# Cisco DHCP Configuration with VLANs – Packet Tracer Lab

## Project Overview

This lab demonstrates DHCP configuration on a Cisco Router in a network with multiple VLANs using Cisco Packet Tracer.

The network uses Router-on-a-Stick for Inter-VLAN Routing and DHCP services for automatic IP address assignment.

## Network Topology

**Devices:**
- 1 Cisco Router (2911)
- 1 Cisco Switch (2960)
- 4 PCs
- 1 Server

**Technologies:**
- VLAN Configuration
- IEEE 802.1Q Trunking
- Router-on-a-Stick
- Inter-VLAN Routing
- DHCP Configuration
- Static IP Addressing
- ARP
- ICMP / Ping Testing

## IP Addressing Scheme

| VLAN | Name | Network | Default Gateway |
|---|---|---|---|
| 10 | HR | 192.168.10.0/24 | 192.168.10.1 |
| 20 | IT-Department | 192.168.20.0/24 | 192.168.20.1 |
| 30 | Servers | 192.168.30.0/24 | 192.168.30.1 |

## Device Allocation

| Device | VLAN | IP Configuration |
|---|---|---|
| PC0 | VLAN 10 | DHCP |
| PC1 | VLAN 10 | DHCP |
| PC2 | VLAN 20 | DHCP |
| PC3 | VLAN 20 | DHCP |
| Server | VLAN 30 | Static: 192.168.30.4 |

## Switch Configuration

### 1. Create VLANs

```cisco
enable
configure terminal

vlan 10
 name HR

vlan 20
 name IT-Department

vlan 30
 name Servers

exit
```

### 2. Assign Access Ports

```cisco
interface range fa0/1-2
 switchport mode access
 switchport access vlan 10
 exit

interface range fa0/3-4
 switchport mode access
 switchport access vlan 20
 exit

interface range fa0/5-7
 switchport mode access
 switchport access vlan 30
 exit
```

### 3. Configure Trunk Port

Replace `<TRUNK_PORT>` with the actual switch interface connected to the router.

```cisco
interface <TRUNK_PORT>
 switchport mode trunk
 exit
```

## Router Configuration

### 4. Configure Router-on-a-Stick

Physical interface: GigabitEthernet0/0

```cisco
enable
configure terminal

interface g0/0
 no shutdown
 exit
```

VLAN 10:

```cisco
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit
```

VLAN 20:

```cisco
interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit
```

VLAN 30:

```cisco
interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 exit
```

## DHCP Configuration

DHCP runs directly on the Cisco Router.

VLAN 10 and VLAN 20 use dynamic IP addressing.

The Server in VLAN 30 uses a manually configured static IP address, so no DHCP pool is required for VLAN 30.

### 5. Configure Excluded Addresses

Reserve the first ten addresses in each DHCP subnet.

```cisco
ip dhcp excluded-address 192.168.10.1 192.168.10.10

ip dhcp excluded-address 192.168.20.1 192.168.20.10
```

### 6. Configure DHCP Pool – VLAN 10

```cisco
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 exit
```

### 7. Configure DHCP Pool – VLAN 20

```cisco
ip dhcp pool VLAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 exit
```

## Server Configuration

The Server uses static IP addressing.

Configure under Desktop → IP Configuration:

| Parameter | Value |
|---|---|
| IP Address | 192.168.30.4 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.30.1 |
| DNS Server | Not configured |

## PC Configuration

On each PC:

1. Open Desktop.
2. Select IP Configuration.
3. Select DHCP.

Expected addressing:

| PC | VLAN | Expected IP | Gateway |
|---|---|---|---|
| PC0 | 10 | 192.168.10.11 | 192.168.10.1 |
| PC1 | 10 | DHCP assigned | 192.168.10.1 |
| PC2 | 20 | 192.168.20.11 | 192.168.20.1 |
| PC3 | 20 | DHCP assigned | 192.168.20.1 |

Note: Actual DHCP addresses depend on the order in which clients request leases.

## Verification and Testing

### 8. Verify VLANs

On the Switch:

```cisco
show vlan brief
```

### 9. Verify Trunk

```cisco
show interfaces trunk
```

### 10. Verify Router Interfaces

```cisco
show ip interface brief
```

Expected subinterfaces:

| Interface | IP Address | Status |
|---|---|---|
| G0/0.10 | 192.168.10.1 | up/up |
| G0/0.20 | 192.168.20.1 | up/up |
| G0/0.30 | 192.168.30.1 | up/up |

### 11. Verify DHCP Configuration

```cisco
show running-config | section dhcp
```

### 12. Verify DHCP Bindings

```cisco
show ip dhcp binding
```

This command displays dynamically assigned IP addresses and client MAC addresses.

### 13. Test Connectivity

From PC0:

```text
ping 192.168.10.1
ping 192.168.20.11
ping 192.168.30.4
```

Expected results:
- PC0 can reach its default gateway.
- PC0 can communicate with PC2 in VLAN 20.
- PC0 can communicate with the Server in VLAN 30.

## Test Results

| Test | Result |
|---|---|
| DHCP assignment in VLAN 10 | Successful |
| DHCP binding on Router | Successful |
| Inter-VLAN connectivity: VLAN 10 → VLAN 20 | Successful |
| Inter-VLAN connectivity: VLAN 10 → VLAN 30 | Successful |
| Server static IP connectivity | Successful |

Note: The first ping to PC2 had one timeout (25% packet loss), followed by three successful replies. This can occur during initial ARP resolution. A repeated ping should be used to confirm stable connectivity.

## Key Concepts Learned

- How DHCP dynamically assigns IP addresses.
- How DHCP excluded addresses prevent address conflicts.
- How to configure multiple DHCP pools on a Cisco Router.
- How VLANs separate broadcast domains.
- How 802.1Q trunking transports multiple VLANs.
- How Router-on-a-Stick enables Inter-VLAN Routing.
- How ARP resolves IP addresses into MAC addresses.
- How to troubleshoot network connectivity using Cisco IOS commands.

## Conclusion

Successfully configured DHCP services for two VLANs using a Cisco Router.

Implemented three VLANs, configured 802.1Q trunking, enabled Inter-VLAN Routing, assigned a static IP address to the Server, and verified connectivity between network segments.

**Platform:** Cisco Packet Tracer  
**Focus:** Networking Fundamentals / CCNA Practice  
**Project Type:** Hands-on Networking Lab
