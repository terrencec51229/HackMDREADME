<style>
.fontColor {
  color: #FF5733;
}
.fontColor2 {
  color: #96A5A5;
}
.fontColorH2 {
  color: #FF960F
}
.fontColorH3{
  color: #FFB432
}
.fontColorH4{
  color: #F08080
}
.fontFace {
  font-weight: Bold;
  font-style: Italic;
}
table th:first-of-type {
    width: 5%;
}
table th:nth-of-type(2) {
    width: 10%;
}
table th:nth-of-type(3) {
    width: 5%;
}
</style>

# Why We Need Segment Routing

[TOC]

## <span class="fontColorH2">The Control Plane Simplification</span> 

Intrinsically, Segment Routing (SR) could be treated as NG-MPLS. As the matter of fact, it really is. The main drivers for embracing SR instead of traditional MPLS (LDP/RSVP) are summarized below.

- <span class="fontColor">The optimization of the signaling. It is carried out by decoupling additional protocols.</span>
- ECMP supported. It is one of main shortages of RSVP.
- Easily distinguish the traffic is being stuck on which node from the troubleshooting aspect.

The reason why SR optimizes the signaling is becauase none of the signaling protocol is required to make SR function, the IGP (OSPF/ISIS) extentions take it over instead. Therefore, RSVP is not required at all for fulfilling MPLS TE in the SR domain.

### <span class="fontColorH3">What Does Segment Routing Differ Traditional LDP</span>
One of the significant differences in between is label allocation. By default, LDP allocates the label for both the node itself (loopback) and every link of that node. If TE has not been considered for the entire IP/MPLS transport then the link labels are not required at all. That is because ECMP is a native behavior of SR. <span class="fontColor">Unlike LDP, SR allocates the label for the node only by default.</span> The node and the link labels in the SR domain are called Prefix SID (segment identifier) and Adjacency SID (segment identifier) respectively.

## <span class="fontColorH2">dCloud: MPLS Segment Routing Introduction v2</span>

### <span class="fontColorH3">Overall Topology</span>

