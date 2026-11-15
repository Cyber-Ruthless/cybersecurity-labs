# DHCP Starvation Attack with Yersinia

A lab demonstrating a DHCP starvation attack — flooding a network segment with forged DHCP Discover packets using **Yersinia** to exhaust a DHCP server's available address pool, with the attack confirmed live in Wireshark.

> ⚠️ **Disclaimer:** Performed entirely in an isolated lab environment (Parrot Security VM, interface `ens33`) that I own and control. DHCP starvation can exhaust a real network's address pool and deny service to legitimate clients. Do not run this against any network you do not own or have explicit written authorization to test.

---

## Lab Environment

| Component | Details |
|---|---|
| Attacker machine | Parrot Security OS |
| Interface | `ens33` |
| Attack tool | [Yersinia](https://github.com/tomac/yersinia) v0.8.2 |
| Capture tool | Wireshark |

---

## 1. Launching Yersinia in DHCP Mode

Yersinia was launched in DHCP mode, which surfaces an attack panel with several Layer 2/DHCP-specific options:

```
0 - sending RAW packet
1 - sending DISCOVER packet   [X]
2 - creating DHCP rogue server
3 - sending RELEASE packet    [X]
```

Before launching the flood, the panel shows a small baseline of normal DHCP traffic — a legitimate `REQUEST`/`ACK` exchange between a client (`192.168.172.1`) and the DHCP server (`192.168.172.254`), with MAC spoofing already enabled.

![Yersinia DHCP attack panel with available attack options](./images/yaresina.png)

---

## 2. Launching the DHCP Discover Flood

Selecting option **1 (sending DISCOVER packet)** put Yersinia into a continuous flood, rapidly broadcasting forged `DHCPDISCOVER` packets — each with a spoofed source MAC — from `0.0.0.0` to the broadcast address `255.255.255.255`.

Within seconds, the packet counter climbed into the hundreds of thousands:

```
Total Packets: 191278
DHCP Packets:  191278
MAC Spoofing:  [X]
```

Every forged request mimics a brand-new client attempting to obtain a lease, which — against a real DHCP server — would rapidly exhaust the available address pool and deny service to legitimate devices on the network.

![Yersinia actively flooding the network with forged DHCP Discover packets](./images/yersinia%20dhcp%20attack.png)

---

## 3. Confirming the Flood in Wireshark

A simultaneous Wireshark capture on `ens33` confirmed the attack at the packet level — a continuous stream of `DHCP Discover` packets, each with a unique transaction ID, interspersed with legitimate ARP broadcasts and occasional unanswered ICMP echo requests from the real DHCP server attempting to respond to exhausted/invalid leases.

![Wireshark capture confirming the DHCP Discover flood](./images/wireshark.png)

---

## How This Attack Works

DHCP servers allocate IP addresses from a finite pool, tracking each lease against the client's MAC address. By spoofing a different MAC address on every `DHCPDISCOVER` request, an attacker can request (and tie up) leases far faster than legitimate clients can request real ones — eventually exhausting the entire pool. Once exhausted, legitimate devices on the network can no longer obtain an IP address at all, resulting in a denial of service. This is also frequently combined with option `2` (rogue DHCP server) to then hand out attacker-controlled DNS/gateway settings once the real server's pool is dead — enabling follow-on man-in-the-middle attacks.

---

## Key Takeaways / Mitigations

- **DHCP Snooping** — enable on switches to track legitimate DHCP server/client bindings and drop forged DHCP traffic from untrusted ports.
- **Port Security** — limit the number of MAC addresses allowed per switch port, which directly undermines the MAC-spoofing technique this attack depends on.
- **Rate-limit DHCP traffic per port** — many enterprise switches support throttling DHCP packet rates to prevent exactly this kind of flood.
- **Monitor DHCP server logs** — a sudden spike in `DISCOVER` requests from new MAC addresses in a short window is a strong indicator of this attack.

---

## Tools Referenced

- [Yersinia](https://github.com/tomac/yersinia) — Layer 2 attack framework (DHCP, STP, CDP, DTP, and more)
- [Wireshark](https://www.wireshark.org/)
