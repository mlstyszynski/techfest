# EVPN-VXLAN virtual-lab
## Centrally Routed Bridging Overlay architecture


### Lab topology
![Lab topology-1](topologies/evpn-vxlan-techfest_topo1.png)


### Lab objectives

The goal of the present lab is to build the **Centrally Routed Bridging Overlay** architecture using the Juniper QFX series EVPN-VXLAN technologies to deliver L2 active/active forwarding within the DC-1 between the hosts connected to CE1 and CE2 as well as L3 forwarding between the DC-1 and DC-2 using the unified VXLAN transport and evpn Type-5 routes for L3 prefix advertisement.  

The end-host emulation is done at the CE1/CE2 and core3-re by the IRB interfaces mapped to the given vlan - irb.100 at the ce1-re and irb.100, irb.101 at ce2-re inside the routing-instance TEST 

The iBGP overlay using EVPN-type2 routes will be used in order to advertise the MAC@ and MAC+IP in DC-1 and EVPN-type5 between the DC1 and DC2. 
 
The inter-vni routing will be taking place at the spine1-re and spine2-re therefore the MAC+IP routes will be injected by the spine1-re/Spine2-re on behalf of the layer 2 leafs. 
 
Spine3-re is deployed in DC-2 is connected to the same overlay ASN 64512 and  Spine1-re/Spine2-re but DC-1 to DC-2 exchanges only EVPN type-5 route for prefix-advertisement. 

The ultimate goal of the present lab is to deliver:
 - L2 communication between CE1 (VNI-50100) and CE2 (VNI 50100)
 - L3 inter-vni communication CE-1 VNI-50100 to CE-2 VNI-50101 
 - L3 communication between the DC-1 and DC-2 using EVPN type-5 routes and VXLAN transport 


### Lab environment

The  environment is composed of the following vqfx nodes: 
- 3 x vQFX Spines ( Spine1-re/Spine2-re are the EVPN-VXLAN enabled spines in DC-1, Spine3-re in DC-2 )
- 4 x vQFX Leafs in DC-1 
- 3 x vQFX CEs (CE1-re/CE2-re dual homed to EVPN-VXLAN fabric in DC-1 and single-homed CE3-re in DC-2 to spine3-re) 

The underlay eBGP is already pre-provisioned in order to deliver full IP reachability between the loopback0.0 IP@.  

> All VMs are accessible from internet, so you can run everything from your laptop using SSH sessions 

Use the username: `root` and password: `Juniper1!`

Here's the access information to your POD : [my_pod_access_info](pod1/README.md)

### Lab tasks

`L1-task1`: verify the full IPv4 underlay reachability within the  [main topology](topologies/evpn-vxlan-techfest_topo1.png)

`L1-task2`: provision and verify the overlay iBGP(spine1/spine2 as overlay route-reflectors)  with EVPN signaling at all DC-1 fabric nodes using the local ASN 64512 as shown on the diagram

`L1-taks3`: enable and verify the underlay and overlay IP-ECMP within routing-options forwarding-options and protocol bgp level

`L1-task4`: provision the VNI values at the VLAN level - create vlan100 with vxlan vni 50100 within the DC-1 fabric

`L1-task5`: set the protocol evpn encapsulation type to vxlan, extended-vni list to the vni numbers 50100 and 50101 and the multicast-mode. 
            Make sure each given vni under evpn vni-options has vrf-target `target:x:y` defined and corresponding to the one you'll be later importing in the MY-FAB-IMP-POLICY policy-statement

`L1-task6`: provision switch-options global route-target community for the default-switch EVI - EVPN-route type-1 dedicated global target community. Make sure it's also part of the accepted term in the import policy statement MY-FAB-IMP-POLICY. 

`L1-task7`: set the switch-options vtep-source-interface, unique route-distinguisher, vrf-import policy-statement configured in previous task as well as the global switch-options EVI vrf-target target:1:8888 (Type1-evpn route dedicated)
          The task6 provisioned global EVI vrf-target target:1:9999 is to be shared across all leaf nodes in the DC-1 and target:1:8888 for the spine1-re/spine2-re - set at the switch-options level. Make sure both of these are imported with the switch-option level vrf-import policy-statement; 

`L1-task8`: enable per VNI route-target communities for VNI 50100 target:1:100 and VNI 50101 target:1:101

`L1-task9`: provision an import policy-options policy-statement MY-FAB-IMP-POLICY to accept the global EVI route-target community and accept the customized per VNI target communities.
          Make sure that when the new VNI gets provisioned it's not going to be rejected due to the final reject term. 


`L1-task10`: set the ESI 10 byte values all-active towards the CE1 and CE2
           ESI leaf1/leaf2 towards CE1: `00:01:01:01:01:01:01:01:01:01`
           ESI leaf3/leaf4 towards CE1: `00:01:02:02:02:02:02:02:02:02`
           
`L1-task11`: set the same active LACP system-id for the given AE interface towards the CE devices - same LACP system-id towards the given CE
            LACP system-id leaf1/leaf2: `00:00:01:00:00:01`
            LACP system-id leaf3/leaf4: `00:00:02:00:00:02`
                        
`L1-task12`: provision the active LACP protocol based aggregated AE interface at the CE1(dual homed to leaf1/leaf2) and CE2(dual homed to leaf3/leaf4)

`L1-task13`: enable the VLAN-ids on the LAG interfaces towards the CE1 and CE2

`L1-task14`: verify using local IRB.100 interfaces at CE1/CE2 that the L2 reachability works fine within the VNI 50100

`L1-task15`: verify the EVPN database and EVPN route information for the MAC@ 00:01:99:00:00:01 and 00:01:99:00:00:02 

`L1-task16`: provision at the spine1/spine2 the IRB-VGA IP gateway interfaces for vlan100 and vlan101 and allocate them into the routing-instance type virtual-router VRF-1
  
`L1-task17`: make sure the CE1 irb.100 sourced IP can ping the CE2 irb.101 destination IP within the TEST routing-instance

`L1-task18`: provision at spine1 with an additional regular extended community for the VNI 50100 and make sure the T2 MAC and MAC+IP routes at the leaf3/leaf4 gets the routes with an additional extended community 1:50100

`L1-task19`: enable the IPv4 prefix exchange between DC-1 and DC-2 using EVPN Type-5 signaling and vxlan transport within the routing-instance name T5-VRF1, instance-type vrf. The new EVPN type-5 dedicated routing-instance should be enabled with interfaces irb.x used in the given data center and enabled with new loopback lo0.1 interface; Each Spine should advertise additionally a static discard route as type-5 route; We'll have to explicitly accept also the new route-target at the switch-options level; 


The switch-options and protocol evpn configuration are dependent so will need to be configured together in order to have the candidate commit configuration ready. 

| VLAN       | VNI           | Route-target  |
| ------------- |:-------------:| -----:|
| vlan100      | 50100      | target:1:100 |
| vlan101      | 50101      |  target:1:101 |
| vlan250      | 50250      | target:1:250  |