![Overall Topology](https://i.imgur.com/MKXe8uC.png)

### <span class="fontColorH3">SR Configuration</span>
Both the demo platform and the routing protocol are IOS-XRv and OSPF. Other than how to deploy the basic SR domain, the thing needs to be kept in mind in this section is that <span class="fontColor">when LDP and SR both are presenting LSP, LSP is preferred over LDP by default</span> (as the label *#2400x* shown in the 1st block). The design purpose is to ensure when the MPLS backbone migrates to SR from LDP as seamless as possible. Therefore, the behavior of LDP-preferable is able to be overridden (as the label *#1600x* shown in the 2nd block).

```shell=
# Source: XRv-1
# Destination: XRv-6

RP/0/0/CPU0:XR1(config)#do traceroute 10.10.6.6 verbose
Wed Jan 22 17:25:04.685 UTC
Type escape sequence to abort.
Tracing the route to 10.10.6.6
 1  172.16.3.3 [MPLS: Label 24007 Exp 0] 9 msec  9 msec  9 msec 
 2  172.16.4.4 [MPLS: Label 24004 Exp 0] 9 msec  9 msec  9 msec 
 3  172.16.6.6 9 msec  *  9 msec
```

```shell=
# Source: XRv-1
# Destination: XRv-6

router ospf 100
 segment-routing sr-prefer
!

RP/0/0/CPU0:XR1(config)#do traceroute 10.10.6.6 verbose 
Wed Jan 22 17:27:32.515 UTC
Type escape sequence to abort.
Tracing the route to 10.10.6.6
 1  172.16.3.3 [MPLS: Label 16006 Exp 0] 9 msec  9 msec  9 msec 
 2  172.16.4.4 [MPLS: Label 16006 Exp 0] 9 msec  9 msec  9 msec 
 3  172.16.6.6 9 msec  *  19 msec 
```

### <span class="fontColorH3">LDP Configuration Removal
</span>

Continue with the previous section. The most ideal way to seamlessly migrate traditional MPLS backbone from LDP to SR is...

- Enable SR at first. Both LDP and SR are coexistent in this stage.
- Override the forwarding path via SR. Both LDP and SR are still coexistent in this stage.
- Decommission LDP.

### <span class="fontColorH3">SRTE Configuration</span>

As presented by [Why We Need Segment Routing](https://hackmd.io/@terrencec51229/why-we-need-segment-routing) *(When Do We Need The Adjacency SID)*. There are two scenarios of TE in SR, per Prefix (node) level, and per Adjacency (link) level. No matter which one, one of the significant differences was presented in this section, <span class="fontColor">the TE tunnel does not need to be a pair in the SR domain, the one-way tunnel could function well.</span> That is why XRv-6 (the receiver) did not deploy any tunnels that point toward XRv-1 (the initiator).

![SRTE Configuration](https://i.imgur.com/nK4lFN1.png)

```shell=
# Source: XRv-1
# Destination: XRv-6

interface tunnel-te16
 ipv4 unnumbered Loopback0
 destination 10.10.6.6
 path-option 1 explicit name SRTE segment-routing
 autoroute announce
!

RP/0/0/CPU0:XR1(config)#do show route 10.10.6.6
Wed Jan 22 17:44:09.946 UTC
Routing entry for 10.10.6.6/32
  Known via "ospf 100", distance 110, metric 4, labeled SR, type intra area
  Installed Jan 22 17:40:32.111 for 00:03:37
  Routing Descriptor Blocks
    10.10.6.6, from 10.10.6.6, via tunnel-te16
      Route metric is 4
  No advertising protos. 
!

RP/0/0/CPU0:XR1(config)#do show cef 10.10.6.6/32
Wed Jan 22 17:44:23.845 UTC
10.10.6.6/32, version 51, internal 0x1000001 0x83 (ptr 0xa1422874) [1], 0x0 (0xa13edcb0), 0xa20 (0xa1583230)
 Updated Jan 22 17:40:32.131
 Prefix Len 32, traffic index 0, precedence n/a, priority 1
   via 10.10.6.6/32, tunnel-te16, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 0 NHID 0x0 [0xa10e45ec 0x0]
    next hop 10.10.6.6/32
    local adjacency
     labels imposed {None}
!

RP/0/0/CPU0:XR1(config)#do traceroute 10.10.6.6 verbose 
Wed Jan 22 17:44:46.464 UTC
Type escape sequence to abort.
Tracing the route to 10.10.6.6
 1   *  *  * 
 2   *  *  * 
 3  172.16.8.6 9 msec  *  9 msec
```

<span class="fontColor">The other thing is that SR supports ECMP natively.</span> If we only care about the destination XRv-6 instead of what the preferred path is then both XRv-2 and XRv-3 are not only just the valid next-hops but also the best next-hops to forward the traffic due to they have the same length forwarding path.

![ECMP Supported](https://i.imgur.com/926BfoX.png)

```shell=
# Source: XRv-1
# Destination: XRv-6

RP/0/0/CPU0:XR1(config)#do show mpls traffic-eng segment-routing 
Wed Jan 22 17:36:00.370 UTC

(omitted)

  IGP Id: 10.10.6.6, MPLS TE Id: 10.10.6.6

    Segment-Routing:
      TE Node-SID Index: 6
      SRGB Info: Start 16000, Size 8000

    Link[0]:Point-to-Point, Nbr IGP Id:10.10.5.5, Nbr Node Id:6, gen:23
      Frag Id:3, Intf Address:172.16.5.6, Intf Id:0
      Segment-Routing Adjacency-SIDs: 2
      Adjacency-SID[0]: 24010, Flags: V, L to Nbr:: IGP Id: 10.10.5.5, MPLS TE Id: 10.10.5.5
      Adjacency-SID[1]: 24009, Flags: B, V, L to Nbr:: IGP Id: 10.10.5.5, MPLS TE Id: 10.10.5.5
      Nbr Intf Address:172.16.5.5, Nbr Intf Id:0
      TE Metric:1, IGP Metric:1
      Ext Admin Group: 
        Length: 256 bits
        Value : 0x::
      Attribute Names:
      
    Link[1]:Point-to-Point, Nbr IGP Id:10.10.4.4, Nbr Node Id:5, gen:24
      Frag Id:4, Intf Address:172.16.6.6, Intf Id:0
      Segment-Routing Adjacency-SIDs: 2
      Adjacency-SID[0]: 24012, Flags: V, L to Nbr:: IGP Id: 10.10.4.4, MPLS TE Id: 10.10.4.4
      Adjacency-SID[1]: 24011, Flags: B, V, L to Nbr:: IGP Id: 10.10.4.4, MPLS TE Id: 10.10.4.4
      Nbr Intf Address:172.16.6.4, Nbr Intf Id:0
      TE Metric:1, IGP Metric:1
      Ext Admin Group: 
        Length: 256 bits
        Value : 0x::
      Attribute Names:
      
    Link[2]:Point-to-Point, Nbr IGP Id:10.10.4.4, Nbr Node Id:5, gen:25
      Frag Id:5, Intf Address:172.16.8.6, Intf Id:0
      Segment-Routing Adjacency-SIDs: 2
      Adjacency-SID[0]: 24014, Flags: V, L to Nbr:: IGP Id: 10.10.4.4, MPLS TE Id: 10.10.4.4
      Adjacency-SID[1]: 24013, Flags: B, V, L to Nbr:: IGP Id: 10.10.4.4, MPLS TE Id: 10.10.4.4
      Nbr Intf Address:172.16.8.4, Nbr Intf Id:0
      TE Metric:1, IGP Metric:1
      Ext Admin Group: 
        Length: 256 bits
        Value : 0x::
      Attribute Names: 
```

### <span class="fontColorH3">SR TI-LFA Protection</span>

Intrinsically, there is nothing different between LDP and SR from the LFA aspect.

![SR TI-LFA Protection](https://i.imgur.com/OKoMwUe.png)

```shell=
# Source: XRv-1
# Destination: XRv-6

router ospf 100
 area 0
  fast-reroute per-prefix
  fast-reroute per-prefix ti-lfa enable
 !
!

RP/0/0/CPU0:XR1(config)#do show route ospf
Wed Jan 22 17:54:40.953 UTC
Codes: C - connected, S - static, R - RIP, B - BGP, (>) - Diversion path
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - ISIS, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, su - IS-IS summary null, * - candidate default
       U - per-user static route, o - ODR, L - local, G  - DAGR, l - LISP
       A - access/subscriber, a - Application route
       M - mobile route, r - RPL, (!) - FRR Backup path

Gateway of last resort is not set

O    10.10.2.2/32 [110/2] via 172.16.1.2, 00:01:03, GigabitEthernet0/0/0/0
                  [110/0] via 172.16.3.3, 00:01:03, GigabitEthernet0/0/0/1 (!)
O    10.10.3.3/32 [110/0] via 172.16.1.2, 00:01:03, GigabitEthernet0/0/0/0 (!)
                  [110/2] via 172.16.3.3, 00:01:03, GigabitEthernet0/0/0/1
O    10.10.4.4/32 [110/0] via 172.16.1.2, 00:01:03, GigabitEthernet0/0/0/0 (!)
                  [110/3] via 172.16.3.3, 00:01:03, GigabitEthernet0/0/0/1
O    10.10.5.5/32 [110/3] via 172.16.1.2, 00:01:03, GigabitEthernet0/0/0/0
                  [110/0] via 172.16.3.3, 00:01:03, GigabitEthernet0/0/0/1 (!)
O    10.10.6.6/32 [110/4] via 10.10.6.6, 00:14:08, tunnel-te16
O    172.16.2.0/24 [110/2] via 172.16.1.2, 00:01:03, GigabitEthernet0/0/0/0
                   [110/0] via 172.16.3.3, 00:01:03, GigabitEthernet0/0/0/1 (!)
O    172.16.4.0/24 [110/0] via 172.16.1.2, 00:01:03, GigabitEthernet0/0/0/0 (!)
                   [110/2] via 172.16.3.3, 00:01:03, GigabitEthernet0/0/0/1
O    172.16.5.0/24 [110/3] via 172.16.1.2, 00:01:03, GigabitEthernet0/0/0/0
                   [110/0] via 172.16.3.3, 00:01:03, GigabitEthernet0/0/0/1 (!)
O    172.16.6.0/24 [110/0] via 172.16.1.2, 00:01:03, GigabitEthernet0/0/0/0 (!)
                   [110/3] via 172.16.3.3, 00:01:03, GigabitEthernet0/0/0/1
O    172.16.8.0/24 [110/0] via 172.16.1.2, 00:01:03, GigabitEthernet0/0/0/0 (!)
                   [110/3] via 172.16.3.3, 00:01:03, GigabitEthernet0/0/0/1
!

RP/0/0/CPU0:XR1(config)#do show mpls forwarding labels 16002 detail 
Wed Jan 22 17:57:56.580 UTC
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes       
Label  Label       or ID              Interface                    Switched    
------ ----------- ------------------ ------------ --------------- ------------
16002  Pop         SR Pfx (idx 2)     Gi0/0/0/0    172.16.1.2      0           
     Updated: Jan 22 17:53:37.538
     Path Flags: 0x400 [  BKUP-IDX:1 (0xa0c9c63c) ]
     Version: 54, Priority: 1
     Label Stack (Top -> Bottom): { Imp-Null }
     NHID: 0x0, Encap-ID: N/A, Path idx: 0, Backup path idx: 1, Weight: 0
     MAC/Encaps: 14/14, MTU: 1500
     Packets Switched: 0

       16006       SR Pfx (idx 2)     Gi0/0/0/1    172.16.3.3      0            (!)
     Updated: Jan 22 17:53:37.538
     Path Flags: 0x8300 [  IDX:1 BKUP, NoFwd ]
     Version: 54, Priority: 1
     Label Stack (Top -> Bottom): { 16006 16002 }
     NHID: 0x0, Encap-ID: N/A, Path idx: 1, Backup path idx: 0, Weight: 0
     MAC/Encaps: 14/22, MTU: 1500
     Packets Switched: 0
     (!): FRR pure backup

  Traffic-Matrix Packets/Bytes Switched: 0/0
```

### <span class=fontColorH3>Appendix</span>

#### <span class=fontColorH4>Segment Routing Global Block</span>

The default range of SRGB is from 16,000 through 23,999 (8,000 labels could be allocated in total). However, IOS-XR adopts different hierarchy allocation model for both IGP and BGP. As for IGP, the following behaviors are in sequence.

1. By the instance-level setup. Each instance is able to have respective definition.
2. By the global-level setup.
3. By the default range.

Unlike IGP, BGP supports the global-level setup only and there is no default range as well.

### <span class="fontColorH2">References</span>

> [Podcast: Network Collective - Introduction To Segment Routing](https://networkcollective.com/2019/03/ep47-intro-to-segment-routing/)

> [YouTube: iNE - Introduction to Segment Routing](https://www.youtube.com/playlist?list=PLCnmCTMRj5GwQGxXizASzwCfq7jFI2sCM)

> [YouTube: Juniper - Segment Routing (SR)](https://www.youtube.com/playlist?list=PLGvolzhkU_gQcpye7Xr5SzM_yYgGPZA0t)

:::info
###### tags: `Cisco` `dCloud` `SPN` `MPLS` `SR`
:::

{%hackmd BJrTq20hE %}