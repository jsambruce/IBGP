# Addressing in the saved configurations

The two scenarios share one EVE-NG lab file but have no physical connection. Their router names and some addresses repeat, so the scenario name is part of each config path.

## Full mesh

| Link or interface | Side A | Side B |
| --- | --- | --- |
| R1–R2 | R1 Ethernet0/0 `1.1.1.1/8` | R2 Ethernet0/0 `1.1.1.2/8` |
| R2–R3 | R2 Ethernet0/1 `2.2.2.1/8` | R3 Ethernet0/1 `2.2.2.2/8` |
| R3–R4 | R3 Ethernet0/0 `3.3.3.1/8` | R4 Ethernet0/1 `3.3.3.2/8` |
| R4–R1 | R4 Ethernet0/0 `4.4.4.1/8` | R1 Ethernet0/1 `4.4.4.2/8` |

| Router | Loopback0 | LAN interface |
| --- | --- | --- |
| R1 | `11.0.0.1/32` | Ethernet0/2 `10.1.1.1/8` |
| R2 | `12.0.0.1/32` | Ethernet0/2 `20.1.1.1/8` |
| R3 | `13.0.0.1/32` | Ethernet0/2 `30.1.1.1/8` |
| R4 | `14.0.0.1/32` | Ethernet0/2 `40.1.1.1/8` |

## Route reflector

| Link or interface | Side A | Side B |
| --- | --- | --- |
| R1-RR–R2-RR | R1-RR Ethernet0/0 `1.1.1.1/8` | R2-RR Ethernet0/0 `1.1.1.2/8` |
| R2-RR–R3-RR | R2-RR Ethernet0/1 `2.2.2.1/8` | R3-RR Ethernet0/0 `2.2.2.2/8` |

| Router | LAN interface |
| --- | --- |
| R1-RR | Ethernet0/1 `10.1.1.1/8` |
| R2-RR | Ethernet0/2 `20.1.1.1/8` |
| R3-RR | Ethernet0/1 `30.1.1.1/8` |

R3-RR's `/8` is confirmed in the latest saved export and matches the topology label and BGP `network 30.0.0.0` statement.