| Node-name     | Underlay ASN  | Overlay ASN | switch-options RD | lo0.0 IP@| switch-options vrf-target |
| ------------- |:-------------:| -----:|-----:| -------------:| -------------:|
| leaf1      | 65501 | 64512 | 1.1.1.1:1 | 1.1.1.1|target:1:9999|
| leaf2      | 65502 | 64512   |   1.1.1.2:1 | 1.1.1.2|target:1:9999|
| leaf3 | 65503      | 64512 |  1.1.1.3:1 | 1.1.1.3|target:1:9999|
| leaf4 | 65504      | 64512 |   1.1.1.4:1 | 1.1.1.4|target:1:9999| 
| spine1 | 65511      | 64512 |    1.1.1.11:1 | 1.1.1.11|target:1:8888|
| spine2 | 65512      | 64512 |    1.1.1.12:1 | 1.1.1.12|target:1:8888|
| spine3 | 65513      | 64512 | 1.1.1.13:1    | 1.1.1.13|target:1:8888| 

spine1/spine2/spine3 level IRB-VGA configurations: 

| VLAN       | VNI           | IRB IP@  | virtual-gateway-address |  virtual-gateway-v4-mac | 
| ------------- |:-------------:| -----:| -----:| -----:|
| vlan100      | 50100      | 150.100.1.1, 150.100.1.2 | 150.100.1.254 | 00:00:01:01:00:01|
| vlan101      | 50101      |  150.101.1.1, 150.101.1.2 | 150.101.1.254 | 00:00:02:02:00:02|
| vlan250      | 50250      | 150.250.1.1  | 150.250.1.254|   00:00:03:03:03:01|


Make sure the EVPN type-5 routes dedicated routing-instance has an additional lo0.1 enabled: 


| node-name       | RD           | T5 Route-Target  | T5 instance loopback0.1 |
| ------------- |:-------------:| -----:| -----:|
| spine1      | 1.1.1.111:1      |  target:64512:1000 | 1.1.1.111/32|
| spine2      | 1.1.1.112:1      |   target:64512:1000 |1.1.1.112/32|
| spine3      | 1.1.1.113:1      |  target:64512:1000  |1.1.1.113/32|

### Solution guide for EVPN/VXLAN lab ####


Confirm connectivity to the leaf lo0 addresses of all leaf devices. 
These are exchanged via the underlay eBGP session and will be required to setup the overlay iBGP session between leaf devices

##### `L1-task1`: verify the full IPv4 underlay reachability within the  [main topology](topologies/evpn-vxlan-techfest_topo1.png)

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

##### `L1-task6`: provision a global route-target community target:1:9999 (at all leafs) and target:1:8888 at spine1/spine2 at the default-switch EVI. This is for the purpose of the AD EVPN-route type-1 dedicated global target community.  Make sure it's also part of the import policy statement MY-FAB-IMP-POLICY. 
                  
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
##### `L1-task7`: set the switch-options vtep-source-interface, unique route-distinguisher, vrf-import policy-statement configured in previous task as well as the global switch-options EVI vrf-target target:1:8888 (Type1-evpn route dedicated). The global EVI vrf-target target:1:9999 is to be shared across all leaf nodes in the DC-1 and target:1:8888 for the spine1-re/spine2-re - set at the switch-options level
 
 ###### Leaf1 - switch-option configuration example with vtep-source-interface, unique route-distinguisher 
 
```
root@leaf1# show switch-options 
vtep-source-interface lo0.0;
route-distinguisher 1.1.1.1:1;
vrf-import MY-FABRIC-IMPORT;
vrf-target target:1:9999;

{master:0}[edit]
root@leaf1# 
```
 ###### Spine1 - switch-option configuration example with vtep-source-interface, unique route-distinguisher 
```
root@spine1# show switch-options 
vtep-source-interface lo0.0;
route-distinguisher 1.1.1.11:1;
vrf-import MY-FABRIC-IMPORT;
vrf-target target:1:8888;

{master:0}[edit]
root@spine1# 
```
As you can see each node will have to be provisioned with a different route-distinguisher. 
Spine1/Spine2 have different global vrf-target comparing to leafs but both have to be imported by all nodes within the policy-statement MY-FAB-COMMUNITY
##### `L1-task8`: enable per VNI route-target communities for VNI 50100 target:1:100 and VNI 50101 target:1:101

```
root@leaf1# show policy-options community COM-VNI-50100  
members target:1:100;

{master:0}[edit]
root@leaf1# show policy-options community COM-VNI-50101    
members target:1:101;

{master:0}[edit]
root@leaf1# 
```

##### `L1-task9`: provision an import policy-options policy-statement MY-FAB-IMP-POLICY to accept the global EVI route-target community and accept the customized per VNI target communities. Make sure that when the new VNI gets provisioned it's not going to be rejected due to the final reject term.

###### Leaf1 import policy-statement example - to be attached at the switch-options vrf-import 
```
root@leaf1# show policy-options policy-statement MY-FABRIC-IMPORT 
term term1 {
    from community MY-FAB-COMMUNITY;
    then accept;
}
term term-spine-esi {
    from community SPINE-ESI;
    then accept;
}
term term2 {
    from community COM-VNI-50100;
    then accept;
}
term term3 {
    from community COM-VNI-50101;
    then accept;
}
term term1000 {
    then reject;
}

{master:0}[edit]
root@leaf1#
root@leaf1# show switch-options vrf-import 
vrf-import MY-FABRIC-IMPORT;

{master:0}[edit]
root@leaf1# 
```

The same policy-statement is to be enabled at the leaf2/leaf3/leaf4. 

###### Spine1 import policy statement example - to be attached at the switch-options vrf-import 

```
root@spine1# show policy-options policy-statement MY-FABRIC-IMPORT 
term term1 {
    from community MY-FAB-COMMUNITY;
    then accept;
}
term term-spine-esi {
    from community SPINE-ESI;
    then accept;
}
term term2 {
    from community COM-VNI-50100;
    then accept;
}
term term3 {
    from community COM-VNI-50101;
    then accept;
}
term term1000 {
    then reject;
}

{master:0}[edit]
root@spine1# 
root@spine1# show switch-options vrf-import 
vrf-import MY-FABRIC-IMPORT;

{master:0}[edit]
root@spine1# 
```
The same policy-statement is to be enabled at the spine2


##### `L1-task10`: set the ESI 10 byte values all-active towards the CE1 and CE2
            ESI leaf1/leaf2 towards CE1: `00:01:01:01:01:01:01:01:01:01`
            ESI leaf3/leaf4 towards CE1: `00:01:02:02:02:02:02:02:02:02`
##### `L1-task11`: set the same active LACP system-id for the given AE interface towards the CE devices - same LACP system-id towards the given CE
             LACP system-id leaf1/leaf2: `00:00:01:00:00:01`
             LACP system-id leaf3/leaf4: `00:00:02:00:00:02`
            
###### Leaf1 and Leaf2 example of provisioning the aggregated interface towards the CE1. 

```
root@leaf1# show interfaces ae0 
esi {
    00:01:01:01:01:01:01:01:01:01;
    all-active;
}
aggregated-ether-options {
    lacp {
        active;
        system-id 00:00:01:00:00:01;
    }
}
unit 0 {
    family ethernet-switching {
        interface-mode trunk;
        vlan {
            members 100-101;
        }
    }
}

{master:0}[edit]
root@leaf1# 
```   
```
root@leaf2# show interfaces ae0 
esi {
    00:01:01:01:01:01:01:01:01:01;
    all-active;
}
aggregated-ether-options {
    lacp {
        active;
        system-id 00:00:01:00:00:01;
    }
}
unit 0 {
    family ethernet-switching {
        interface-mode trunk;
        vlan {
            members 100-101;
        }
    }
}

{master:0}[edit]
root@leaf2# 
```
###### Leaf3 and Leaf4 example of provisioning the aggregated interface towards the CE2

