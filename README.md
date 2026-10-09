# DHCP Snooping and Rogue DHCP Server Prevention

**CCNA Networking Project | Cisco Packet Tracer | Layer 2 Security**

## 1. Project Overview

This project demonstrates the configuration and testing of DHCP Snooping on a Cisco Catalyst 2960 switch using Cisco Packet Tracer.

The main purpose is to protect the network from unauthorized DHCP servers, commonly known as Rogue DHCP Servers.

DHCP Snooping classifies switch interfaces as trusted or untrusted and filters unauthorized DHCP server messages received on untrusted interfaces.

## 2. Project Objectives

* Understand DHCP and DHCP Snooping.
* Configure a legitimate DHCP server.
* Configure a Rogue DHCP server for security testing.
* Configure trusted and untrusted switch ports.
* Apply DHCP Snooping to VLAN 1.
* Configure DHCP message rate limiting.
* Verify DHCP Snooping status and bindings.
* Test client DHCP address assignment.
* Identify and troubleshoot unauthorized DHCP responses.

## 3. Tools and Technologies

* Cisco Packet Tracer
* Cisco Catalyst 2960 Switch
* DHCP Server
* Rogue DHCP Server
* Client PCs
* Cisco IOS CLI
* DHCP Snooping
* Layer 2 Network Security

## 4. Network Topology

The lab contains the following devices:

| Device  | Connection     | Purpose                     |
| ------- | -------------- | --------------------------- |
| Switch0 | Central switch | DHCP Snooping configuration |
| Server0 | Fa0/1          | Legitimate DHCP server      |
| PC0     | Fa0/2          | DHCP client                 |
| PC1     | Fa0/3          | DHCP client                 |
| Server1 | Fa0/4          | Rogue DHCP server           |

### Port Security Design

* Fa0/1: Trusted interface connected to the legitimate DHCP server.
* Fa0/2: Untrusted client interface.
* Fa0/3: Untrusted client interface.
* Fa0/4: Untrusted interface connected to the Rogue DHCP server.

**Important:** The legitimate DHCP server must be reachable through the trusted interface. Server0 and Server1 should use static IP addresses for this lab.

## 5. IP Addressing Plan

| Device  | IP Address    | Subnet Mask   |
| ------- | ------------- | ------------- |
| Server0 | 192.168.10.2  | 255.255.255.0 |
| Server1 | 192.168.10.3  | 255.255.255.0 |
| PC0     | DHCP assigned | 255.255.255.0 |
| PC1     | DHCP assigned | 255.255.255.0 |

### Legitimate DHCP Pool

* Pool Name: OFFICE
* Default Gateway: 192.168.10.1
* DNS Server: 8.8.8.8
* Starting IP Address: 192.168.10.100
* Subnet Mask: 255.255.255.0
* Maximum Users: 50

### Rogue DHCP Pool

* Pool Name: ROGUE
* Default Gateway: 192.168.10.254
* DNS Server: 1.1.1.1
* Starting IP Address: 192.168.10.200
* Subnet Mask: 255.255.255.0
* Maximum Users: 20

The Rogue DHCP pool intentionally uses different gateway and DNS settings to demonstrate the risk of unauthorized DHCP services.

**Note:** This isolated lab uses VLAN 1 and does not require a functioning gateway for basic DHCP address assignment. The gateway addresses are used to demonstrate the difference between the two DHCP pools.

## 6. Step-by-Step Implementation

### Step 1: Create the Topology

1. Open Cisco Packet Tracer.
2. Add one Cisco 2960 switch.
3. Add two server devices.
4. Add two PC devices.
5. Connect all devices using Copper Straight-Through cables.
6. Wait for the switch interfaces to become active.

### Step 2: Configure Server0

Open Server0 → Desktop → IP Configuration.

Configure:

* IP Address: 192.168.10.2
* Subnet Mask: 255.255.255.0

Open Services → DHCP.

