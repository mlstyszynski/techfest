# EVPN-VXLAN lab Section 1 
## Centrally Routed Bridging Overlay architecture

`Lab section objectives`

The goal of the section 1 is to build the Centrally Routed Bridging Overlay architecture using the Juniper QFX series EVPN-VXLAN technologies. 

The iBGP EVPN-type2 routes will be used in order to advertise the MAC@ and MAC+IP between the leafs of the same fabric. 
 
The inter-vni routing will be taking place at the spine1-re and spine2-re therefore the MAC+IP routes will be injected by the spine1-re/Spine2-re on behalf of the layer 2 leafs. 
 
CE1 (VNI 100) and CE2 (VNI 100) will have to communicate . The baseline environment provided has EBGP set up for the Spine-Spine and Spine-Leaf underlay connections.
 
In Section 1, you will configure MP-iBGP as the overlay between the leaf devices. Section 2 will require you to establish EVPN type 2 (Asymmetric mode) communication between VNI 10 and VNI 30. In Section 3,  you will establish EVPN type 5 (Symmetric mode) communication between VNI 20 and VNI 40. Finally, in the same section, type 5 will be introduced on the remaining two leaf devices to facilitate full inter-VXLAN communication. 


`Environment`

Each team of 2 will be provided with a dedicated environment composed of the following
- 3 x vQFX Spines
- 4 x vQFX Leafs
- 3 x vQFX CEs
Work with your partner to divide and concur the tasks. 


> All VMs are accessible from internet, so you can run everything from your laptop or from one of the ubuntu server provided




### Physical connections - lab section 1
![Lab topology-1](topologies/evpn-vxlan-techfest_topo1.png)

# Guide for EVPN/VXLAN hands on lab


Confirm connectivity to the leaf lo0 addresses of all leaf devices. These are exchanged via the underlay eBGP session and will be required to setup the overlay iBGP session between leaf devices

#### Leaf - 1

```
{master:0}
root@leaf1> ping 150.100.1.101

root@leaf1> ping 150.250.1.100

root@leaf1> ping 150.251.1.100

```
## Part 1: Steps for setting up overlay on Leaf 1, Leaf 2, Leaf 3 and Leaf 4

#### Leaf-1 config

###### Setup MP-iBGP sessions with other Leaf devices

```
{master:0}[edit]
root@leaf1# show protocols bgp group overlay
type internal;
local-address 1.1.1.1;
family evpn {
    signaling;
}
local-as 64512;
neighbor 1.1.1.11;
neighbor 1.1.1.12;

{master:0}[edit]
root@leaf1#
```

###### Leaf-2

```
{master:0}[edit]
root@LEAF-2# show protocols bgp group overlay
type internal;
local-address 2.2.2.2;
family evpn {
    signaling;
}
local-as 65100;
neighbor 1.1.1.1;
neighbor 4.4.4.4;
neighbor 3.3.3.3;

{master:0}[edit]
root@LEAF-2#
```

###### Leaf-3

```
{master:0}[edit]
root@LEAF-3# show protocols bgp group overlay
type internal;
local-address 3.3.3.3;
family evpn {
    signaling;
}
local-as 65100;
neighbor 1.1.1.1;
neighbor 2.2.2.2;
neighbor 4.4.4.4;

{master:0}[edit]
root@LEAF-3#
```

###### Leaf-4

```
{master:0}[edit]
root@LEAF-4# show protocols bgp group overlay
type internal;
local-address 4.4.4.4;
family evpn {
    signaling;
}
local-as 65100;
neighbor 1.1.1.1;
neighbor 2.2.2.2;
neighbor 3.3.3.3;

{master:0}[edit]
root@LEAF-4#
```
### Verification

Verify BGP connections between leaf devices and make sure they are established and evpn tables are reflected under the sessions