```
root@leaf3# show interfaces ae0 
esi {
    00:01:02:02:02:02:02:02:02:02;
    all-active;
}
aggregated-ether-options {
    lacp {
        active;
        system-id 00:00:02:00:00:02;
    }
}
unit 0 {
    family ethernet-switching {
        interface-mode trunk;
        vlan {
            members 100-101;
        }
    }
}

{master:0}[edit]
root@leaf3# 

```
```
root@leaf4# show interfaces ae0 
esi {
    00:01:02:02:02:02:02:02:02:02;
    all-active;
}
aggregated-ether-options {
    lacp {
        active;
        system-id 00:00:02:00:00:02;
    }
}
unit 0 {
    family ethernet-switching {
        interface-mode trunk;
        vlan {
            members 100-101;
        }
    }
}

{master:0}[edit]
root@leaf4# 
```

##### `L1-task12`: provision the active LACP protocol based aggregated AE interface at the CE1(dual homed to leaf1/leaf2) and CE2(dual homed to leaf3/leaf4)
##### `L1-task13`: enable the VLAN-ids on the LAG interfaces towards the CE1 and CE2

###### CE1 example of provisioning the dual homed aggregated interface towards the leaf1/leaf2 
```
root@ce1# show chassis aggregated-devices 
ethernet {
    device-count 1;
}

{master:0}[edit]
root@ce1# show interfaces ae0                
aggregated-ether-options {
    lacp {
        active;
    }
}
unit 0 {
    family ethernet-switching {
        interface-mode trunk;
        vlan {
            members 100-101;
        }
    }
}

{master:0}[edit]
root@ce1# 
root@ce1# show interfaces xe-0/0/0 
ether-options {
    802.3ad ae0;
}

{master:0}[edit]
root@ce1# 
root@ce1# show interfaces xe-0/0/1 
ether-options {
    802.3ad ae0;
}

{master:0}[edit]
root@ce1# 
```

The similar approach should be taken for CE2 connectivity towards the leaf3/leaf4


##### `L1-task14`: verify using local IRB.100 interfaces at CE1/CE2 that the L2 reachability works fine within the VNI 50100

```
root@ce1# run show interfaces terse routing-instance TEST 
Interface               Admin Link Proto    Local                 Remote
irb.100                 up    up   inet     150.100.1.100/24

{master:0}[edit]
root@ce1# 
root@ce1# run ping 150.100.1.101 source 150.100.1.100 routing-instance TEST    
PING 150.100.1.101 (150.100.1.101): 56 data bytes
64 bytes from 150.100.1.101: icmp_seq=0 ttl=64 time=11.495 ms
64 bytes from 150.100.1.101: icmp_seq=1 ttl=64 time=11.146 ms
64 bytes from 150.100.1.101: icmp_seq=2 ttl=64 time=11.164 ms
^C
--- 150.100.1.101 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 11.146/11.268/11.495/0.160 ms

{master:0}[edit]
root@ce1# 
```