1. Turn DHCP On.
2. Enter the OFFICE pool details from the IP Addressing Plan.
3. Click Add.

### Step 3: Configure Server1

Open Server1 → Desktop → IP Configuration.

Configure:

* IP Address: 192.168.10.3
* Subnet Mask: 255.255.255.0

Open Services → DHCP.

1. Turn DHCP On.
2. Enter the ROGUE pool details from the IP Addressing Plan.
3. Click Add.

The second server represents an unauthorized DHCP service.

### Step 4: Configure DHCP Snooping

Open Switch0 → CLI.

```text
enable
configure terminal
ip dhcp snooping
ip dhcp snooping vlan 1
```

These commands enable DHCP Snooping globally and apply it to VLAN 1.

### Step 5: Configure the Trusted Interface

```text
interface fastEthernet 0/1
ip dhcp snooping trust
exit
```

Fa0/1 is trusted because it connects to the legitimate DHCP server.

### Step 6: Configure Untrusted Interfaces

```text
interface range fastEthernet 0/2 - 4
ip dhcp snooping limit rate 10
exit
```

The client ports and Rogue DHCP server port remain untrusted by default.

The rate-limit command limits DHCP packets on these interfaces. It is not the command that makes the interfaces untrusted; their default untrusted status provides that behavior.

### Step 7: Save the Configuration

```text
end
write memory
```

If prompted to confirm the destination filename, press Enter.

## 7. Complete Switch Configuration

The following is the combined configuration for reference:

```text
enable
configure terminal

ip dhcp snooping
ip dhcp snooping vlan 1

interface fastEthernet 0/1
ip dhcp snooping trust
exit

interface range fastEthernet 0/2 - 4
ip dhcp snooping limit rate 10
exit

end
write memory
```

## 8. Verification Commands

Run these commands on Switch0.

### Check DHCP Snooping Status

```text
show ip dhcp snooping
```

Verify that DHCP Snooping is enabled and VLAN 1 is listed as a snooping VLAN.

Check the trusted interface information. Fa0/1 should be trusted, while Fa0/2 through Fa0/4 should be untrusted.

### Check DHCP Snooping Bindings

```text
show ip dhcp snooping binding
```

This command displays learned DHCP bindings, such as client MAC address, assigned IP address, VLAN, interface and lease information, where supported.

Bindings are normally learned when clients obtain addresses through DHCP. Static server IP configurations will not automatically appear as client DHCP bindings.

### Check Interface Status

```text
show interfaces status
```

This helps verify whether the switch ports are connected.

### Check VLAN Membership

```text
show vlan brief
```

Verify that the devices' switch ports belong to VLAN 1 in this lab.

### Check the Running Configuration

```text
show running-config
```

Review the saved-in-RAM configuration and confirm the DHCP Snooping settings.

### Check the Startup Configuration

```text
show startup-config
```

Use this command to verify the configuration saved to startup memory after `write memory`.

**Platform note:** Available commands and displayed fields can differ by Packet Tracer device model and IOS simulation support.

## 9. Testing and Validation

### Test 1: Legitimate DHCP Server

1. Open PC0.
2. Select Desktop → IP Configuration.
3. Click DHCP.
4. Check the assigned address and subnet mask.
5. Repeat the process on PC1.

Expected result: Clients should receive an address from the legitimate DHCP pool if the DHCP service and snooping configuration are working correctly.

### Test 2: Rogue DHCP Server Prevention

1. Confirm that Fa0/4 is untrusted.
2. Confirm that Fa0/1 is trusted.
3. Renew a client DHCP lease.
4. Observe DHCP messages using Packet Tracer's Simulation Mode.
5. Filter or inspect DHCP packets and follow the Discover, Offer, Request and Acknowledgment exchange.
6. Check whether the rogue server's DHCP Offer is dropped by the switch.

Expected result: DHCP server replies arriving through the untrusted Rogue DHCP interface should be dropped by DHCP Snooping.

### Test 3: Binding Table Verification

