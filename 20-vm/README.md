# VM-based nodes in containerlab

VM nodes integration in containerlab is based on the [hellt/vrnetlab](https://github.com/hellt/vrnetlab) project which is a **fork** of `vrnetlab/vrnetlab` where things were added to make it work with the container networking.  
For more detail refer to the [CLAB VM-based routers integration](https://containerlab.dev/manual/vrnetlab/).

Start with cloning the project:

```bash
cd ~
git clone https://github.com/hellt/vrnetlab.git
cd ~/vrnetlab
```

## Building Nokia SR OS image

SR OS qcow image is located at `~/images/24.10.R6/sros-vsim.qcow2` and should be copied to the `~/vrnetlab/nokia/sros/` directory before building the container image.

```bash
cp ~/images/24.10.R6/sros-vsim.qcow2 ~/vrnetlab/nokia/sros/sros-vm-24.10.R6.qcow2
```

Once copied, we can enter in the `~/vrnetlab/nokia/sros` image and build the container image:

```bash
cd ~/vrnetlab/nokia/sros
make
```

The image will be built and tagged as `vrnetlab/nokia_sros:24.10.R6`. The tag is auto-derived from the file name.

Check what images are available on the system:

```bash
docker images
```

Output should look similar to this (IMAGE ID will be different)
```bash
[*]─[vm4]─[~/vrnetlab/nokia/sros]
└──> docker images
REPOSITORY                           TAG        IMAGE ID       CREATED          SIZE
vrnetlab/nokia_sros                  24.10.R6   684bb05aa297   37 minutes ago   1.01GB
ghcr.io/nokia/srlinux                latest     ae21f72bdbc7   3 weeks ago      2.27GB
nokia_srsim                          25.7.R2    da84681892a3   4 weeks ago      1.97GB
```
## Deploying the VM-based nodes lab

With the images built, we can proceed with the lab deployment. First, let's switch back to the lab directory:

```bash
cd ~/containerlab-workshop-ch/20-vm
```

Now lets deploy the lab:

```bash
clab dep -c 
```

### Monitoring the boot process

You will notice that the lab deployment log will show the following message and the deployment process will appear stuck:

```
Waiting for clab-vm-vsim to be ready. This may take a while. In another terminal monitor boot log with `docker logs -f clab-vm-vsim`
```

This is because containerlab waits for vSIM SR OS node to boot and become ready for containerlab to inject SSH keys to enabled passwordless SSH access. To monitor the boot process, you can open a new terminal and run the following command:

```bash
docker logs -f clab-vm-vsim
```

> the SR OS vSIM boot time is approx 3 minutes.

You will see the boot process of the SR OS vSIM node as it would have been seen in a terminal connected to the real hardware console.

Check the running labs and ensure that the vSIM is running

```
[x]─[vm4]─[~/containerlab-workshop-ch/20-vm]
└──> clab ins -a
╭─────────────┬──────────┬───────────────┬──────────────────────────────┬───────────┬───────────────────╮
│   Topology  │ Lab Name │      Name     │          Kind/Image          │   State   │   IPv4/6 Address  │
├─────────────┼──────────┼───────────────┼──────────────────────────────┼───────────┼───────────────────┤
│ vm.clab.yml │ vm       │ clab-vm-srsim │ nokia_srsim                  │ running   │ 172.20.20.3       │
│             │          │               │ nokia_srsim:25.7.R1          │           │ 3fff:172:20:20::3 │
│             │          ├───────────────┼──────────────────────────────┼───────────┼───────────────────┤
│             │          │ clab-vm-vsim  │ nokia_sros                   │ running   │ 172.20.20.2       │
│             │          │               │ vrnetlab/nokia_sros:24.10.R6 │ (healthy) │ 3fff:172:20:20::2 │
╰─────────────┴──────────┴───────────────┴──────────────────────────────┴───────────┴───────────────────╯
```

## Connecting to the nodes

To connect to vSIM node:

```bash
ssh clab-vm-vsim
```

No user/password is needed, since username is injected by containerlab as well as the local public key for passwordless SSH access.

## Configuring the nodes

### Nokia vSIM

If not logged in yet, open a new terminal and connect to the vSIM node:

```bash
ssh clab-vm-vsim
```

While logged in in the vSIM CLI paste the following snippet to configure vSIM:

```
edit-config private
/configure port 1/1/c1 admin-state enable
/configure port 1/1/c1 connector breakout c1-100g
/configure port 1/1/c1/1 admin-state enable
/configure port 1/1/c1/1 description "port 1/1/c1/1"
/configure port 1/1/c1/1 ethernet mode hybrid
/configure port 1/1/c1/1 ethernet lldp 
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge notification true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge port-id-subtype tx-if-name
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge receive true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge transmit true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs port-desc true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs sys-name true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs sys-desc true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs sys-cap true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-mgmt-address oob admin-state enable
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-mgmt-address system admin-state enable

/configure router "Base" interface "toSRSIM" admin-state enable
/configure router "Base" interface "toSRSIM" port 1/1/c1/1:0
/configure router "Base" interface "toSRSIM" ipv4 primary address 192.168.1.2
/configure router "Base" interface "toSRSIM" ipv4 primary prefix-length 24
commit
exit all
```

### Nokia SR-SIM

If not logged in yet, open a new terminal and connect to the SRSIM SR OS node:

```bash
ssh clab-vm-srsim
```

And paste the following configuration:

```
edit-config private
/configure card 1 card-type iom-1
/configure card 1 mda 1 mda-type me6-100gb-qsfp28
/configure card 1 mda 2 mda-type me12-100gb-qsfp28

/configure port 1/1/c1 admin-state enable
/configure port 1/1/c1 connector breakout c1-100g
/configure port 1/1/c1/1 admin-state enable
/configure port 1/1/c1/1 description "port 1/1/c1/1"
/configure port 1/1/c1/1 ethernet mode hybrid
/configure port 1/1/c1/1 ethernet lldp 
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge notification true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge port-id-subtype tx-if-name
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge receive true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge transmit true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs port-desc true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs sys-name true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs sys-desc true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-tlvs sys-cap true
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-mgmt-address oob admin-state enable
/configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-mgmt-address system admin-state enable

/configure router "Base" interface "toVSIM" admin-state enable
/configure router "Base" interface "toVSIM" port 1/1/c1/1:0
/configure router "Base" interface "toVSIM" ipv4 primary address 192.168.1.1
/configure router "Base" interface "toVSIM" ipv4 primary prefix-length 24
commit
exit all
```

Now we configured the two systems to be able to communicate with each other and enabled LLDP on both sides. Perform a ping from SRSIM towards the vSIM node and let it run:

### Verify the connectivity

While in vSIM CLI, let's check a few things. First, let's see if LLDP is working and we detect the SRSIM node:

```
(pr)[/]
A:admin@vsim# show port 1/1/c1/1 ethernet lldp remote-info

==============================================================================
Link Layer Discovery Protocol (LLDP) Port Information
==============================================================================
Port 1/1/c1/1 Bridge nearest-bridge Remote Peer Information
-------------------------------------------------------------------------------
Remote Peer Index 1 at timestamp 10/05/2025 10:41:20:
Supported Caps        : bridge router
Enabled Caps          : bridge router
Chassis Id Subtype    : 4 (macAddress)
Chassis Id            : 1C:D0:00:00:00:00
PortId Subtype        : 5 (interfaceName)
Port Id               : 31:2F:31:2F:63:31:2F:31
                        "1/1/c1/1"
Port Description      : 1/1/c1/1, 100-Gig Ethernet, "port 1/1/c1/1"
System Name           : srsim
System Description    : TiMOS-B-25.7.R1 both/x86_64 Nokia 7750 SR Copyright
                        (c) 2000-2025 Nokia.
                        All rights reserved. All use subject to applicable
                        license agreements.
                        Built on Wed Jul 16 21:13:09 UTC 2025 by builder in /
                        builds/257B/R1/panos/main/srux
Age                   : 518 seconds


Port 1/1/c1/1 Bridge nearest-non-tpmr Remote Peer Information
-------------------------------------------------------------------------------
No remote peers found

Port 1/1/c1/1 Bridge nearest-customer Remote Peer Information
-------------------------------------------------------------------------------
No remote peers found

==============================================================================
```

Great, the neighboring router is detected, which means that L2 protocols such as LLDP are working fine.

Next, check the configuration status of the interface towards the c8000v node:

```
(pr)[/]
A:admin@vsim# / show router "Base" interface "toSRSIM"

===============================================================================
Interface Table (Router: Base)
===============================================================================
Interface-Name                   Adm       Opr(v4/v6)  Mode    Port/SapId
   IP-Address                                                  PfxState
-------------------------------------------------------------------------------
toSRSIM                          Up        Up/Down     Network 1/1/c1/1:0
   192.168.1.2/24                                              n/a
-------------------------------------------------------------------------------
Interfaces : 1
===============================================================================
```

The interface is up and the remote IP address is 192.168.1.2.

On the SRSIM let's start a ping to the vSIM to verify the datapath and make it run for a while with thousands of probes requested:

```bash
ssh clab-vm-srsim
```

```
[/]
A:admin@srsim# ping 192.168.1.2 count 2000
PING 192.168.1.2 56 data bytes
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=44.8ms.
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=1.46ms.
64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=1.65ms.
64 bytes from 192.168.1.2: icmp_seq=4 ttl=64 time=1.94ms.
```

Pings are flowing, neat. Let them run in this terminal window, we will need them later.

## Exploring the VM networking setup

In a new terminal window open vSIM node's container shell:

```bash
docker exec -it clab-vm-vsim bash
```

Explore the Linux link interfaces available in the container:

```bash
ip link
```

the `tap1` interface corresponds to the VM's interfaces and it is transparently "stitched" (with `tc mirred` rules)

You can sniff the traffic on both `eth1` and `tap1` interface, and see that they carry the same traffic, as they are merely two ends of the same virtual link.

```bash
tcpdump -nni eth1
```

Output eth1:
```
root@vsim:/# tcpdump -nni eth1
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
10:53:29.969801 IP 192.168.1.1 > 192.168.1.2: ICMP echo request, id 250, seq 32779, length 64
10:53:29.970888 IP 192.168.1.2 > 192.168.1.1: ICMP echo reply, id 250, seq 32779, length 64
10:53:30.975563 IP 192.168.1.1 > 192.168.1.2: ICMP echo request, id 250, seq 32780, length 64
10:53:30.977135 IP 192.168.1.2 > 192.168.1.1: ICMP echo reply, id 250, seq 32780, length 64
10:53:31.971096 IP 192.168.1.1 > 192.168.1.2: ICMP echo request, id 250, seq 32781, length 64
```

```bash
tcpdump -nni tap1
```
Output tap1:
```
root@vsim:/# tcpdump -nni tap1
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tap1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
11:34:43.157413 IP 192.168.1.1 > 192.168.1.2: ICMP echo request, id 250, seq 32772, length 64
11:34:43.158636 IP 192.168.1.2 > 192.168.1.1: ICMP echo reply, id 250, seq 32772, length 64
11:34:44.159706 IP 192.168.1.1 > 192.168.1.2: ICMP echo request, id 250, seq 32773, length 64
11:34:44.160917 IP 192.168.1.2 > 192.168.1.1: ICMP echo reply, id 250, seq 32773, length 64
11:34:45.162010 IP 192.168.1.1 > 192.168.1.2: ICMP echo request, id 250, seq 32774, length 64
11:34:45.163327 IP 192.168.1.2 > 192.168.1.1: ICMP echo reply, id 250, seq 32774, length 64
```

From vSIM node's container shell you may access to the vSIM console using telnet to localhost port 5000. This may be useful for troubleshooting.  
Test login to the vSIM with:
```
telnet localhost 5000
```