#### Leaf - 1
```sh
root@LEAF-1> show bgp summary    
Groups: 2 Peers: 4 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
bgp.evpn.0           
                      16         16          0          0          0          0
inet.0               
                      12          9          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
2.2.2.2               65100        621        630       0       0     4:40:52 Establ
  bgp.evpn.0: 2/2/2/0
  default-switch.evpn.0: 1/1/1/0
  __default_evpn__.evpn.0: 0/0/0/0
3.3.3.3               65100        625        624       0       0     4:38:27 Establ
  bgp.evpn.0: 12/12/12/0
  default-switch.evpn.0: 12/12/12/0
  __default_evpn__.evpn.0: 0/0/0/0
4.4.4.4               65100        616        623       0       0     4:38:18 Establ
  bgp.evpn.0: 2/2/2/0
  default-switch.evpn.0: 1/1/1/0
  __default_evpn__.evpn.0: 0/0/0/0
192.168.10.2          65005        625        621       0       0     4:40:53 Establ
  inet.0: 9/12/12/0

```

## Part-2: Inter-VNI using Type 2 routes ( Asymmetric mode)
In this section we will configure  Leaf 1 and Leaf 3

Goal is to establish connectivity between Server 1 and Server 3 which are part of VNI 10 and VNI 30 respectively. Server 1 is only connected to Leaf 1 and Server 3 is only connected to Leaf 3. We need to configure both VNI 10 and VNI 30 on both Leaf 1 and Leaf 3

#### Leaf - 1

##### Step 1: Configure bridge domains bd10 and bd30 

```
{master:0}[edit]
root@LEAF-1# show vlans
bd10 {
    vlan-id 10;
    l3-interface irb.10;
    vxlan {
        vni 10;
    }
}
bd30 {
    vlan-id 30;
    l3-interface irb.30;
    vxlan {
        vni 30;
    }
}
```
Here we are creating bridge domain 10 and 30 and assigning them to VNI 10 and VNI 30 respectively. We are also assigning the IRB interface for routing.

##### Step 2: Configure IRB interface

```
{master:0}[edit]
root@LEAF-1# show interfaces irb
unit 10 {
    family inet {
        address 10.10.10.1/24 {
            virtual-gateway-address 10.10.10.100;
        }
    }
}
unit 30 {
    family inet {
        address 30.30.30.1/24 {
            virtual-gateway-address 30.30.30.100;
        }
    }
}
```

We are using the virtual-gateway-address model here. This address will serve as the default gateway for the servers. The virtual-gateway-address will be same on Leaf - 3 as well for both irb 10 and irb 30

##### Step 3: Assign server facing interface to the bridge domain

```

{master:0}[edit]
root@LEAF-1# show interfaces ae1
aggregated-ether-options {
    lacp {
        active;
        periodic fast;
        system-id 01:01:01:01:01:01;
    }
}
unit 0 {
    family ethernet-switching {
        interface-mode trunk;
        vlan {
            members all;
        }
    }
}

```

##### Step 4: Configure protocol evpn

```
root@LEAF-1# show protocols evpn
encapsulation vxlan;
extended-vni-list [ 10 30 ];
vni-options {
    vni 10 {
        vrf-target export target:1:10;
    }
    vni 30 {
        vrf-target export target:1:30;
    }
}
```

Here you specify what VNI the PE is interested in using extended-vni-list command. We also assign Route target to be exported for vni 10 and vni 30 for Type 2 and Type 3 routes. We also specify the dataplane encapsulation type as vxlan here.


##### Step 5: Configure switch-options

```
master:0}[edit]
root@LEAF-1# show switch-options
vtep-source-interface lo0.0;
route-distinguisher 1.1.1.1:100;
vrf-import EVPN-IMPORT;
vrf-target target:1:100;
```

Here we specify the local vtep endpoint as lo0.
Route distinguisher is specified to make local PE routes unique in case of overlapping addresses.
We also define the import ( EVPN-IMPORT) for Type1, Type2 and Type3 routes and export policy  for the Type 1 routes


##### Step 6: Configure Import policy for Type 1, Type 2 and Type 3 routes

```
{master:0}[edit]
root@LEAF-1# show policy-options
policy-statement EVPN-IMPORT {
    term ESI {
        from community esi;
        then accept;
    }
    term vni10 {
        from community vni10;
        then accept;
    }
    term vni30 {
        from community vni30;
        then accept;
    }
}

community esi members target:1:100;
community vni10 members target:1:10;
community vni30 members target:1:30;
```

##### Step 7: Configure routing instance
```
root@LEAF-1# show routing-instances VRF-1
instance-type vrf;
interface irb.10;
interface irb.30;
interface lo0.10;
route-distinguisher 1.1.1.10:10;
vrf-target target:10:10;
```
Here we create IP VRF and put the IRB interfaces in. The RD and RT configured here are for IP VRF and are different from the MAC VRF.