```
root@leaf4# run show route table default-switch.evpn.0 evpn-mac-address 00:01:99:00:00:01 active-path 

default-switch.evpn.0: 50 destinations, 95 routes (48 active, 0 holddown, 4 hidden)
+ = Active Route, - = Last Active, * = Both

2:1.1.1.1:1::50100::00:01:99:00:00:01/304               
                   *[BGP/170] 01:46:30, localpref 100, from 1.1.1.11
                      AS path: I, validation-state: unverified
                      to 10.10.12.1 via xe-0/0/0.0
                    > to 10.10.10.1 via xe-0/0/1.0
2:1.1.1.2:1::50100::00:01:99:00:00:01/304               
                   *[BGP/170] 01:46:29, localpref 100, from 1.1.1.11
                      AS path: I, validation-state: unverified
                      to 10.10.12.1 via xe-0/0/0.0
                    > to 10.10.10.1 via xe-0/0/1.0
2:1.1.1.1:1::50100::00:01:99:00:00:01::150.100.1.100/304               
                   *[BGP/170] 01:46:26, localpref 100, from 1.1.1.12
                      AS path: I, validation-state: unverified
                      to 10.10.12.1 via xe-0/0/0.0
                    > to 10.10.10.1 via xe-0/0/1.0
2:1.1.1.2:1::50100::00:01:99:00:00:01::150.100.1.100/304               
                   *[BGP/170] 01:46:25, localpref 100, from 1.1.1.12
                      AS path: I, validation-state: unverified
                      to 10.10.12.1 via xe-0/0/0.0
                    > to 10.10.10.1 via xe-0/0/1.0

{master:0}[edit]
root@leaf4# 
root@leaf4# 
root@leaf4#


{master:0}[edit]
root@leaf4# run show route table default-switch.evpn.0 evpn-mac-address 00:01:99:00:00:01 active-path extensive    

default-switch.evpn.0: 50 destinations, 95 routes (48 active, 0 holddown, 4 hidden)
2:1.1.1.1:1::50100::00:01:99:00:00:01/304 (2 entries, 1 announced)
        *BGP    Preference: 170/-101
                Route Distinguisher: 1.1.1.1:1
                Next hop type: Indirect, Next hop index: 0
                Address: 0x9db6c70
                Next-hop reference count: 20
                Source: 1.1.1.11
                Protocol next hop: 1.1.1.1
                Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                State: <Secondary Active Int Ext>
                Local AS: 65504 Peer AS: 64512
                Age: 1:47:33    Metric2: 0 
                Validation State: unverified 
                Task: BGP_64512_64512.1.1.1.11
                Announcement bits (1): 0-default-switch-evpn 
                AS path: I (Originator)
                Cluster list:  1.1.1.11
                Originator ID: 1.1.1.1
                Communities: 64512:50100 target:1:100 encapsulation0:0:0:0:vxlan
                Import Accepted
                Route Label: 50100
                ESI: 00:01:01:01:01:01:01:01:01:01
                Localpref: 100
                Router ID: 1.1.1.11
                Primary Routing Table bgp.evpn.0
                Indirect next hops: 1
                        Protocol next hop: 1.1.1.1
                        Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                        Indirect path forwarding next hops: 2
                                Next hop type: Router
                                Next hop: 10.10.12.1 via xe-0/0/0.0
                                Session Id: 0x0
                                Next hop: 10.10.10.1 via xe-0/0/1.0
                                Session Id: 0x0
                        1.1.1.1/32 Originating RIB: inet.0
                          Node path count: 1
                          Forwarding nexthops: 2
                                Nexthop: 10.10.12.1 via xe-0/0/0.0

2:1.1.1.2:1::50100::00:01:99:00:00:01/304 (2 entries, 1 announced)
        *BGP    Preference: 170/-101
                Route Distinguisher: 1.1.1.2:1
                Next hop type: Indirect, Next hop index: 0
                Address: 0x9db7630      
                Next-hop reference count: 20
                Source: 1.1.1.11
                Protocol next hop: 1.1.1.2
                Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                State: <Secondary Active Int Ext>
                Local AS: 65504 Peer AS: 64512
                Age: 1:47:32    Metric2: 0 
                Validation State: unverified 
                Task: BGP_64512_64512.1.1.1.11
                Announcement bits (1): 0-default-switch-evpn 
                AS path: I (Originator)
                Cluster list:  1.1.1.11
                Originator ID: 1.1.1.2
                Communities: 64512:50100 target:1:100 encapsulation0:0:0:0:vxlan
                Import Accepted
                Route Label: 50100
                ESI: 00:01:01:01:01:01:01:01:01:01
                Localpref: 100
                Router ID: 1.1.1.11
                Primary Routing Table bgp.evpn.0
                Indirect next hops: 1
                        Protocol next hop: 1.1.1.2
                        Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                        Indirect path forwarding next hops: 2
                                Next hop type: Router
                                Next hop: 10.10.12.1 via xe-0/0/0.0
                                Session Id: 0x0
                                Next hop: 10.10.10.1 via xe-0/0/1.0
                                Session Id: 0x0
                        1.1.1.2/32 Originating RIB: inet.0
                          Node path count: 1
                          Forwarding nexthops: 2
                                Nexthop: 10.10.12.1 via xe-0/0/0.0

2:1.1.1.1:1::50100::00:01:99:00:00:01::150.100.1.100/304 (2 entries, 1 announced)
        *BGP    Preference: 170/-101
                Route Distinguisher: 1.1.1.1:1
                Next hop type: Indirect, Next hop index: 0
                Address: 0x9db6c70
                Next-hop reference count: 20
                Source: 1.1.1.12
                Protocol next hop: 1.1.1.1
                Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                State: <Secondary Active Int Ext>
                Local AS: 65504 Peer AS: 64512
                Age: 1:47:29    Metric2: 0 
                Validation State: unverified 
                Task: BGP_64512_64512.1.1.1.12
                Announcement bits (1): 0-default-switch-evpn 
                AS path: I
                Communities: target:1:100 encapsulation0:0:0:0:vxlan
                Import Accepted
                Route Label: 50100
                ESI: 00:01:01:01:01:01:01:01:01:01
                Localpref: 100
                Router ID: 1.1.1.12
                Primary Routing Table bgp.evpn.0
                Indirect next hops: 1
                        Protocol next hop: 1.1.1.1
                        Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                        Indirect path forwarding next hops: 2
                                Next hop type: Router
                                Next hop: 10.10.12.1 via xe-0/0/0.0
                                Session Id: 0x0
                                Next hop: 10.10.10.1 via xe-0/0/1.0
                                Session Id: 0x0
                        1.1.1.1/32 Originating RIB: inet.0
                          Node path count: 1
                          Forwarding nexthops: 2
                                Nexthop: 10.10.12.1 via xe-0/0/0.0

2:1.1.1.2:1::50100::00:01:99:00:00:01::150.100.1.100/304 (2 entries, 1 announced)
        *BGP    Preference: 170/-101
                Route Distinguisher: 1.1.1.2:1
                Next hop type: Indirect, Next hop index: 0
                Address: 0x9db7630
                Next-hop reference count: 20
                Source: 1.1.1.12
                Protocol next hop: 1.1.1.2
                Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                State: <Secondary Active Int Ext>
                Local AS: 65504 Peer AS: 64512
                Age: 1:47:28    Metric2: 0 
                Validation State: unverified 
                Task: BGP_64512_64512.1.1.1.12
                Announcement bits (1): 0-default-switch-evpn 
                AS path: I
                Communities: target:1:100 encapsulation0:0:0:0:vxlan
                Import Accepted
                Route Label: 50100      
                ESI: 00:01:01:01:01:01:01:01:01:01
                Localpref: 100
                Router ID: 1.1.1.12
                Primary Routing Table bgp.evpn.0
                Indirect next hops: 1
                        Protocol next hop: 1.1.1.2
                        Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                        Indirect path forwarding next hops: 2
                                Next hop type: Router
                                Next hop: 10.10.12.1 via xe-0/0/0.0
                                Session Id: 0x0
                                Next hop: 10.10.10.1 via xe-0/0/1.0
                                Session Id: 0x0
                        1.1.1.2/32 Originating RIB: inet.0
                          Node path count: 1
                          Forwarding nexthops: 2
                                Nexthop: 10.10.12.1 via xe-0/0/0.0

{master:0}[edit]
root@leaf4# 

root@leaf4> show ethernet-switching table 

MAC flags (S - static MAC, D - dynamic MAC, L - locally learned, P - Persistent static
           SE - statistics enabled, NM - non configured MAC, R - remote PE MAC, O - ovsdb MAC)


Ethernet switching table : 9 entries, 9 learned
Routing instance : default-switch
   Vlan                MAC                 MAC      Logical                Active                        
   name                address             flags    interface              source                       
   vlan100             00:00:01:01:00:01   DR       esi.1739               05:00:00:ff:e8:00:00:c3:b4:00  
   vlan100             00:01:99:00:00:01   DR       esi.1749               00:01:01:01:01:01:01:01:01:01  
   vlan100             00:01:99:00:00:02   DLR      ae0.0                
   vlan100             02:05:86:71:47:00   D        vtep.32769             1.1.1.11                       
   vlan100             02:05:86:71:cb:00   D        vtep.32770             1.1.1.12                       
   vlan101             00:00:02:02:00:02   DR       esi.1738               05:00:00:ff:e8:00:00:c3:b5:00  
   vlan101             00:01:88:00:00:02   DLR      ae0.0                
   vlan101             02:05:86:71:47:00   D        vtep.32769             1.1.1.11                       
   vlan101             02:05:86:71:cb:00   D        vtep.32770             1.1.1.12                       

{master:0}
root@leaf4> 

root@leaf4> show ethernet-switching vxlan-tunnel-end-point remote mac-table 

MAC flags (S -static MAC, D -dynamic MAC, L -locally learned, C -Control MAC
           SE -Statistics enabled, NM -Non configured MAC, R -Remote PE MAC)

Logical system   : <default>
Routing instance : default-switch
 Bridging domain : vlan100+100, VLAN : 100, VNID : 50100
   MAC                 MAC      Logical          Remote VTEP
   address             flags    interface        IP address
   00:00:01:01:00:01   DR       esi.1739         1.1.1.12     
   00:01:99:00:00:01   DR       esi.1749         1.1.1.2 1.1.1.1 
   02:05:86:71:47:00   D        vtep.32769       1.1.1.11     
   02:05:86:71:cb:00   D        vtep.32770       1.1.1.12     

MAC flags (S -static MAC, D -dynamic MAC, L -locally learned, C -Control MAC
           SE -Statistics enabled, NM -Non configured MAC, R -Remote PE MAC)

 Bridging domain : vlan101+101, VLAN : 101, VNID : 50101
   MAC                 MAC      Logical          Remote VTEP
   address             flags    interface        IP address
   00:00:02:02:00:02   DR       esi.1738         1.1.1.12     
   02:05:86:71:47:00   D        vtep.32769       1.1.1.11     
   02:05:86:71:cb:00   D        vtep.32770       1.1.1.12     

{master:0}
root@leaf4> 
root@leaf4> show route forwarding-table destination 00:01:99:00:00:01 extensive 
Routing table: default-switch.evpn-vxlan [Index 3] 
Bridging domain: vlan100.evpn-vxlan [Index 3] 
VPLS:
    
Destination:  00:01:99:00:00:01/48
  Learn VLAN: 0                        Route type: user                  
  Route reference: 0                   Route interface-index: 544 
  Multicast RPF nh index: 0         
  IFL generation: 0                    Epoch: 0   
  Sequence Number: 0                   Learn Mask: 0x4000000000000000030000000000000000000000
  L2 Flags: control_dyn, esi
  Flags: sent to PFE
  Next-hop type: indirect              Index: 131079   Reference: 2    
  Nexthop:  
  Next-hop type: composite             Index: 1749     Reference: 2    
  Nexthop:  
  Next-hop type: composite             Index: 1750     Reference: 6    
  Next-hop type: indirect              Index: 131081   Reference: 3    
  Next-hop type: unilist               Index: 131076   Reference: 5    
  Nexthop: 10.10.12.1
  Next-hop type: unicast               Index: 1732     Reference: 9    
  Next-hop interface: xe-0/0/0.0       Weight: 0x0  
  Nexthop: 10.10.10.1
  Next-hop type: unicast               Index: 1733     Reference: 9    
  Next-hop interface: xe-0/0/1.0       Weight: 0x0  
  Nexthop:  
  Next-hop type: composite             Index: 1740     Reference: 6    
  Next-hop type: indirect              Index: 131077   Reference: 3    
  Next-hop type: unilist               Index: 131076   Reference: 5    
  Nexthop: 10.10.12.1
  Next-hop type: unicast               Index: 1732     Reference: 9    
  Next-hop interface: xe-0/0/0.0       Weight: 0x0  
  Nexthop: 10.10.10.1
  Next-hop type: unicast               Index: 1733     Reference: 9    
  Next-hop interface: xe-0/0/1.0       Weight: 0x0  

{master:0}
root@leaf4> 

root@leaf4> show ethernet-switching vxlan-tunnel-end-point esi    
ESI                           RTT                      VLNBH INH     ESI-IFL   LOC-IFL   #RVTEPs
00:01:01:01:01:01:01:01:01:01 default-switch           1749  131079  esi.1749            2    
    RVTEP-IP             RVTEP-IFL      VENH     MASK-ID   FLAGS
    1.1.1.2              vtep.32773     1750     1         2         
    1.1.1.1              vtep.32771     1740     0         2         
ESI                           RTT                      VLNBH INH     ESI-IFL   LOC-IFL   #RVTEPs
00:01:02:02:02:02:02:02:02:02 default-switch           1752  131083  esi.1752  ae0.0     1    
    RVTEP-IP             RVTEP-IFL      VENH     MASK-ID   FLAGS
    1.1.1.3              vtep.32774     1751     0         2         
ESI                           RTT                      VLNBH INH     ESI-IFL   LOC-IFL   #RVTEPs
05:00:00:ff:e7:00:00:c3:b4:00 default-switch           1736  131071  esi.1736            1    
    RVTEP-IP             RVTEP-IFL      VENH     MASK-ID   FLAGS
    1.1.1.11             vtep.32769     1734     0         2         
ESI                           RTT                      VLNBH INH     ESI-IFL   LOC-IFL   #RVTEPs
05:00:00:ff:e7:00:00:c3:b5:00 default-switch           1735  131070  esi.1735            1    
    RVTEP-IP             RVTEP-IFL      VENH     MASK-ID   FLAGS
    1.1.1.11             vtep.32769     1734     0         2         
ESI                           RTT                      VLNBH INH     ESI-IFL   LOC-IFL   #RVTEPs
05:00:00:ff:e8:00:00:c3:b4:00 default-switch           1739  131073  esi.1739            1    
    RVTEP-IP             RVTEP-IFL      VENH     MASK-ID   FLAGS
    1.1.1.12             vtep.32770     1737     0         2         
ESI                           RTT                      VLNBH INH     ESI-IFL   LOC-IFL   #RVTEPs
05:00:00:ff:e8:00:00:c3:b5:00 default-switch           1738  131072  esi.1738            1    
    RVTEP-IP             RVTEP-IFL      VENH     MASK-ID   FLAGS
    1.1.1.12             vtep.32770     1737     0         2         
ESI                           RTT                      VLNBH INH     ESI-IFL   LOC-IFL   #RVTEPs
05:00:00:ff:e9:00:00:c4:4a:00 default-switch           1748  131078  esi.1748            1    
    RVTEP-IP             RVTEP-IFL      VENH     MASK-ID   FLAGS
    1.1.1.13             vtep.32772     1747     0         2         

root@leaf4> 
root@leaf4> show ethernet-switching vxlan-tunnel-end-point remote summary 
Logical System Name       Id  SVTEP-IP         IFL   L3-Idx
<default>                 0   1.1.1.4          lo0.0    0  
 RVTEP-IP         IFL-Idx   NH-Id
 1.1.1.1          570       1740     
 1.1.1.2          572       1750     
 1.1.1.3          573       1751     
 1.1.1.11         568       1734     
 1.1.1.12         569       1737     
 1.1.1.13         571       1747     

{master:0}
root@leaf4> 

```

