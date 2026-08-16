# Case Study: Xbox Double NAT Caused by ISP Gateway IP Passthrough Failure

## Ticket Information

| Field                 | Details                                   |
| --------------------- | ----------------------------------------- |
| **Case ID**           | INC-001                                   |
| **Category**          | Network / Connectivity                    |
| **Priority**          | Normal                                    |
| **Environment**       | Home network                              |
| **Status**            | Resolved                                  |
| **Issue Type**        | Incident                                  |
| **Affected Device**   | Xbox console                              |
| **Network Equipment** | AT&T BGW320-500 + Netgear Nighthawk RAX54 |

> **Privacy / OPSEC:** Public IP addresses, MAC addresses, account information, and other identifying network information have been redacted or generalized.

---

## User-Reported Issue

The Xbox began reporting a **Double NAT** condition and experienced connectivity issues affecting online gaming and peer-to-peer connectivity.

The network had previously operated with the AT&T BGW320-500 configured for IP Passthrough to a downstream Netgear Nighthawk RAX54 router.

---

## User Impact

The Double NAT condition resulted in restricted Xbox NAT behavior and affected online multiplayer connectivity.

The Xbox reported a NAT status other than Open, indicating that the expected downstream routing configuration was no longer functioning correctly.

---

## Initial Technical Observations

The first troubleshooting step was to inspect the WAN status of the downstream Netgear RAX54.

The RAX54 was receiving:

```text
WAN IP: 192.168.1.65
```

This indicated that the RAX54 was receiving a private address from the AT&T BGW320-500 rather than directly receiving the expected public WAN assignment.

### Initial assessment

The observed WAN address indicated that the BGW320 was performing routing/NAT for the RAX54 rather than passing the external address through to it.

This was consistent with the Xbox's Double NAT report.

---

## Troubleshooting Process

### 1. Verify the downstream router WAN configuration

Checked the RAX54 WAN status and confirmed it was receiving a `192.168.1.x` address from the BGW320.

**Result:** Confirmed the downstream router was behind the ISP gateway's private network.

**Next step:** Inspect the BGW320 IP Passthrough configuration.

---

### 2. Inspect the AT&T BGW320 configuration

Logged into the BGW320 administration interface and inspected the IP Passthrough configuration.

The expected passthrough configuration was no longer active.

**Finding:** The gateway was not passing the public WAN assignment to the RAX54 as expected.

At this point, the Double NAT condition had been narrowed down to the upstream gateway configuration rather than the Xbox itself.

---

### 3. Investigate the gateway device association

The BGW320 device list contained an incorrect or unexpected hostname associated with the RAX54's WAN interface.

The device was displayed as an **"iPhone 16"** rather than being identified by its expected router/device name.

Because the gateway's IP Passthrough configuration requires selection of the downstream device, this incorrect device association complicated the configuration process.

The device list was cleared to force the gateway to rediscover connected devices.

---

### 4. Restore IP Passthrough

Navigated to:

```text
Firewall → IP Passthrough
```

Configured:

```text
Allocation Mode: Passthrough
Passthrough Mode: DHCPS-Fixed
```

The RAX54's WAN interface MAC address was selected as the passthrough target.

The MAC address is redacted in this public documentation.

The configuration was saved and the RAX54 was restarted to obtain a new WAN assignment.

---

### 5. Verify downstream WAN assignment

After the RAX54 restarted, its WAN interface was checked again.

The router successfully obtained a public WAN address rather than a `192.168.1.x` private address.

**Result:** The expected IP Passthrough behavior had been restored.

---

### 6. Disable unused BGW320 wireless radios

Because the RAX54 is the primary wireless router for the network, the BGW320's 2.4 GHz and 5 GHz wireless radios were disabled.

This removed the unused ISP gateway wireless network from the environment and left the RAX54 responsible for the primary wireless network.

This was a configuration cleanup step rather than the root cause of the Double NAT issue.

---

## Root Cause