Please make sure you assign lo0.10 in the VRF

##### Step 8: Configure lo0.10 
```

{master:0}[edit]
root@LEAF-1# show interfaces lo0
unit 0 {
    family inet {
        address 1.1.1.1/32;
    }
}
unit 10 {
    family inet {
        address 1.1.1.10/32;
    }
}

```

Now we move to Leaf - 3

#### Leaf - 3

##### Step 1: Configure bridge domains bd10 and bd30 

```
{master:0}[edit]
root@LEAF-3# show vlans
bd10 {
    vlan-id 10;
    l3-interface irb.10;
    vxlan {
        vni 10;
    }
}
bd30 {
    vlan-id 30;
    l3-interface irb.30;
    vxlan {
        vni 30;
    }
}
```
Here we are creating bridge domain 10 and 30 and assigning them to VNI 10 and VNI 30 respectively. We are also assigning the IRB interface for routing.

##### Step 2: Configure IRB interface

```
{master:0}[edit]
root@LEAF-3# show interfaces irb
unit 10 {
    family inet {
        address 10.10.10.3/24 {
            virtual-gateway-address 10.10.10.100;
        }
    }
}
unit 30 {
    family inet {
        address 30.30.30.3/24 {
            virtual-gateway-address 30.30.30.100;
        }
    }
}
```

We are using the virtual-gateway-address model here. This address will serve as the default gateway for the servers. The virtual-gateway-address will be same on Leaf - 3 as well for both irb 10 and irb 30

##### Step 3: Assign server facing interface to the bridge domain

```

{master:0}[edit]
root@LEAF-3# show interfaces ae1
aggregated-ether-options {
    lacp {
        active;
        periodic fast;
        system-id 03:03:03:03:03:03;
    }
}
unit 0 {
    family ethernet-switching {
        interface-mode trunk;
        vlan {
            members all;
        }
    }
}
```

##### Step 4: Configure protocol evpn

```
root@LEAF-3# show protocols evpn
encapsulation vxlan;
extended-vni-list [ 10 30 ];
vni-options {
    vni 10 {
        vrf-target export target:1:10;
    }
    vni 30 {
        vrf-target export target:1:30;
    }
}
```

Here you specify what VNI the PE is interested in using extended-vni-list command. We also assign Route target to be exported for vni 10 and vni 30 for Type 2 and Type 3 routes. We also specify the dataplane encapsulation type as vxlan here.


##### Step 5: Configure switch-options

```
{master:0}[edit]
root@LEAF-3# show switch-options
vtep-source-interface lo0.0;
route-distinguisher 3.3.3.3:100;
vrf-import EVPN-IMPORT;
vrf-target target:1:100;
```

Here we specify the local vtep endpoint as lo0.
Route distinguisher is specified to make local PE routes unique in case of overlapping addresses.
We also define the import ( EVPN-IMPORT) for Type1, Type2 and Type3 routes and export policy  for the Type 1 routes


##### Step 6: Configure Import policy for Type 1, Type 2 and Type 3 routes

```
{master:0}[edit]
root@LEAF-3# show policy-options
policy-statement EVPN-IMPORT {
    term ESI {
        from community esi;
        then accept;
    }
    term vni10 {
        from community vni10;
        then accept;
    }
    term vni30 {
        from community vni30;
        then accept;
    }
}

community esi members target:1:100;
community vni10 members target:1:10;
community vni30 members target:1:30;
```

##### Step 7: Configure routing instance
```
root@LEAF-3# show routing-instances VRF-1
instance-type vrf;
interface irb.10;
interface irb.30;
interface lo0.10;
route-distinguisher 3.3.3.10:10;
vrf-target target:10:10;
```
Here we create IP VRF and put the IRB interfaces in. The RD and RT configured here are for IP VRF and are different from the MAC VRF.

Please make sure you assign lo0.10 in the VRF

##### Step 8: Configure lo0.10 
```

{master:0}[edit]
root@LEAF-3# show interfaces lo0
unit 0 {
    family inet {
        address 3.3.3.3/32;
    }
}
unit 10 {
    family inet {
        address 3.3.3.10/32;
    }
}

```

#### Verify connectivity between Server 1 and Server 3