##### `L1-task15`: verify the EVPN database and EVPN route information for the MAC@ 00:01:99:00:00:01 and 00:01:99:00:00:02 

```
root@leaf4> show evpn database mac-address 00:01:99:00:00:01 extensive    
Instance: default-switch

VN Identifier: 50100, MAC address: 00:01:99:00:00:01
  Source: 00:01:01:01:01:01:01:01:01:01, Rank: 1, Status: Active
    Remote origin: 1.1.1.1
    Remote origin: 1.1.1.2
    Timestamp: Jul 11 10:38:51 (0x5b45de3b)
    State: <Remote-To-Local-Adv-Done>
    IP address: 150.100.1.100
      Remote origin: 1.1.1.1
      Remote origin: 1.1.1.2

{master:0}
root@leaf4> 
```
Make sure the remote-origine IP@ corresponds to the route table information for the given MAC@ 

##### `L1-task16`: provision at the spine1/spine2 the IRB-VGA IP gateway interfaces for vlan100 and vlan101 and allocate them into the routing-instance type virtual-router VRF-1

###### EVPN IRB-VGA (Virtual-Gateway-Address) configuration to be used at spine1/spine2  
```
root@spine1# show interfaces irb 
unit 100 {
    proxy-macip-advertisement;
    virtual-gateway-accept-data;
    family inet {
        address 150.100.1.1/24 {
            preferred;
            virtual-gateway-address 150.100.1.254;
        }
    }
    virtual-gateway-v4-mac 00:00:01:01:00:01;
}
unit 101 {
    proxy-macip-advertisement;
    virtual-gateway-accept-data;
    family inet {
        address 150.101.1.1/24 {
            preferred;
            virtual-gateway-address 150.101.1.254;
        }
    }
    virtual-gateway-v4-mac 00:00:02:02:00:02;
}                                       

{master:0}[edit]
root@spine1# 
```
```
root@spine2# show interfaces irb 
unit 100 {
    proxy-macip-advertisement;
    virtual-gateway-accept-data;
    family inet {
        address 150.100.1.2/24 {
            preferred;
            virtual-gateway-address 150.100.1.254;
        }
    }
    virtual-gateway-v4-mac 00:00:01:01:00:01;
}
unit 101 {
    proxy-macip-advertisement;
    virtual-gateway-accept-data;
    family inet {
        address 150.101.1.2/24 {
            preferred;
            virtual-gateway-address 150.101.1.254;
        }
    }
    virtual-gateway-v4-mac 00:00:02:02:00:02;
}                                       

{master:0}[edit]
root@spine2# 
```
Make sure the spine1/spine2 injects the Type-2 evpn routes with MAC+IP on behalf of the leafs 
This statement is only required when the Centrally Routed Overlay architecture is used for evpn-vxlan.  
```
root@spine1# show protocols evpn default-gateway 
default-gateway no-gateway-community;

{master:0}[edit]
root@spine1# 
root@spine2# show protocols evpn default-gateway 
default-gateway no-gateway-community;

{master:0}[edit]
root@spine2# 
```

##### `L1-task17`: make sure the CE1 irb.100 sourced IP can ping the CE2 irb.101 destination IP