```text
show ip dhcp snooping binding
```

Expected result: A valid client binding should appear after a successful DHCP lease if the simulated device supports binding-table learning.

### Test 4: Ping Test

Open PC0 → Desktop → Command Prompt.

```text
ipconfig
ping 192.168.10.2
ping 192.168.10.3
```

The server ping tests verify basic connectivity to the server IPs. They do not, by themselves, prove that DHCP Snooping blocked the rogue server.

### Test 5: Compare DHCP Options

Compare the IP configuration received by the clients with the two DHCP pools.

* Legitimate pool gateway: 192.168.10.1
* Rogue pool gateway: 192.168.10.254
* Legitimate pool DNS: 8.8.8.8
* Rogue pool DNS: 1.1.1.1

Expected result: Clients should receive the legitimate DHCP options rather than the unauthorized options.

Record the actual assigned IP, gateway, DNS and Simulation Mode results in the project evidence.

## 10. Troubleshooting

### Problem: PC does not receive an IP address

Check:

* DHCP service is On.
* DHCP pool was added successfully.
* Server IP and subnet mask are correct.
* Server is connected to the intended switch port.
* DHCP Snooping is enabled for the correct VLAN.
* Fa0/1 is trusted.
* No interface or DHCP rate-limit errors are occurring.

### Problem: Rogue DHCP Offer is not blocked

Check:

* Rogue server is connected to Fa0/4.
* Fa0/4 has not been configured as trusted.
* DHCP Snooping is enabled globally.
* VLAN 1 is enabled for snooping.
* The Rogue DHCP service is configured and active.
* The correct packet is being inspected in Simulation Mode.

### Problem: Binding table is empty

Check that clients completed a DHCP lease process. Static IP configuration alone does not create a DHCP Snooping binding.

### Problem: Ping fails

Check IP addressing, cable connections, switch port status and subnet masks. Ping failure does not automatically mean DHCP Snooping is configured incorrectly.

## 11. Security Concepts Demonstrated

### DHCP Snooping

A Layer 2 security feature that filters DHCP messages based on trusted and untrusted switch interfaces.

### Trusted Interface

An interface permitted to receive authorized DHCP server messages.

### Untrusted Interface

An interface that does not permit unauthorized DHCP server replies.

### Rogue DHCP Server

An unauthorized server that may provide incorrect IP addressing, gateway or DNS information to clients.

### DHCP Rate Limiting

A control that limits the rate of DHCP packets received on an interface and helps mitigate DHCP-related abuse.

## 12. Project Outcome

This project demonstrates how to configure DHCP Snooping and establish a trust boundary on a Cisco switch.

The expected security outcome is that DHCP server replies arriving through untrusted interfaces are blocked, while legitimate DHCP communication through the trusted interface is permitted.

Actual results should be confirmed using switch verification commands and Packet Tracer Simulation Mode.

## 13. Skills Demonstrated

* Cisco IOS CLI configuration
* DHCP server configuration
* DHCP Snooping
* Trusted and untrusted interface configuration
* DHCP security testing
* Layer 2 switching
* Network troubleshooting
* Packet Tracer simulation and verification

## 14. Project Evidence

Add screenshots of the following:

1. Complete network topology.
2. Legitimate DHCP pool configuration.
3. Rogue DHCP pool configuration.
4. DHCP Snooping configuration on the switch.
5. `show ip dhcp snooping` output.
6. `show ip dhcp snooping binding` output.
7. Client IP configuration.
8. Simulation Mode evidence of a rogue DHCP Offer being dropped.

## 15. Conclusion

The lab demonstrates the importance of DHCP Snooping in protecting a switched network against unauthorized DHCP servers.

It provides practical experience with switch security, DHCP behavior, IOS configuration, verification and troubleshooting.

**Project Type:** CCNA Networking Lab
**Platform:** Cisco Packet Tracer
**Focus:** DHCP Security and Layer 2 Switching
