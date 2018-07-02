# EVPN-VXLAN lab Section 1 
## Centrally Routed Bridging Overlay architecture


### Lab topology
![Lab topology-1](topologies/evpn-vxlan-techfest_topo1.png)


### Lab objectives

The goal of the section 1 is to build the **Centrally Routed Bridging Overlay** architecture using the Juniper QFX series EVPN-VXLAN technologies to deliver L2 active/active forwarding within the same broadcast-domain(same vlan-id) between the hosts connected to CE1 and CE2. 

The end-host emulation is done at the CE1/CE2 and core3-re by the IRB interfaces mapped to the given vlan - irb.100 at the ce1-re and irb.100, irb.101 at ce2-re inside the routing-instance TEST 

The iBGP overlay using EVPN-type2 routes will be used in order to advertise the MAC@ and MAC+IP between the leafs of the same fabric. 
 
The inter-vni routing will be taking place at the spine1-re and spine2-re therefore the MAC+IP routes will be injected by the spine1-re/Spine2-re on behalf of the layer 2 leafs. 
 
Spine3-re is deployed in DC-2 in a pure IP routed mode - connected to Spine1-re/Spine2-re underlay using OSPFv2 area 1 NSSA no-summary with OSPFv2 default-route being injected by spine3-re towards spine1-re/spine2-re. 

The ultimate goal of the lab section 1 is to deliver:
 - L2 communication between CE1 (VNI-50100) and CE2 (VNI 50100)
 - L3 inter-vni communication CE-1 VNI-50100 to CE-2 VNI-50101 
 - L3 communication between the DC-1 and DC-2 core3-re connected hosts (emulated by irb.250)

Spine3-re is to be deployed in underlay eBGP mode and should advertise only the default-route via eBGP to the border-spines spine1-re/spine2-re. 

### Lab environment