```
root@ce1# run ping 150.101.1.101 source 150.100.1.100 routing-instance TEST    
PING 150.101.1.101 (150.101.1.101): 56 data bytes
64 bytes from 150.101.1.101: icmp_seq=0 ttl=64 time=13.110 ms
64 bytes from 150.101.1.101: icmp_seq=1 ttl=64 time=11.212 ms
64 bytes from 150.101.1.101: icmp_seq=2 ttl=64 time=11.212 ms
^C
--- 150.101.1.101 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 11.212/11.845/13.110/0.895 ms

{master:0}[edit]
root@ce1# 
```

##### `L1-task18`: provision at spine1 with an additional regular extended community for the VNI 50100 and make sure the T2 MAC and MAC+IP routes at the leaf3/leaf4 gets the routes with an additional extended community 1:50100

```
root@spine1# show policy-options policy-statement CUSTOM-100    
term term1 {
    from {
        family evpn;
        community COM-VNI-50100;
        nlri-route-type 2;
    }
    then {
        community add ADD-COMMUNITY-100;
        accept;
    }
}
term term100 {
    then accept;
}

{master:0}[edit]
root@spine1# 
```
```
root@spine1# show policy-options community ADD-COMMUNITY-100 
members 64512:50100;

{master:0}[edit]
root@spine1# show protocols bgp group overlay export 
export CUSTOM-100;

{master:0}[edit]
root@spine1# 
```
```
root@leaf3# run show route evpn-mac-address 00:01:99:00:00:01 extensive community 64512:50100 active-path    

inet.0: 19 destinations, 24 routes (18 active, 0 holddown, 1 hidden)

:vxlan.inet.0: 18 destinations, 18 routes (17 active, 0 holddown, 1 hidden)

inet6.0: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)

bgp.evpn.0: 62 destinations, 124 routes (60 active, 0 holddown, 4 hidden)
2:1.1.1.1:1::50100::00:01:99:00:00:01/304 MAC/IP (2 entries, 0 announced)
        *BGP    Preference: 170/-101
                Route Distinguisher: 1.1.1.1:1
                Next hop type: Indirect, Next hop index: 0
                Address: 0xb6260b0
                Next-hop reference count: 32
                Source: 1.1.1.11
                Protocol next hop: 1.1.1.1
                Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                State: <Active Int Ext>
                Local AS: 65505 Peer AS: 64512
                Age: 45         Metric2: 0 
                Validation State: unverified 
                Task: BGP_64512_64512.1.1.1.11+179
                AS path: I  (Originator)
                Cluster list:  1.1.1.11
                Originator ID: 1.1.1.1
                Communities: 64512:50100 target:1:100 encapsulation:vxlan(0x8)
                Import Accepted
                Route Label: 50100
                ESI: 00:01:01:01:01:01:01:01:01:01
                Localpref: 100
                Router ID: 1.1.1.11
                Secondary Tables: default-switch.evpn.0
                Indirect next hops: 1
                        Protocol next hop: 1.1.1.1
                        Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                        Indirect path forwarding next hops: 2
                                Next hop type: Router
                                Next hop: 10.10.15.1 via et-0/0/48.0
                                Session Id: 0x0
                                Next hop: 10.10.16.1 via et-0/0/49.0
                                Session Id: 0x0
                        1.1.1.1/32 Originating RIB: inet.0
                          Node path count: 1
                          Forwarding nexthops: 2
                                Nexthop: 10.10.15.1 via et-0/0/48.0
                                Session Id: 0
                                Nexthop: 10.10.16.1 via et-0/0/49.0
                                Session Id: 0

2:1.1.1.1:1::50100::00:01:99:00:00:01::150.100.1.100/304 MAC/IP (2 entries, 0 announced)
        *BGP    Preference: 170/-101
                Route Distinguisher: 1.1.1.1:1
                Next hop type: Indirect, Next hop index: 0
                Address: 0xb6260b0
                Next-hop reference count: 32
                Source: 1.1.1.11
                Protocol next hop: 1.1.1.1
                Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                State: <Active Int Ext>
                Local AS: 65505 Peer AS: 64512
                Age: 45         Metric2: 0 
                Validation State: unverified 
                Task: BGP_64512_64512.1.1.1.11+179
                AS path: I 
                Communities: 64512:50100 target:1:100 encapsulation:vxlan(0x8)
                Import Accepted
                Route Label: 50100
                ESI: 00:01:01:01:01:01:01:01:01:01
                Localpref: 100
                Router ID: 1.1.1.11
                Secondary Tables: default-switch.evpn.0
                Indirect next hops: 1
                        Protocol next hop: 1.1.1.1
                        Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                        Indirect path forwarding next hops: 2
                                Next hop type: Router
                                Next hop: 10.10.15.1 via et-0/0/48.0
                                Session Id: 0x0
                                Next hop: 10.10.16.1 via et-0/0/49.0
                                Session Id: 0x0
                        1.1.1.1/32 Originating RIB: inet.0
                          Node path count: 1
                          Forwarding nexthops: 2
                                Nexthop: 10.10.15.1 via et-0/0/48.0
                                Session Id: 0
                                Nexthop: 10.10.16.1 via et-0/0/49.0
                                Session Id: 0

```

##### `L1-task19`: enable the IPv4 prefix exchange between DC-1 and DC-2 using EVPN Type-5 signaling and vxlan transport using the routing-instance name T5-VRF1 , instance-type vrf. The Type-5 routing-instance should be enabled with interfaces irb.x used in the given data center and enabled with new loopback lo0.1 interface;Each Spine should advertise additionally a static discard route as type-5 route; 


Make sure the EVPN type-5 routes dedicated routing-instance has an additional lo0.1 enabled: 


| node-name       | RD           | T5 Route-Target  | T5 instance loopback0.1 |
| ------------- |:-------------:| -----:| -----:|
| spine1      | 1.1.1.111:1      |  target:64512:1000 | 1.1.1.111/32|
| spine2      | 1.1.1.112:1      |   target:64512:1000 |1.1.1.112/32|
| spine3      | 1.1.1.113:1      |  target:64512:1000  |1.1.1.113/32| 


Enable the underlay eBGP peerings between the DC-1 and DC-2 and then the iBGP overlay peerings with evpn signaling: 