Console into server 1 and server 3

Login: ravello/ravelloCloud

##### Ping between Server 1 and Server 3

From Server 1:

>Ping 30.30.30.30

From Server 3:

>Ping 10.10.10.10

###### Note
```
If the pings do not work to remote server, try the ping from the remote side as well
```

Although Server 1 is just connected to Leaf 1 and Server 3 is just connected to Leaf 3, we have configured both VNI 10 and VNI 30 on both leaves. This demonstrates the type 2 asymmetric routing case that we discussed earlier.

Server 1 is configured with irb.10 virtual gateway address as its gateway and Server 3 is configured with irb.30 virtual gateway address as its gateway

Lets check for the forwarding state on leaf 1 and leaf 2


##### Leaf 1:
Lets find the type 2 route for 30.30.30.30
```sh
{master:0}
root@LEAF-1> show route table default-switch.evpn.0 extensive | find 30.30.30.30
2:3.3.3.3:100::30::2c:c2:60:52:80:a8::30.30.30.30/304 (1 entry, 1 announced)
        *BGP    Preference: 170/-101
                Route Distinguisher: 3.3.3.3:100
                Next hop type: Indirect, Next hop index: 0
                Address: 0x9db6970
                Next-hop reference count: 28
                Source: 3.3.3.3
                Protocol next hop: 3.3.3.3
                Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                State: <Secondary Active Int Ext>
                Local AS: 65001 Peer AS: 65100
                Age: 19:34      Metric2: 0 
                Validation State: unverified 
                Task: BGP_65100_65100.3.3.3.3
                Announcement bits (1): 0-default-switch-evpn 
                AS path: I
                Communities: target:1:30 encapsulation0:0:0:0:vxlan
                Import Accepted
                Route Label: 30
                ESI: 00:00:00:00:00:00:00:00:00:00
                Localpref: 100
                Router ID: 3.3.3.3
                Primary Routing Table bgp.evpn.0
                Indirect next hops: 1
                        Protocol next hop: 3.3.3.3
                        Indirect next hop: 0x2 no-forward INH Session ID: 0x0
                        Indirect path forwarding next hops: 1
                                Next hop type: Router
                                Next hop: 192.168.10.2 via xe-0/0/0.0
                                Session Id: 0x0
                        3.3.3.3/32 Originating RIB: inet.0
                          Node path count: 1
                          Forwarding nexthops: 1
                                Nexthop: 192.168.10.2 via xe-0/0/0.0


root@LEAF-1> show arp no-resolve | match 30.30.30.30 
2c:c2:60:52:80:a8 30.30.30.30     irb.30 [vtep.32770]      permanent remote


root@LEAF-1> show interfaces vtep.32770 
  Logical interface vtep.32770 (Index 572) (SNMP ifIndex 543)
    Flags: Up SNMP-Traps Encapsulation: ENET2
    VXLAN Endpoint Type: Remote, VXLAN Endpoint Address: 3.3.3.3, L2 Routing Instance: default-switch, L3 Routing Instance: default
    Input packets : 24
    Output packets: 83
    Protocol eth-switch, MTU: Unlimited
      Flags: Trunk-Mode

root@LEAF-1> show evpn database mac-address 2c:c2:60:52:80:a8 extensive 
Instance: default-switch

VN Identifier: 30, MAC address: 2c:c2:60:52:80:a8
  Source: 3.3.3.3, Rank: 1, Status: Active
    Timestamp: May 14 23:11:06 (0x5918e40a)
    State: <Remote-To-Local-Adv-Done>
    IP address: 30.30.30.30
```

Follow the same steps on Leaf 3 for IP address 10.10.10.10


At this point you will not be able to ping 20.20.20.20 from Leaf 1, Leaf 3 and Leaf 4 as there is no VNI 20 present on these leaves. Hence there is not type 2 routes exchanged for 20.20.20.20

For the same reason you will not be able to ping 40.40.40.40 from Leaf 1, Leaf 2 and Leaf 3

In the next section we will enable Type 5 routes on all the leaves so that we can establish the reachability. Since this connectivity is established via Type 5 routes we do not need to configure VNI 20 and VNI 40 on remote leaves. This demonstrates symmetric routing use case. Remote leaves will use L3 VNI 5555


## Part-3: Inter-VNI using Type 5 routes ( Symmetric mode)