The immediate cause of the incident was that the AT&T BGW320-500 was no longer providing the expected IP Passthrough behavior to the Netgear RAX54.

As a result:

```text
Before remediation:

Internet
   ↓
AT&T BGW320
   ↓ NAT
192.168.1.x
   ↓
Netgear RAX54
   ↓ NAT
10.0.0.x
   ↓
Xbox
```

The RAX54 was therefore operating behind the BGW320's private network, producing a Double NAT condition.

### Suspected trigger

The gateway configuration had apparently changed prior to troubleshooting.

A gateway firmware update, reboot, or other configuration/state change is a possible trigger, but the exact cause of the configuration change was not independently verified.

---

## Resolution

The incident was resolved by:

1. Clearing the BGW320 device list to refresh device identification.
2. Restoring IP Passthrough on the BGW320.
3. Configuring the passthrough mode as `DHCPS-Fixed`.
4. Selecting the RAX54 WAN interface as the passthrough target.
5. Restarting the RAX54 to obtain the new WAN assignment.
6. Disabling unused BGW320 wireless radios.

---

## Verification

The following checks confirmed successful remediation:

* RAX54 WAN interface changed from a private `192.168.1.x` address to a public WAN address.
* Xbox no longer reported Double NAT.
* Xbox NAT status changed to **Open**.
* Online multiplayer and peer-to-peer connectivity returned to normal.

---

## Additional Observation: DHCP Lease Behavior

After restoring IP Passthrough, the BGW320 provided a relatively short DHCP lease to the downstream WAN interface.

The observed lease duration was approximately **10 minutes**.

The short lease did not result in recurring connectivity problems during testing.

The downstream router was observed successfully maintaining its WAN connectivity through DHCP renewal.

This was documented as an operational observation rather than a contributing factor to the incident.

---

## Lessons Learned

### 1. Start at the point of failure

The initial symptom appeared on the Xbox, but the Xbox was not necessarily the source of the problem.

Checking the downstream router's WAN address quickly established that the problem existed upstream of the console.

### 2. Verify assumptions about infrastructure state

The network had previously been configured correctly, but the configuration could not be assumed to remain unchanged.

Checking the actual gateway and router state was more useful than troubleshooting the Xbox in isolation.

### 3. Use evidence to narrow the troubleshooting domain

The progression was:

```text
Xbox Double NAT
      ↓
RAX54 WAN address
      ↓
Private WAN address detected
      ↓
BGW320 configuration
      ↓
IP Passthrough failure identified
      ↓
Configuration restored
      ↓
Public WAN address restored
      ↓
Xbox NAT verified
```

Each step provided evidence for the next troubleshooting action.

### 4. Separate observed facts from assumptions

The investigation initially suggested several possible causes for the gateway's unexpected state.

The final documentation distinguishes between:

* What was directly observed
* What was tested
* What was resolved
* What remains a suspected trigger

This prevents unsupported assumptions from being presented as confirmed root causes.

---

## Skills Demonstrated

* Tier 1 incident troubleshooting
* Network connectivity troubleshooting
* IPv4 addressing
* DHCP
* NAT / Double NAT
* WAN vs. LAN troubleshooting
* Router and gateway configuration
* MAC address identification
* Structured troubleshooting methodology
* Root-cause isolation
* Post-resolution verification
* Technical documentation
* Privacy / OSINT-conscious documentation

---

## Related Knowledge Base Article

**KB-001: Troubleshooting Double NAT in a Downstream Router Configuration**

This incident can be generalized into a reusable troubleshooting procedure covering:

1. Verify the endpoint symptom.
2. Check the downstream router's WAN address.
3. Determine whether the WAN address is public or private.
4. Inspect the upstream gateway's routing/IP Passthrough configuration.
5. Verify the passthrough target device.
6. Restore the intended routing configuration.
7. Renew the downstream WAN assignment.
8. Verify the endpoint's NAT/connectivity status.

---

## Evidence

Screenshots and configuration evidence are stored in the `screenshots/` directory.

All identifying information has been removed or redacted before publication.