```
root@spine3# show protocols bgp 
group dci {
    type internal;
    local-address 1.1.1.13;
    family evpn {
        signaling;
    }
    vpn-apply-export;
    local-as 64512;
    neighbor 1.1.1.11;
    neighbor 1.1.1.12;
}
group underlay {
    type external;
    export MY_VTEPS;
    neighbor 10.10.5.1 {
        peer-as 65511;
    }
    neighbor 10.10.6.1 {
        peer-as 65512;
    }
}

{master:0}[edit]
root@spine3# 

## enabled at spine1/spine2/spine3 the Type5 EVPN routing-instance - different route-distinguisher at each spine but same route-target: 

root@spine3# show routing-instances 
T5-VRF1 {
    instance-type vrf;
    interface irb.250;
    interface lo0.1;
    route-distinguisher 1.1.1.113:1;
    vrf-target target:64512:1000;
    vrf-table-label;
    routing-options {
        static {
            route 100.100.100.103/32 discard;
        }
        multipath;
    }
    protocols {
        evpn {
            ip-prefix-routes {
                advertise direct-nexthop;
                encapsulation vxlan;
                vni 1100;
                export TYPE5-POLICY;
            }
        }
    }
}

{master:0}[edit]
root@spine3# 

## advertise the local irb.x interface as well as a dummy T5 local static route to your iBGP type5 neighbors - this is related to the current JunOS implementation where you need to advertise at least one prefix in order to receive one; 

root@spine3# show policy-options policy-statement TYPE5-POLICY            
term term1 {
    from {
        route-filter 100.100.100.103/32 exact;
        route-filter 150.250.1.0/24 exact;
    }
    then accept;
}
term term1000 {
    then reject;
}

{master:0}[edit]
root@spine3# 

root@spine3# show policy-options 
policy-statement LB {
    term term1 {
        from protocol evpn;
        then {
            load-balance per-packet;
        }
    }
}

## make sure the T5 route-target is added as accepted route-target in the switch-options import policy-statement

policy-statement MY-FABRIC-IMPORT {
    term term1 {
        from community MY-FAB-COMMUNITY;
        then accept;
    }
    term term-spine-esi {
        from community SPINE-ESI;
        then accept;
    }
    term term2 {
        from community COM-VNI-50250;
        then accept;
    }
    term term5 {
        from community T5-COM1;
        then accept;
    }
}
policy-statement MY_VTEPS {
    term term1 {
        from {
            route-filter 1.1.1.0/24 prefix-length-range /32-/32;
        }
        then accept;
    }
    term term2 {
        then reject;
    }
}
policy-statement TYPE5-POLICY {
    term term1 {
        from {
            route-filter 100.100.100.103/32 exact;
            route-filter 150.250.1.0/24 exact;
        }
        then accept;
    }
    term term1000 {
        then reject;
    }
}
community COM-VNI-50250 members target:1:250;
community MY-FAB-COMMUNITY members target:1:9999;
community SPINE-ESI members target:1:8888;
community T5-COM1 members target:64512:1000;

{master:0}[edit]
root@spine3# 

root@spine1> show configuration routing-instances 
T5-VRF1 {
    instance-type vrf;
    interface irb.100;
    interface irb.101;
    interface lo0.1;
    route-distinguisher 1.1.1.111:1;
    vrf-target target:64512:1000;
    vrf-table-label;
    routing-options {
        static {
            route 100.100.100.100/32 discard;
        }
        multipath;
    }
    protocols {
        evpn {
            ip-prefix-routes {
                advertise direct-nexthop;
                encapsulation vxlan;
                vni 1100;
                export TYPE5-POLICY;
            }
        }
    }
}

{master:0}
root@spine1> 
root@spine1> show configuration policy-options 
policy-statement LB {
    term term1 {
        from protocol evpn;
        then {
            load-balance per-packet;
        }
    }
}
policy-statement MY-FABRIC-IMPORT {
    term term1 {
        from community MY-FAB-COMMUNITY;
        then accept;
    }
    term term-spine-esi {
        from community SPINE-ESI;
        then accept;
    }
    term term2 {
        from community COM-VNI-50100;
        then accept;
    }
    term term3 {
        from community COM-VNI-50101;
        then accept;
    }
    term term5 {
        from community T5-COM1;
        then accept;
    }
}
policy-statement MY_VTEPS {
    term term1 {
        from {
            route-filter 1.1.1.0/24 prefix-length-range /32-/32;
        }
        then accept;
    }
    term term2 {
        then reject;
    }
}
policy-statement TYPE5-POLICY {
    term term1 {
        from {
            route-filter 150.100.1.0/24 exact;
            route-filter 150.101.1.0/24 exact;
            route-filter 100.100.100.100/32 exact;
        }
        then accept;
    }
    term term1000 {
        then reject;
    }
}
community COM-VNI-50100 members target:1:100;
community COM-VNI-50101 members target:1:101;
community MY-FAB-COMMUNITY members target:1:9999;
community SPINE-ESI members target:1:8888;
community T5-COM1 members target:64512:1000;

{master:0}
root@spine1> 

root@spine2> show configuration routing-instances 
T5-VRF1 {
    instance-type vrf;
    interface irb.100;
    interface irb.101;
    interface lo0.1;
    route-distinguisher 1.1.1.112:1;
    vrf-target target:64512:1000;
    routing-options {
        static {
            route 100.100.100.101/32 discard;
        }
        multipath;
    }
    protocols {
        evpn {
            ip-prefix-routes {
                advertise direct-nexthop;
                encapsulation vxlan;
                vni 1100;
                export TYPE5-POLICY;
            }
        }
    }
}

{master:0}
root@spine2> show configuration policy-options 
policy-statement LB {
    term term1 {
        from protocol evpn;
        then {
            load-balance per-packet;
        }
    }
}
policy-statement MY-FABRIC-IMPORT {
    term term1 {
        from community MY-FAB-COMMUNITY;
        then accept;
    }
    term term-spine-esi {
        from community SPINE-ESI;
        then accept;
    }
    term term2 {
        from community COM-VNI-50100;
        then accept;
    }
    term term3 {
        from community COM-VNI-50101;
        then accept;
    }
    term term5 {
        from community T5-COM1;
        then accept;
    }
}
policy-statement MY_VTEPS {
    term term1 {
        from {
            route-filter 1.1.1.0/24 prefix-length-range /32-/32;
        }
        then accept;
    }
    term term2 {
        then reject;
    }
}
policy-statement TYPE5-POLICY {
    term term1 {
        from {
            route-filter 150.100.1.0/24 exact;
            route-filter 150.101.1.0/24 exact;
            route-filter 100.100.100.101/32 exact;
        }
        then accept;
    }
    term term1000 {
        then reject;
    }
}
community COM-VNI-50100 members target:1:100;
community COM-VNI-50101 members target:1:101;
community MY-FAB-COMMUNITY members target:1:9999;
community SPINE-ESI members target:1:8888;
community T5-COM1 members target:64512:1000;

{master:0}
root@spine2> 

```

EVPN type5 verification: 