The  environment is composed of the following vqfx nodes: 
- 3 x vQFX Spines ( Spine1-re/Spine2-re are the EVPN-VXLAN enabled spines in DC-1, Spine3-re in DC-2 is enabled with IP underlay routing only )
- 4 x vQFX Leafs (L2 leafs in Section-1 and L2L3 leafs in Section-2)
- 2 x vQFX CEs (CE1-re/CE2-re dual homed to EVPN-VXLAN fabric in DC-1
- 1 x vQFX core3-re IP underlay connected to the Spine3-re in DC-2

The underlay eBGP is already pre-provisioned in order to deliver full IP reachability between the loopback0.0 IP@.  

> All VMs are accessible from internet, so you can run everything from your laptop using SSH sessions 

Use the username: `root` and password: `Juniper1!`

Here's the access information to your POD : [my_pod_access_info](pod1/README.md)

### Lab section 1 tasks

`L1-task1`: verify the full IPv4 underlay reachability within the  [section 1 topology](topologies/evpn-vxlan-techfest_topo1.png)

`L1-task2`: provision and verify the overlay iBGP(spine1/spine2 as overlay route-reflectors)  with EVPN signaling at all DC-1 fabric nodes using the local ASN 64512 as shown on the diagram

`L1-taks3`: enable and verify the underlay and overlay IP-ECMP within routing-options forwarding-options and protocol bgp level

`L1-task4`: provision the VNI values at the VLAN level - create vlan100 with vxlan vni 50100 

`L1-task5`: set the protocol evpn encapsulation type to vxlan, extended-vni list to the vni numbers 50100 and 50101 and the multicast-mode. 
            Make sure each given vni under evpn vni-options has vrf-target `target:x:y` defined 

`L1-task6`: provision a global route-target community for the default-switch EVI - EVPN-route type-1 dedicated global target community

`L1-task7`: enable per VNI route-target communities to be used in the 

`L1-task8`: provision an import policy-options policy-statement MY-FAB-IMP-POLICY to accept the global EVI route-target community and accept the customized per VNI target communities.
          Make sure that when the new VNI gets provisioned it's not going to be rejected due to the final reject term. 

`L1-task9`: put in place the switch-options vtep-source-interface, unique route-distinguisher, vrf-import policy-statement configured in previous task as well as the global switch-options EVI vrf-target target:1:8888 (Type1-evpn route dedicated)
          The task6 provisioned global EVI vrf-target target:1:9999 is to be shared across all leaf nodes in the DC-1 and target:1:8888 for the spine1-re/spine2-re - set at the switch-options level

`L1-task10`: set the ESI 10 byte values all-active towards the CE1 and CE2
           ESI leaf1/leaf2 towards CE1: `00:01:01:01:01:01:01:01:01:01`
           ESI leaf3/leaf4 towards CE1: `00:01:02:02:02:02:02:02:02:02`
           
`L1-task11`: set the same active LACP system-id for the given AE interface towards the CE devices - same LACP system-id towards the given CE
            LACP system-id leaf1/leaf2: `00:00:01:00:00:01`
            LACP system-id leaf3/leaf4: `00:00:02:00:00:02`
                        
`L1-task12`: provision the active LACP protocol based aggregated AE interface at the CE1(dual homed to leaf1/leaf2) and CE2(dual homed to leaf3/leaf4)

`L1-task13`: enable the VLAN-ids on the LAG interfaces towards the CE1 and CE2

`L1-task14`: verify using local IRB.100 interfaces at CE1/CE2 that the L2 reachability works fine within the VNI 5100

`L1-task15`: verify the EVPN database and EVPN route information for the MAC@ 00:01:99:00:00:01 and 00:01:99:00:00:02 

`L1-task16`: provision at the spine1/spine2 the IRB-VGA IP gateway interfaces for vlan100 and vlan101 and allocate them into the routing-instance type virtual-router VRF-1
  
`L1-task17`: make sure the CE1 irb.100 sourced IP can ping the CE2 irb.101 destination IP

`L1-task18`: provision at leaf1 an additional regular extended community for the VNI 50100 and make sure the T2 MAC and MAC+IP routes at the leaf3/leaf4 gets the routes with an additional extended community 1:50100

`L1-task19`: at the spine3-re enabled the core ip routing connectivity between the DC-1 and DC-2 using OSPFv2 NSSA area 0.0.0.1 nssa no-summaries default-lsa default-metric 101 inside the existing routing-instance MY-IPVPN-1 virtual-router instance-type. 
             Same virtual-router instance MY-IPVRF-1 should be enabled in area 0.0.0.1 nssa at spine1-re/spine2-re in order to receive the default route 0.0.0.0/0 


| VLAN       | VNI           | Route-target  |
| ------------- |:-------------:| -----:|
| vlan100      | 50100      | target:1:100 |
| vlan101      | 50101      |  target:1:101 |


| Node-name     | Underlay ASN  | Overlay ASN | switch-options RD | lo0.0 IP@|
| ------------- |:-------------:| -----:|-----:|:-------------:|
| leaf1      | 65501 | 64512 | 1.1.1.1:1 | 1.1.1.1|
| leaf2      | 65502 | 64512   |   1.1.1.2:1 | 1.1.1.2|
| leaf3 | 65503      | 64512 |  1.1.1.3:1 | 1.1.1.3|
| leaf4 | 65504      | 64512 |   1.1.1.4:1 | 1.1.1.4| 
| spine1 | 65511      | 64512 |    1.1.1.11:1 | 1.1.1.11|
| spine2 | 65512      | 64512 |    1.1.1.12:1 | 1.1.1.12| 




### Solution guide for EVPN/VXLAN lab section 1 ####


Confirm connectivity to the leaf lo0 addresses of all leaf devices. 
These are exchanged via the underlay eBGP session and will be required to setup the overlay iBGP session between leaf devices

##### `L1-task1`: verify the full IPv4 underlay reachability within the  [section 1 topology](topologies/evpn-vxlan-techfest_topo1.png)

```
root@leaf1# run show bgp summary group underlay   
Groups: 2 Peers: 4 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
bgp.evpn.0           
                     124         60          0          0          0          0
inet.0               
                       8          8          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.10.2.1             65512      10792      10903       0       0 3d 10:46:09 Establ
  inet.0: 4/4/4/0
10.10.4.1             65511      10804      10902       0       0 3d 10:46:29 Establ
  inet.0: 4/4/4/0

{master:0}[edit]
root@leaf1# 
{master:0}
root@leaf1> ping 1.1.1.11 source 1.1.1.1

root@leaf1> ping 1.1.1.12 source 1.1.1.1

root@leaf1> ping 1.1.1.2 source 1.1.1.1

root@leaf1> ping 1.1.1.3 source 1.1.1.1

root@leaf1> ping 1.1.1.4 source 1.1.1.1
```

##### `L1-task2`: provision and verify the overlay iBGP(spine1/spine2 as overlay route-reflectors)  with EVPN signaling at all DC-1 fabric nodes using the local ASN 64512 as shown on the diagram

###### Leaf-1 ibgp overlay configuration 

```
root@leaf1# show protocols bgp group overlay 
type internal;
local-address 1.1.1.1;
family evpn {
    signaling;
}
local-as 64512;
multipath;
neighbor 1.1.1.11;
neighbor 1.1.1.12;

{master:0}[edit]
root@leaf1# 
```

###### Leaf-2 ibgp overlay configuration 

```
root@leaf2# show protocols bgp group overlay 
type internal;
local-address 1.1.1.2;
family evpn {
    signaling;
}
local-as 64512;
multipath multiple-as;
neighbor 1.1.1.11;
neighbor 1.1.1.12;

{master:0}[edit]
root@leaf2# 
```
###### Spine1 ibgp overlay configuration example 
```
root@spine1# show protocols bgp group overlay 
type internal;
local-address 1.1.1.11;
family evpn {
    signaling;
}
vpn-apply-export;
cluster 1.1.1.11;
local-as 64512;
multipath;
neighbor 1.1.1.12;
neighbor 1.1.1.1;
neighbor 1.1.1.2;
neighbor 1.1.1.3;
neighbor 1.1.1.4;

{master:0}[edit]
root@spine1# 
```
###### Spine2 ibgp overlay configuration example 
```
root@spine2# show protocols bgp group overlay 
type internal;
local-address 1.1.1.12;
family evpn {
    signaling;
}
vpn-apply-export;
cluster 1.1.1.12;
local-as 64512;
multipath;
neighbor 1.1.1.11;
neighbor 1.1.1.1;
neighbor 1.1.1.2;
neighbor 1.1.1.3;
neighbor 1.1.1.4;

{master:0}[edit]
root@spine2# 
```

##### `L1-taks3`: enable and verify the underlay and overlay IP-ECMP within routing-options forwarding-options and protocol bgp level

###### Leaf1 load-balancing example
```
root@leaf1# show policy-options policy-statement LB 
term term1 {
    from protocol evpn;
    then {
        load-balance per-packet;
    }
}

{master:0}[edit]
root@leaf1# show routing-options 
router-id 1.1.1.1;
autonomous-system 65501;
forwarding-table {
    export LB;
}

{master:0}[edit]
root@leaf1# 
root@leaf1# show protocols bgp group underlay multipath 
multiple-as;

{master:0}[edit]
root@leaf1# 
```

##### `L1-task4`: provision the VNI values at the VLAN level of all leafs and spine1/spine2

###### Leaf1 vlan config example
```
root@leaf1# show vlans         
vlan100 {
    vlan-id 100;
    ##
    ## Warning: requires 'vxlan' license
    ##
    vxlan {
        vni 50100;
        ingress-node-replication;
    }
}
vlan101 {
    vlan-id 101;
    ##
    ## Warning: requires 'vxlan' license
    ##
    vxlan {
        vni 50101;
        ingress-node-replication;
    }
}

{master:0}[edit]
root@leaf1# 

```
###### Spine1 vlan config example
```
root@spine1# show vlans 
vlan100 {
    vlan-id 100;
    l3-interface irb.100;
    vxlan {
        vni 50100;
        ingress-node-replication;
    }
}
vlan101 {
    vlan-id 101;
    l3-interface irb.101;
    vxlan {
        vni 50101;
        ingress-node-replication;
    }
}

{master:0}[edit]
root@spine1# 

```

##### `L1-task5`: set the protocol evpn encapsulation type to vxlan, extended-vni list to the vni numbers 50100 and 50101 and the multicast-mode. 
                  Make sure each given vni under evpn vni-options has vrf-target `target:x:y` defined

###### evpn protocol configuration example at spine1
```
root@spine1# show protocols evpn 
vni-options {
    vni 50100 {
        vrf-target target:1:100;
    }
    vni 50101 {
        vrf-target target:1:101;
    }
}
encapsulation vxlan;
multicast-mode ingress-replication;
extended-vni-list [ 50100 50101 ];

{master:0}[edit]
root@spine1# 
```

The same evpn configuration is to be used at spine2,leaf1/leaf2, leaf3/leaf4

##### `L1-task6`: provision a global route-target community target:1:9999 (at all leafs) and target:1:8888 at spine1/spine2 at the default-switch EVI
                  This is for the purpose of the AD EVPN-route type-1 dedicated global target community
                  
###### Leaf1 EVPN T1-route route route-target global EVPN Auto-Discovery dedicated route-target community. Same to be enabled at leaf2/leaf3/leaf4            
```
root@leaf1# show switch-options vrf-target 
target:1:9999;

{master:0}[edit]
root@leaf1# 
```
```
root@leaf1# show policy-options community MY-FAB-COMMUNITY    
members target:1:9999;

{master:0}[edit]
root@leaf1# 
root@leaf1# show policy-options community SPINE-ESI                                               
members target:1:8888;

{master:0}[edit]
root@leaf1# 
root@leaf1# show policy-options policy-statement MY-FABRIC-IMPORT term term1      
from community MY-FAB-COMMUNITY;
then accept;

{master:0}[edit]
root@leaf1# 
root@leaf1# show policy-options policy-statement MY-FABRIC-IMPORT term term-spine-esi 
from community SPINE-ESI;
then accept;

{master:0}[edit]
root@leaf1# 

```
###### Spine1 EVPN T1-route route-target global EVPN Auto-Discovery dedicated route-target community. Same to be enabled at spine2

```
root@spine1# show switch-options vrf-target 
target:1:8888;

{master:0}[edit]
root@spine1# 
root@spine1# show policy-options community SPINE-ESI 
members target:1:8888;

{master:0}[edit]
root@spine1# 
root@spine1# show policy-options community MY-FAB-COMMUNITY                               
members target:1:9999;

{master:0}[edit]
root@spine1# 
root@spine1# show policy-options policy-statement MY-FABRIC-IMPORT term term1             
from community MY-FAB-COMMUNITY;
then accept;

{master:0}[edit]
root@spine1# show policy-options policy-statement MY-FABRIC-IMPORT term term-spine-esi    
from community SPINE-ESI;
then accept;

{master:0}[edit]
root@spine1# 
```








