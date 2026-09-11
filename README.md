# 🏥 Hospital Network Design — Cisco Packet Tracer Lab

A 3-site hospital campus network (IT Department, Clinical Ward, Entrance & Lobby) built and validated in Cisco Packet Tracer, with full OSPF routing between sites.

![Hospital Network Topology](./hospital-network-topology.svg)



---

## 📖 Overview

This lab simulates a hospital's internal network across three physical sites, connected in a hub-and-spoke topology through a central IT-Router:

| Site | Role | Router | Switch(es) |
|---|---|---|---|
| **IT Department** | Core services — DNS, HTTP, SMTP, FTP; hub of the WAN | `IT-Router` | `IT-switch` |
| **Entrance & Lobby** | Reception, billing, guest WiFi | `Entrance-Router` | `Entrance-switch` |
| **Clinical Ward** | Exam rooms, doctors' offices, general & private wards | `Clinical Router` | `clinical Switch` → `General Switch` → `Private Switch` (cascaded) |

All three sites run **OSPF (Area 0)**, converging into a single routed domain so every host can reach every other host and both servers.

---

## 🖧 Topology

- **30 devices** total: 3 routers, 5 switches, 2 servers, 14 PCs, 1 wireless router (guest WiFi), 4 wireless clients (3 smartphones + 1 tablet), 1 power distribution unit.
- **Router-to-router links** run over fiber expansion ports (`GigabitEthernet0/x/0`) instead of legacy Serial/HWIC — Serial WAN isn't renderable by the PT exporter, so all inter-site links were rebuilt on fiber.
- **Router-to-switch uplinks** use copper `GigabitEthernet0/0` ↔ `GigabitEthernet0/1`.
- Every switch is a **2960-24TT** (L2 access layer only — no SVIs used for routing, management-only Vlan1 IPs).

```
                         ┌────────────────┐
                         │   IT-Router    │  (hub)
                         │ 192.168.1.1    │
                         └───┬────────┬───┘
                 fiber Gi0/0/0    fiber Gi0/1/0
                    ┌──────┘          └──────┐
          ┌─────────▼────────┐     ┌─────────▼─────────┐
          │  Entrance-Router  │     │  Clinical Router   │
          │  192.168.3.1      │     │  192.168.2.1       │
          └─────────┬─────────┘     └─────────┬──────────┘
                     │                          │
             Entrance-switch              clinical Switch
           (Billing, Info, Main,          (Clinical Recep., Test Room,
            Recep., Guest WiFi)            Ultrasound, Op. Theater)
                                                  │
                                           General Switch (cascade)
                                          (Dr. Khan, Dr. Ali, Dr. Malik)
                                                  │
                                           Private Switch (cascade)
                                    (2nd Recep., Dr. Ahmed, Private Ward)
```

---

## 🌐 IP Addressing Plan

| Subnet | CIDR | Assigned to |
|---|---|---|
| `lan-it` | `192.168.1.0/24` | IT-Router, IT-switch, IT Reception, DNS+HTTP Server, SMTP+FTP Server |
| `lan-clinical` | `192.168.2.0/24` | Clinical Router, clinical/General/Private switches, all clinical PCs |
| `lan-entrance` | `192.168.3.0/24` | Entrance-Router, Entrance-switch, lobby PCs, guest WiFi router (static WAN) |
| `wan-it-entrance` | `192.168.100.0/30` | IT-Router ↔ Entrance-Router fiber link |
| `wan-it-clinical` | `192.168.100.4/30` | IT-Router ↔ Clinical Router fiber link |

**Routing:** OSPF Area 0, advertised on all LAN and WAN interfaces of all three routers. IT-Router acts as the transit hub — traffic between Entrance and Clinical sites passes through it.

---

## 🔁 Substitutions from the original design

The original hand-built design used **Cisco 1941 routers**, which Packet Tracer's programmatic exporter can't re-render exactly. They were substituted 1:1 with **Cisco 2911 routers**, keeping the same hostnames, interface roles, and routed subnets:

| Original | Substituted with | Why |
|---|---|---|
| `IT-Router` (1941) | 2911 | 1941 not supported for re-export |
| `Clinical Router` (1941) | 2911 | 1941 not supported for re-export |
| `Entrance-Router` (1941) | 2911 | 1941 not supported for re-export |
| Serial (HWIC-2T) inter-router links | Fiber `GigabitEthernet0/x/0` | Serial WAN ports aren't renderable by the exporter |

Two latent bugs from the original file were also fixed so end-to-end connectivity actually works:
- `clinical Switch` management IP (`192.168.2.2`) collided with **Clinical Reception's** host IP — moved the switch to `192.168.2.20`.
- `IT-Router` and `Entrance-Router` both had a WAN interface set to `192.168.8.1` — resolved with clean, non-overlapping `/30` links (`192.168.100.0/30`, `192.168.100.4/30`).

---

## ✅ Verifying end-to-end connectivity

1. Open `Hospital Design.pkt` in Cisco Packet Tracer.
2. Give OSPF a few seconds to converge after opening (routers exchange hellos on first load).
3. From any PC's Desktop → Command Prompt, ping a PC on a different site, e.g.:
   ```
   C:\> ping 192.168.2.13    (IT Reception → Dr Ahmed, Private Ward)
   C:\> ping 192.168.1.4     (Billing Counter → DNS+HTTP Server)
   ```
4. Expect 4/4 replies once OSPF has converged. `tracert` will show the path hopping through `IT-Router` for any cross-site pair.

---

## 📁 Files

| File | Description |
|---|---|
| `Hospital Design.pkt` | The Packet Tracer lab file — open directly in Packet Tracer 9.0+ |
| `hospital-network-topology.svg` | Animated topology diagram used in this README |
| `LICENSE` | MIT License |

---