```
root@spine3> show evpn ip-prefix-database 
L3 context: T5-VRF1

IPv4->EVPN Exported Prefixes
Prefix                                       EVPN route status
100.100.100.103/32                           Created
150.250.1.0/24                               Created

EVPN->IPv4 Imported Prefixes
Prefix                                       Etag
100.100.100.100/32                           0       
  Route distinguisher    VNI/Label  Router MAC         Nexthop/Overlay GW/ESI
  1.1.1.111:1            1100       02:05:86:71:94:00  1.1.1.11
100.100.100.101/32                           0       
  Route distinguisher    VNI/Label  Router MAC         Nexthop/Overlay GW/ESI
  1.1.1.112:1            1100       02:05:86:71:62:00  1.1.1.12
150.100.1.0/24                               0       
  Route distinguisher    VNI/Label  Router MAC         Nexthop/Overlay GW/ESI
  1.1.1.111:1            1100       02:05:86:71:94:00  1.1.1.11
  1.1.1.112:1            1100       02:05:86:71:62:00  1.1.1.12
150.101.1.0/24                               0       
  Route distinguisher    VNI/Label  Router MAC         Nexthop/Overlay GW/ESI
  1.1.1.111:1            1100       02:05:86:71:94:00  1.1.1.11
  1.1.1.112:1            1100       02:05:86:71:62:00  1.1.1.12

{master:0}
root@spine3> show evpn ip-prefix-database extensive 
L3 context: T5-VRF1

IPv4->EVPN Exported Prefixes

Prefix: 100.100.100.103/32
  EVPN route status: Created
  Change flags: 0x0
  Advertisement mode: Direct nexthop
  Encapsulation: VXLAN
  VNI: 1100
  Router MAC: 02:05:86:71:db:00

Prefix: 150.250.1.0/24
  EVPN route status: Created
  Change flags: 0x0
  Advertisement mode: Direct nexthop
  Encapsulation: VXLAN
  VNI: 1100
  Router MAC: 02:05:86:71:db:00

EVPN->IPv4 Imported Prefixes

Prefix: 100.100.100.100/32, Ethernet tag: 0
  Change flags: 0x0
  Remote advertisements:
    Route Distinguisher: 1.1.1.111:1
      VNI: 1100
      Router MAC: 02:05:86:71:94:00
      BGP nexthop address: 1.1.1.11
      IP route status: Created

Prefix: 100.100.100.101/32, Ethernet tag: 0
  Change flags: 0x0
  Remote advertisements:
    Route Distinguisher: 1.1.1.112:1
      VNI: 1100
      Router MAC: 02:05:86:71:62:00
      BGP nexthop address: 1.1.1.12
      IP route status: Created
<omitted>

root@spine3> show route table T5-VRF1.evpn.0 

T5-VRF1.evpn.0: 8 destinations, 14 routes (8 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

5:1.1.1.111:1::0::150.100.1.0::24/304               
                   *[BGP/170] 01:41:52, localpref 100, from 1.1.1.11
                      AS path: I, validation-state: unverified
                    > to 10.10.5.1 via xe-0/0/2.0
                    [BGP/170] 01:41:52, localpref 100, from 1.1.1.12
                      AS path: I, validation-state: unverified
                    > to 10.10.5.1 via xe-0/0/2.0
5:1.1.1.111:1::0::150.101.1.0::24/304               
                   *[BGP/170] 01:41:52, localpref 100, from 1.1.1.11
                      AS path: I, validation-state: unverified
                    > to 10.10.5.1 via xe-0/0/2.0
                    [BGP/170] 01:41:52, localpref 100, from 1.1.1.12
                      AS path: I, validation-state: unverified
                    > to 10.10.5.1 via xe-0/0/2.0
5:1.1.1.111:1::0::100.100.100.100::32/304               
                   *[BGP/170] 01:41:52, localpref 100, from 1.1.1.11
                      AS path: I, validation-state: unverified
                    > to 10.10.5.1 via xe-0/0/2.0
                    [BGP/170] 01:41:52, localpref 100, from 1.1.1.12
                      AS path: I, validation-state: unverified
                    > to 10.10.5.1 via xe-0/0/2.0
5:1.1.1.112:1::0::150.100.1.0::24/304               
                   *[BGP/170] 01:41:52, localpref 100, from 1.1.1.12
                      AS path: I, validation-state: unverified
                    > to 10.10.6.1 via xe-0/0/0.0
                    [BGP/170] 01:41:52, localpref 100, from 1.1.1.11
                      AS path: I, validation-state: unverified
                    > to 10.10.6.1 via xe-0/0/0.0
5:1.1.1.112:1::0::150.101.1.0::24/304               
                   *[BGP/170] 01:41:52, localpref 100, from 1.1.1.12
                      AS path: I, validation-state: unverified
                    > to 10.10.6.1 via xe-0/0/0.0
                    [BGP/170] 01:41:52, localpref 100, from 1.1.1.11
                      AS path: I, validation-state: unverified
                    > to 10.10.6.1 via xe-0/0/0.0
5:1.1.1.112:1::0::100.100.100.101::32/304               
                   *[BGP/170] 01:41:52, localpref 100, from 1.1.1.12
                      AS path: I, validation-state: unverified
                    > to 10.10.6.1 via xe-0/0/0.0
                    [BGP/170] 01:41:52, localpref 100, from 1.1.1.11
                      AS path: I, validation-state: unverified
                    > to 10.10.6.1 via xe-0/0/0.0
5:1.1.1.113:1::0::150.250.1.0::24/304               
                   *[EVPN/170] 01:39:26
                      Indirect
5:1.1.1.113:1::0::100.100.100.103::32/304               
                   *[EVPN/170] 04:40:25
                      Indirect

{master:0}
root@spine3> show route table T5-VRF1.inet.0 

T5-VRF1.inet.0: 8 destinations, 12 routes (8 active, 0 holddown, 0 hidden)
@ = Routing Use Only, # = Forwarding Use Only
+ = Active Route, - = Last Active, * = Both

1.1.1.113/32       *[Direct/0] 04:40:40
                    > via lo0.1
100.100.100.100/32 *[EVPN/170] 01:42:08
                    > to 10.10.5.1 via xe-0/0/2.0
100.100.100.101/32 *[EVPN/170] 01:42:08
                    > to 10.10.6.1 via xe-0/0/0.0
100.100.100.103/32 *[Static/5] 04:40:42
                      Discard
150.100.1.0/24     @[EVPN/170] 01:42:08
                    > to 10.10.5.1 via xe-0/0/2.0
                    [EVPN/170] 01:42:08
                    > to 10.10.6.1 via xe-0/0/0.0
                   #[Multipath/255] 01:42:08, metric2 0
                    > to 10.10.5.1 via xe-0/0/2.0
                      to 10.10.6.1 via xe-0/0/0.0
150.101.1.0/24     @[EVPN/170] 01:42:08
                    > to 10.10.5.1 via xe-0/0/2.0
                    [EVPN/170] 01:42:08
                    > to 10.10.6.1 via xe-0/0/0.0
                   #[Multipath/255] 01:42:08, metric2 0
                    > to 10.10.5.1 via xe-0/0/2.0
                      to 10.10.6.1 via xe-0/0/0.0
150.250.1.0/24     *[Direct/0] 02:48:00
                    > via irb.250
150.250.1.1/32     *[Local/0] 03:53:26
                      Local via irb.250

{master:0}
root@spine3> 

root@ce2# set routing-instances TEST routing-options static route 0.0.0.0/0 next-hop 150.100.1.254 

{master:0}[edit]
root@ce2# commit 
configuration check succeeds
commit complete

{master:0}[edit]
root@ce2# 

{master:0}
root@ce2> 

{master:0}
root@ce2> ping 150.250.1.100 routing-instance TEST 
PING 150.250.1.100 (150.250.1.100): 56 data bytes
64 bytes from 150.250.1.100: icmp_seq=0 ttl=62 time=806.318 ms
64 bytes from 150.250.1.100: icmp_seq=1 ttl=62 time=762.705 ms
64 bytes from 150.250.1.100: icmp_seq=2 ttl=62 time=1234.241 ms
64 bytes from 150.250.1.100: icmp_seq=3 ttl=62 time=1372.196 ms
64 bytes from 150.250.1.100: icmp_seq=4 ttl=62 time=1076.550 ms
^C
--- 150.250.1.100 ping statistics ---
6 packets transmitted, 5 packets received, 16% packet loss
round-trip min/avg/max/stddev = 762.705/1050.402/1372.196/236.803 ms

{master:0}
root@ce2> 
root@ce3> ping 150.100.1.101    
PING 150.100.1.101 (150.100.1.101): 56 data bytes
64 bytes from 150.100.1.101: icmp_seq=0 ttl=62 time=1061.817 ms
64 bytes from 150.100.1.101: icmp_seq=1 ttl=62 time=1039.036 ms
^C
--- 150.100.1.101 ping statistics ---
4 packets transmitted, 2 packets received, 50% packet loss
round-trip min/avg/max/stddev = 1039.036/1050.427/1061.817/11.390 ms

{master:0}
root@ce3> 

```

