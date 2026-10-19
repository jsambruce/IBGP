# Configuration review: first export and corrected export

I compared the first EVE-NG export with the latest saved export and the CLI screenshots. The public config files are from the **latest** export. They preserve the routing settings but remove the full-mesh BGP neighbor password lines.

The latest supplied export confirms the commands and interface mask below. The BGP screenshots separately confirm session state and route exchange after the peer-group rename. Later routing-table screenshots confirm that remote full-mesh LAN routes were installed from BGP, and one sourced ping succeeded after the live OSPF change. The saved export of that change is still pending.

## Full mesh R1: loopback update source corrected

The first export showed R1 peering with `12.0.0.1`, `13.0.0.1`, and `14.0.0.1` without an update source, while R2–R4 peered with R1 at `11.0.0.1` and used `update-source Loopback0`. R1 could have sourced an outgoing connection from a physical interface address the other routers did not configure as a neighbor. The latest R1 config contains `neighbor IBGP-PEERS update-source Loopback0`; its fresh CLI screenshot shows three established peers.

The correction made on full-mesh R1, when the group was still named `NEXT`, was:

```text
configure terminal
router bgp 500
 neighbor NEXT update-source Loopback0
end
copy running-config startup-config
```

The [four full-mesh screenshots](../evidence/full-mesh) show three established neighbors per router after the rename. A saved config alone would not have proved those sessions stayed up.

## Full mesh: peer-group name standardized

The latest export changes the local group name from `NEXT` on R1 and `CCIE` on R2–R4 to `IBGP-PEERS` on all four routers. The group settings and three neighbor assignments per router are otherwise unchanged in the saved configs. Peer-group names are local labels; fresh `show ip bgp summary` output confirms all three neighbors established on each router after the change.

## Full mesh R1: LAN network statement added

The first export had `10.1.1.1/8` on R1 Ethernet0/2 but no BGP `network 10.0.0.0` statement. The latest R1 config includes it, and R1's BGP screenshot shows `10.0.0.0/8` locally originated. The correction was:

```text
configure terminal
router bgp 500
 network 10.0.0.0
end
copy running-config startup-config
```

The saved full-mesh configs also advertise their LANs into OSPF. This made a successful ping insufficient as proof that BGP carried a LAN route. See the [verification checklist](verification.md).

In the earlier full-mesh BGP screenshots, remote LAN routes carry the `r` status (`RIB-failure` in the command legend). BGP had learned them, but that status did not show installation in the IP routing table. OSPF was a possible competing route source because it advertised those LANs. After the live OSPF change, [new routing-table screenshots](verification.md#later-lab-test-let-ibgp-carry-the-lans) show the remote LAN routes installed as `B [200/0]` on all four full-mesh routers. No pre-change routing table or RIB-failure reason was captured, so I cannot prove the earlier `r` markers were caused solely by OSPF.

## Route reflector R3-RR: LAN mask corrected

The first export had R3-RR Ethernet0/1 at `30.1.1.1/24`, while its topology label and BGP statement used `30.0.0.0/8`. The latest export has `30.1.1.1 255.0.0.0` on that interface, matching the intended `/8`. R3-RR's BGP screenshot shows it originating `30.0.0.0/8`, and R1-RR's table shows the prefix learned through R2-RR.

The correction made on R3-RR was:

```text
configure terminal
interface Ethernet0/1
 ip address 30.1.1.1 255.0.0.0
end
copy running-config startup-config
```

The latest export also confirms the distinct `R1-RR`, `R2-RR`, and `R3-RR` hostnames. The two topologies reuse addresses but remain disconnected.

## Public config note

The exported full-mesh configs contained a shared BGP neighbor password. Its value is absent from this repository. If BGP authentication is needed when recreating the lab, configure the same fresh secret on both ends of every peer relationship.

## Cisco references

- [Configuring internal BGP and route reflectors](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/ip-routing/b-ip-routing/m_irg-int-features-0.html)
- [Troubleshooting BGP routes that do not advertise](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/19345-bgp-noad.html)