We will first configure Type 5 routes on Leaf 2 and Leaf 4 to show that they are communicating using Type 5 routes. No Type 2 routes are exchanged between Leaf 2 and Leaf 4 as they do not have each others VNI

#### Leaf - 2
##### Step 1: Configure bridge domains bd20 

```
{master:0}
root@LEAF-2> show configuration vlans
bd20 {
    vlan-id 20;
    l3-interface irb.20;
    vxlan {
        vni 20;
    }
}

```
##### Step 2: Configure IRB interface
```
root@LEAF-2> show configuration interfaces irb
unit 20 {
    family inet {
        address 20.20.20.100/24;
    }
}
```
##### Step 3: Assign server facing interface to the bridge domain

```
{master:0}
root@LEAF-2> show configuration interfaces xe-0/0/2
description "to VLAN20";
unit 0 {
    family ethernet-switching {
        interface-mode trunk;
        vlan {
            members all;
        }
    }
}
```

##### Step 4: Configure protocol evpn
```
root@LEAF-2> show configuration protocols evpn
encapsulation vxlan;
```

Since we are using Type 5 routes there is no need to configure anything else here.

##### Step 5: Configure switch-options

```
master:0}
root@LEAF-2> show configuration switch-options
vtep-source-interface lo0.0;
route-distinguisher 2.2.2.2:100;
vrf-target target:1:100;
```

We do not need the EVPN-IMPORT policy here since we are not using Type 2 routes here

##### Step 6: Configure routing instance

```

{master:0}
root@LEAF-2> show configuration routing-instances
VRF-1 {
    instance-type vrf;
    interface irb.20;
    interface lo0.10;
    route-distinguisher 2.2.2.10:10;
    vrf-target target:10:10;
    protocols {
        evpn {
            ip-prefix-routes {
                advertise direct-nexthop;
                encapsulation vxlan;
                vni 5555;
            }
        }
    }
}

```
The "protocol evpn" configuration under the VRF enables the Type 5 routes. We are using vni 5555 as the L3 vni.


##### Step 7: Configure lo0.10 

```
{master:0}
root@LEAF-2> show configuration interfaces lo0
unit 0 {
    family inet {
        address 2.2.2.2/32;
    }
}
unit 10 {
    family inet {
        address 2.2.2.10/32;
    }
}
```


#### Leaf - 4
##### Step 1: Configure bridge domains bd20 

```
{master:0}
root@LEAF-4> show configuration vlans
bd40 {
    vlan-id 40;
    l3-interface irb.40;
    vxlan {
        vni 40;
    }
}

```
##### Step 2: Configure IRB interface
```
{master:0}
root@LEAF-4> show configuration interfaces irb
unit 40 {
    family inet {
        address 40.40.40.100/24;
    }
}
```
##### Step 3: Assign server facing interface to the bridge domain

```
{master:0}
root@LEAF-4> show configuration interfaces xe-0/0/2
unit 0 {
    family ethernet-switching {
        interface-mode trunk;
        vlan {
            members all;
        }
    }
}
```

##### Step 4: Configure protocol evpn
```
root@LEAF-4> show configuration protocols evpn
encapsulation vxlan;
```

Since we are using Type 5 routes there is no need to configure anything else here.

##### Step 5: Configure switch-options

```
{master:0}
root@LEAF-4> show configuration switch-options
vtep-source-interface lo0.0;
route-distinguisher 4.4.4.4:100;
vrf-target target:1:100;
```

We do not need the EVPN-IMPORT policy here since we are not using Type 2 routes here

##### Step 6: Configure routing instance

```

{master:0}
root@LEAF-4> show configuration routing-instances
VRF-1 {
    instance-type vrf;
    interface irb.40;
    interface lo0.10;
    route-distinguisher 4.4.4.10:10;
    vrf-target target:10:10;
    protocols {
        evpn {
            ip-prefix-routes {
                advertise direct-nexthop;
                encapsulation vxlan;
                vni 5555;
            }
        }
    }
}

```
The "protocol evpn" configuration under the VRF enables the Type 5 routes. We are using vni 5555 as the L3 vni.


##### Step 7: Configure lo0.10 

```
{master:0}
root@LEAF-4> show configuration interfaces lo0
unit 0 {
    family inet {
        address 4.4.4.4/32;
    }
}
unit 10 {
    family inet {
    
        address 4.4.4.10/32;
    }
}
```



