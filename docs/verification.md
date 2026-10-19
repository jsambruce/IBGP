# Verification checklist

The latest EVE-NG export contains startup configurations for all seven routers, including the full-mesh peer-group rename to `IBGP-PEERS`. The full-mesh CLI screenshots were captured after the rename. They are evidence from one capture, not a continuous health check.

## Full mesh: R1, R2, R3, R4

Captured on **each** full-mesh router after the peer-group rename:

```text
show ip bgp summary
show ip bgp
```

Each router is configured for three iBGP neighbors. In the supplied `show ip bgp summary` screenshots, all three peers on each router show a numeric `State/PfxRcd` of `1`, indicating established sessions on this IOS version. A state such as `Idle` or `Active` would not.

The new screenshots show numeric `State/PfxRcd` values for all three neighbors on each router and the four LAN prefixes in each BGP table.

The screenshots show `10.0.0.0/8`, `20.0.0.0/8`, `30.0.0.0/8`, and `40.0.0.0/8` in each router's BGP table. For a concrete next check, run these on full-mesh **R1** and send the output:

```text
show ip bgp 30.0.0.0
show ip route 30.0.0.0
show ip route 13.0.0.1
ping 30.1.1.1 source 10.1.1.1
```

The saved OSPF configs also advertise all four LANs. A successful ping or a LAN route in the IP routing table therefore does **not**, by itself, prove that BGP carried the prefix. The `show ip route 30.0.0.0` output will identify the installed source. If testing BGP-only LAN routing is the goal, remove the LANs from OSPF as a separate, documented experiment after the underlay and BGP sessions are stable.

The remote LAN entries in the full-mesh BGP screenshots show `r` for `RIB-failure`. That is a reason to check the IP routing table before making any forwarding claim. OSPF advertising those same LANs is a possible explanation, not yet a verified cause.

### Planned experiment: let iBGP carry the LANs

The saved configs use OSPF for the ring links and loopbacks, **and** advertise each router's LAN into OSPF. I want OSPF to keep the ring and loopbacks reachable while iBGP carries the four LAN prefixes. For the same `/8` prefix, OSPF's default administrative distance is 110 and iBGP's is 200, so an OSPF route would win if both paths are otherwise usable. The `r` marker suggests this is happening, but the routing table still needs to confirm it. This change has **not** been made or validated in the supplied export.

Before changing anything, capture `show ip route 30.0.0.0`, `show ip route 13.0.0.1`, `show ip bgp rib-failure`, and `show ip bgp summary` on full-mesh R1. These will show the installed route, loopback reachability, RIB-failure reason if supported, and session state.

In the EVE-NG lab, remove only the matching LAN `network` line under `router ospf 1` on each full-mesh router. Keep the OSPF transit and Loopback0 lines, the connected LAN interface, and the BGP `network` statement:

| Router | Command under `router ospf 1` |
| --- | --- |
| R1 | `no network 10.0.0.0 0.255.255.255 area 0` |
| R2 | `no network 20.0.0.0 0.255.255.255 area 0` |
| R3 | `no network 30.0.0.0 0.255.255.255 area 0` |
| R4 | `no network 40.0.0.0 0.255.255.255 area 0` |

Apply one router at a time and check that OSPF neighbors and iBGP sessions stay up. `passive-interface` on the LAN would stop OSPF hellos but would still advertise that LAN into OSPF, so it does not perform this experiment.

The LAN interface's connected `/8` route should remain in the originating router's routing table, allowing its existing BGP `network` statement to continue originating the prefix. Confirm this with the CLI rather than assuming it from the saved config.

After all four changes, capture `show ip ospf neighbor`, `show ip bgp summary`, `show ip bgp`, `show ip bgp rib-failure`, `show ip route 30.0.0.0`, and `show ip route 13.0.0.1` on R1. A successful result would show the remote `30.0.0.0/8` installed as `B` while `13.0.0.1/32` remains reachable through OSPF. Then run `ping 30.1.1.1 source 10.1.1.1` to test forwarding. Check at least one remote LAN route on R2, R3, and R4 as well. Record the actual output before changing the portfolio's verified claims.

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
| IP route, next-hop, and ping results | Not supplied; forwarding remains unverified |
