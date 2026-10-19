# Verification checklist

The latest supplied EVE-NG export contains startup configurations for all seven routers, including the full-mesh peer-group rename to `IBGP-PEERS`. The full-mesh BGP CLI screenshots were captured after the rename. A later set of routing-table screenshots shows the result after an OSPF change in the running lab. I have not yet received a saved export of that change. Each capture is evidence from that moment, not a continuous health check.

## Full mesh: R1, R2, R3, R4

Captured on **each** full-mesh router after the peer-group rename:

```text
show ip bgp summary
show ip bgp
```

Each router is configured for three iBGP neighbors. In the supplied `show ip bgp summary` screenshots, all three peers on each router show a numeric `State/PfxRcd` of `1`, indicating established sessions on this IOS version. A state such as `Idle` or `Active` would not.

The new screenshots show numeric `State/PfxRcd` values for all three neighbors on each router and the four LAN prefixes in each BGP table.

The screenshots show `10.0.0.0/8`, `20.0.0.0/8`, `30.0.0.0/8`, and `40.0.0.0/8` in each router's BGP table. The later routing-table captures and R1's sourced ping add forwarding evidence. The remaining post-change protocol checks on full-mesh **R1** are:

```text
show ip bgp 30.0.0.0
show ip bgp summary
show ip bgp rib-failure
show ip ospf neighbor
```

The saved OSPF configs also advertise all four LANs. A successful ping or a LAN route in the IP routing table therefore does **not**, by itself, prove that BGP carried the prefix. The later routing-table captures below identify BGP as the installed source after the live OSPF change.

The remote LAN entries in the full-mesh BGP screenshots show `r` for `RIB-failure`. That is a reason to check the IP routing table before making any forwarding claim. OSPF advertising those same LANs is a possible explanation, not yet a verified cause.

### Later lab test: let iBGP carry the LANs

The saved configs use OSPF for the ring links and loopbacks, **and** advertise each router's LAN into OSPF. I wanted OSPF to keep the ring and loopbacks reachable while iBGP carried the four LAN prefixes. For the same `/8` prefix, OSPF's default administrative distance is 110 and iBGP's is 200, so an OSPF route would win if both paths were otherwise usable. The earlier `r` markers were consistent with this, but I did not capture a pre-change `show ip route` or `show ip bgp rib-failure` result to establish the exact cause.

For a repeat of this experiment, capture `show ip route 30.0.0.0`, `show ip route 13.0.0.1`, `show ip bgp rib-failure`, and `show ip bgp summary` on full-mesh R1 before changing anything. These show the installed route, loopback reachability, RIB-failure reason if supported, and session state.

The intended change is to remove only the matching LAN `network` line under `router ospf 1` on each full-mesh router. Keep the OSPF transit and Loopback0 lines, the connected LAN interface, and the BGP `network` statement. The exact post-change saved commands are pending the next EVE-NG export:

| Router | Command under `router ospf 1` |
| --- | --- |
| R1 | `no network 10.0.0.0 0.255.255.255 area 0` |
| R2 | `no network 20.0.0.0 0.255.255.255 area 0` |
| R3 | `no network 30.0.0.0 0.255.255.255 area 0` |
| R4 | `no network 40.0.0.0 0.255.255.255 area 0` |

Apply one router at a time and check that OSPF neighbors and iBGP sessions stay up. `passive-interface` on the LAN would stop OSPF hellos but would still advertise that LAN into OSPF, so it does not perform this experiment.

The LAN interface's connected `/8` route should remain in the originating router's routing table, allowing its existing BGP `network` statement to continue originating the prefix. Confirm this with the CLI rather than assuming it from the saved config.

The new screenshots show the following:

| Router | Screenshot | Remote LAN routes in `show ip route` |
| --- | --- | --- |
| R1 | [Routing table and sourced ping](../evidence/full-mesh/R1-routing-and-ping.png) | `20.0.0.0/8`, `30.0.0.0/8`, `40.0.0.0/8` all show `B [200/0]` |
| R2 | [Routing table](../evidence/full-mesh/R2-routing.png) | `10.0.0.0/8`, `30.0.0.0/8`, `40.0.0.0/8` all show `B [200/0]` |
| R3 | [Routing table](../evidence/full-mesh/R3-routing.png) | `10.0.0.0/8`, `20.0.0.0/8`, `40.0.0.0/8` all show `B [200/0]` |
| R4 | [Routing table](../evidence/full-mesh/R4-routing.png) | `10.0.0.0/8`, `20.0.0.0/8`, `30.0.0.0/8` all show `B [200/0]` |

R1's same screenshot shows OSPF routes to remote loopbacks, including `13.0.0.1/32`, and `ping 30.1.1.1 source 10.1.1.1` succeeding 5/5. That demonstrates one forwarding path in the full mesh. Fresh post-change `show ip bgp summary`, `show ip ospf neighbor`, and `show ip bgp rib-failure` outputs have not been supplied. I also need the new export to confirm the saved configuration matches the live test.

Cisco references: [administrative distance and route installation](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/15986-admin-distance.html), [BGP RIB-failure](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/5816-bgpfaq-5816.html), and [OSPF passive interfaces](https://www.cisco.com/c/en/us/td/docs/routers/ios-xe/ip-routing/b-ip-routing/m_iri-ip-prot-indep-0.html).

## Route reflector: R1-RR, R2-RR, R3-RR

The supplied `show ip bgp summary` and `show ip bgp` screenshots cover all three routers. R2-RR has two established peers with `2` received prefixes each; R1-RR and R3-RR each have one established peer with `4` received prefixes. R1-RR's table contains R3-RR's `30.0.0.0/8`, and R3-RR's table contains R1-RR's `10.0.0.0/8`. To test forwarding, send these outputs from **R1-RR**:

```text
show ip bgp 30.0.0.0
show ip route 2.2.2.2
show ip route 30.0.0.0
ping 30.1.1.1 source 10.1.1.1
```

And from **R3-RR**:

```text
show ip bgp 10.0.0.0
show ip route 1.1.1.1
show ip route 10.0.0.0
ping 10.1.1.1 source 30.1.1.1
```

The latest [R2-RR saved config](../configs/route-reflector/R2-RR.cfg) shows both `route-reflector-client` lines. A BGP table entry alone does not prove the route is installed in the IP routing table or that packets cross the lab.

Text output is preferable to screenshots for checking exact prefixes and neighbor states. A small number of clear CLI screenshots can supplement the text in the final README.

## Evidence status

| Check | Current status |
| --- | --- |
| Saved configurations for seven nodes | Latest export reviewed and sanitized in this repository |
| Full-mesh session states | Three established peers per router in screenshots captured after the peer-group rename |
| Route-reflector session states | Both client sessions established in screenshots |
| BGP tables and reflected route | Both clients show the other client's LAN prefix |
| Full-mesh IP routes and one sourced ping | Remote LAN routes appear as `B [200/0]` on all four routers; R1-to-R3 sourced ping succeeded 5/5 |
| Full-mesh saved config after OSPF change | New export pending |
| Full-mesh post-change BGP summary, OSPF neighbors, and RIB-failure reason | Not supplied |
| Route-reflector IP routes, next-hop resolution, and ping | Not supplied; forwarding remains unverified |