#### Verification

Console into Server 2 and Server 4

Login: ravello/ravelloCloud

##### Ping between Server 2 and Server 4

From Server 2:

>Ping 40.40.40.40

From Server 4:

>Ping 20.20.20.20

###### Note
```
If the pings do not work to remote server, try the ping from the remote side as well
```

At this point we cannot ping Server 2 and Server 4 from Server 1 and Server 3

We need to enable type 5 route on Leaf 1 and Leaf 3

#### Enable Type 5 on Leaf 1 and Leaf 3

##### Step 1: Configure routing instance
##### Leaf - 1

```
{master:0}
root@LEAF-1> show configuration routing-instances
VRF-1 {
    instance-type vrf;
    interface irb.10;
    interface irb.30;
    interface lo0.10;
    route-distinguisher 1.1.1.10:10;
    vrf-target target:10:10;
    protocols {
        evpn {
            ip-prefix-routes {
                advertise direct-nexthop;
                encapsulation vxlan;
                vni 5555;
            }
        }
    }
}
```

##### Leaf - 3
```
{master:0}
root@LEAF-3> show configuration routing-instances
VRF-1 {
    instance-type vrf;
    interface irb.10;
    interface irb.30;
    interface lo0.10;
    route-distinguisher 3.3.3.10:10;
    vrf-target target:10:10;
    protocols {
        evpn {
            ip-prefix-routes {
                advertise direct-nexthop;
                encapsulation vxlan;
                vni 5555;
            }
        }
    }
}
```

At this point you should be able to ping between all the servers.

Login to Server 1 and 
>ping 20.20.20.20
>ping 30.30.30.30
>ping 40.40.40.40

###### Note
```
If the pings do not work to remote server, try the ping from the remote side as well
```

#### Verification

