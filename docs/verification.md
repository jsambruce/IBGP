# Verification checklist

The latest EVE-NG export contains corrected startup configurations for all seven routers. CLI screenshots show live BGP session and table output after those changes. The screenshots are evidence from one capture, not a continuous health check.

## Full mesh: R1, R2, R3, R4

Already captured on **each** router:

```text
show ip bgp summary
show ip bgp
```

Each router is configured for three iBGP neighbors. In the supplied `show ip bgp summary` screenshots, all three peers on each router show a numeric `State/PfxRcd` of `1`, indicating established sessions on this IOS version. A state such as `Idle` or `Active` would not.

The screenshots show `10.0.0.0/8`, `20.0.0.0/8`, `30.0.0.0/8`, and `40.0.0.0/8` in each router's BGP table. For a concrete next check, run these on full-mesh **R1** and send the output:

```text
show ip bgp 30.0.0.0
show ip route 30.0.0.0
show ip route 13.0.0.1
ping 30.1.1.1 source 10.1.1.1
```

The saved OSPF configs also advertise all four LANs. A successful ping or a LAN route in the IP routing table therefore does **not**, by itself, prove that BGP carried the prefix. The `show ip route 30.0.0.0` output will identify the installed source. If testing BGP-only LAN routing is the goal, remove the LANs from OSPF as a separate, documented experiment after the underlay and BGP sessions are stable.

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
| Full-mesh session states | Three established peers per router in screenshots |
| Route-reflector session states | Both client sessions established in screenshots |
| BGP tables and reflected route | Both clients show the other client's LAN prefix |
| IP route, next-hop, and ping results | Not supplied; forwarding remains unverified |