```sh
{master:0}
root@LEAF-1> show evpn ip-prefix-database extensive prefix 20.20.20.0/24 
L3 context: VRF-1

EVPN->IPv4 Imported Prefixes

Prefix: 20.20.20.0/24, Ethernet tag: 0
  Change flags: 0x0
  Remote advertisements:
    Route Distinguisher: 2.2.2.10:10
      VNI: 5555
      Router MAC: 02:05:86:71:b6:00
      BGP nexthop address: 2.2.2.2
      IP route status: Created

{master:0}
root@LEAF-1> show evpn ip-prefix-database extensive prefix 40.40.40.0/24    
L3 context: VRF-1

EVPN->IPv4 Imported Prefixes

Prefix: 40.40.40.0/24, Ethernet tag: 0
  Change flags: 0x0
  Remote advertisements:
    Route Distinguisher: 4.4.4.10:10
      VNI: 5555
      Router MAC: 02:05:86:71:e7:00
      BGP nexthop address: 4.4.4.4
      IP route status: Created


{master:0}
root@LEAF-1> show route table VRF-1.inet.0 extensive 20.20.20.0    

VRF-1.inet.0: 8 destinations, 10 routes (8 active, 0 holddown, 0 hidden)
20.20.20.0/24 (1 entry, 1 announced)
TSI:
KRT in-kernel 20.20.20.0/24 -> {composite(1782)}
        *EVPN   Preference: 170
                Next hop type: Indirect, Next hop index: 0
                Address: 0x9db7ab0
                Next-hop reference count: 2
                Next hop type: Router, Next hop index: 1765
                Next hop: 192.168.10.2 via xe-0/0/0.0, selected
                Session Id: 0x0
                Protocol next hop: 2.2.2.2
                Composite next hop: 0xb4f8190 1782 INH Session ID: 0x0
                  VXLAN tunnel rewrite:
                    MTU: 0, Flags: 0x0
                    Encap table ID: 0, Decap table ID: 4
                    Encap VNI: 5555, Decap VNI: 5555
                    Source VTEP: 1.1.1.1, Destination VTEP: 2.2.2.2
                    SMAC: 02:05:86:71:43:00, DMAC: 02:05:86:71:b6:00
                Indirect next hop: 0x9e485c0 131070 INH Session ID: 0x0
                State: <Active Int Ext>
                Age: 16:41      Metric2: 0 
                Validation State: unverified 
                Task: VRF-1-EVPN-L3-context
                Announcement bits (1): 2-KRT 
                AS path: I
                Composite next hops: 1
                        Protocol next hop: 2.2.2.2
                        Composite next hop: 0xb4f8190 1782 INH Session ID: 0x0
                          VXLAN tunnel rewrite:
                            MTU: 0, Flags: 0x0
                            Encap table ID: 0, Decap table ID: 4
                            Encap VNI: 5555, Decap VNI: 5555
                            Source VTEP: 1.1.1.1, Destination VTEP: 2.2.2.2
                            SMAC: 02:05:86:71:43:00, DMAC: 02:05:86:71:b6:00
                        Indirect next hop: 0x9e485c0 131070 INH Session ID: 0x0
                        Indirect path forwarding next hops: 1
                                Next hop type: Router
                                Next hop: 192.168.10.2 via xe-0/0/0.0
                                Session Id: 0x0
                        2.2.2.2/32 Originating RIB: inet.0
                          Node path count: 1
                          Forwarding nexthops: 1
                                Nexthop: 192.168.10.2 via xe-0/0/0.0

{master:0}
root@LEAF-1> show route table VRF-1.inet.0 extensive 40.40.40.0    

VRF-1.inet.0: 8 destinations, 10 routes (8 active, 0 holddown, 0 hidden)
40.40.40.0/24 (1 entry, 1 announced)
TSI:
KRT in-kernel 40.40.40.0/24 -> {composite(1783)}
        *EVPN   Preference: 170
                Next hop type: Indirect, Next hop index: 0
                Address: 0x9db7d50
                Next-hop reference count: 2
                Next hop type: Router, Next hop index: 1765
                Next hop: 192.168.10.2 via xe-0/0/0.0, selected
                Session Id: 0x0
                Protocol next hop: 4.4.4.4
                Composite next hop: 0xb4f8258 1783 INH Session ID: 0x0
                  VXLAN tunnel rewrite:
                    MTU: 0, Flags: 0x0
                    Encap table ID: 0, Decap table ID: 4
                    Encap VNI: 5555, Decap VNI: 5555
                    Source VTEP: 1.1.1.1, Destination VTEP: 4.4.4.4
                    SMAC: 02:05:86:71:43:00, DMAC: 02:05:86:71:e7:00
                Indirect next hop: 0x9e48c00 131076 INH Session ID: 0x0
                State: <Active Int Ext>
                Age: 16:50      Metric2: 0 
                Validation State: unverified 
                Task: VRF-1-EVPN-L3-context
                Announcement bits (1): 2-KRT 
                AS path: I
                Composite next hops: 1
                        Protocol next hop: 4.4.4.4
                        Composite next hop: 0xb4f8258 1783 INH Session ID: 0x0
                          VXLAN tunnel rewrite:
                            MTU: 0, Flags: 0x0
                            Encap table ID: 0, Decap table ID: 4
                            Encap VNI: 5555, Decap VNI: 5555
                            Source VTEP: 1.1.1.1, Destination VTEP: 4.4.4.4
                            SMAC: 02:05:86:71:43:00, DMAC: 02:05:86:71:e7:00
                        Indirect next hop: 0x9e48c00 131076 INH Session ID: 0x0
                        Indirect path forwarding next hops: 1
                                Next hop type: Router
                                Next hop: 192.168.10.2 via xe-0/0/0.0
                                Session Id: 0x0
                        4.4.4.4/32 Originating RIB: inet.0
                          Node path count: 1
                          Forwarding nexthops: 1
                                Nexthop: 192.168.10.2 via xe-0/0/0.0

{master:0}
root@LEAF-1>

```
# Conclusion
In the above lab we demonstrated Inter-VNI traffic using both Type 2 and Type 5.

When using Type 2 mode, we are doing asymmetric routing. This needs all VNIs everywhere
Anytime a server is added to any leaf, we need to make sure that VNI is added on all leaf along with the import policies
When using Type 5 mode, we are doing symmetric routing and need only local VNIs. No need to add the VNI on leaves that do not have hosts in that VNI.

###### Note
```
As you must have noticed, we used only single homing and not multihoming using ESI. 
We started off with the multihoming case but vQFX did not work well with ESI. 
We see the control plane worked as expected but the servers could not ping each other in some cases
```



