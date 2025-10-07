# REPLACE OUTPUTS

# Containerlab Basics

This workshop section introduces you to containerlab basics - topology file, image management workflows and lab lifecycle. It is loosely based on the official [Containerlab quickstart](https://containerlab.dev/quickstart/).

## Repository

Clone the repository to your workshop VM at SUNRISEWS tag:

```bash
cd ~
git clone https://github.com/alinc01/clab-workshop_snr.git
cd clab-workshop_snr/10-basics
```

The repo should be cloned and you should be in the `clab-workshop_snr` directory as per the output below:

```
[*]─[vm4]─[~/clab-workshop_snr/10-basics]
└──>
```

## Topology

The topology file `basic.clab.yml` defines the lab we are going to use in this basics exercise. It consists of the two nodes:

* Nokia SR Linux
* Nokia SR-SIM

The nodes are interconnected with a single link over their respective first Ethernet interfaces.

```yaml
name: basic

topology:
  nodes:
    srl:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux

    srsim:
      kind: nokia_srsim
      image: srsim:25.7.R1
      license: /home/user/licenses/license_srsim.txt      
      startup-config: |
        /configure card 1 card-type iom-1
        /configure card 1 mda 1 mda-type me6-100gb-qsfp28
        /configure card 1 mda 2 mda-type me12-100gb-qsfp28
        /configure port 1/1/c1 connector breakout c1-100g
        /configure port 1/1/c1 admin-state enable
        /configure port 1/1/c1/1 ethernet mode hybrid
        /configure port 1/1/c1/1 admin-state enable
        /configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge port-id-subtype tx-if-alias
        /configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge receive true
        /configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge transmit true
        /configure port 1/1/c1/1 ethernet lldp dest-mac nearest-bridge tx-mgmt-address oob admin-state enable     
  links:
    - endpoints: [srl:e1-1, srsim:1/1/c1/1]
```

## Deployment attempt #1

Try to deploy the lab:

```bash
containerlab deploy -t basic.clab.yml
```

Deployment fails, why?

## Image management

Check what images are available on the system:

```bash
docker images
```

Pull SR Linux container image (if it has not been pulled during attempt #1):

```bash
docker pull ghcr.io/nokia/srlinux
```

Load the SR-SIM image:

```bash
docker load -i ~/images/25.7.R1/srsim.tar.xz
```

Check the local image store again:

```bash
docker images
```

Tag the image:

```bash
docker tag localhost/nokia/srsim:25.7.R1 nokia_srsim:25.7.R1
```

## Deployment attempt #2

Now that the images are available, try to deploy the lab again:

```bash
containerlab deploy -t basic.clab.yml
```

Note, you can use a shortcut version of the same command - `clab dep -t basic.clab.yml`.

The deployment should succeed.

## Connecting to the nodes

Connect to the Nokia SR Linux node using the container name:

```bash
ssh clab-basic-srl
```

To disconnect from the Nokia SR Linux node use:

```bash
quit
```

Connect to the SR-SIM node using its IP address (note, the IP might be different in your lab):

```bash
ssh clab-basic-srsim
```

To disconnect from the Nokia SR-OS node use:

```bash
logout
```

## Containerlab hosts automation

Containerlab creates `/etc/hosts` entries for each deployed lab so that you can access the nodes using their names. Check the entries:

```bash
cat /etc/hosts
```

## Containerlab ssh config automation

Containerlab creates ssh config entries in `/etc/ssh/ssh_config.d/clab-<lab-name>.conf` file to provide easy access to the nodes. Check the entries:

```bash
cat /etc/ssh/ssh_config.d/clab-basic.conf
```

## Checking network connectivity

SR Linux and SR-SIM are started with their first Ethernet interfaces connected. Check the connectivity between the nodes:

The nodes also come up with LLDP enabled, our goal is to verify that the basic network connectivity is working by inspecting

```bash
ssh clab-basic-srl
```

and checking the LLDP neighbors on ethernet-1/1 interface

```
show /system lldp neighbor interface ethernet-1/1
```

The expected output should be:

```
--{ running }--[  ]--
A:srl# show /system lldp neighbor interface ethernet-1/1
A:admin@srl# show /system lldp neighbor interface ethernet-1/1
  +--------------+-------------------+----------------------+---------------------+------------------------+----------------------+----------------------------+
  |     Name     |     Neighbor      | Neighbor System Name | Neighbor Chassis ID | Neighbor First Message | Neighbor Last Update |       Neighbor Port        |
  +==============+===================+======================+=====================+========================+======================+============================+
  | ethernet-1/1 | 1C:C2:01:00:00:00 |                      | 1C:C2:01:00:00:00   | 8 seconds ago          | 4 seconds ago        | 1/1/c1/1, 100-Gig Ethernet |
  +--------------+-------------------+----------------------+---------------------+------------------------+----------------------+----------------------------+
```

## Listing running labs

When you are in the directory that contains the lab file, you can list the nodes of that lab simply by running:

```bash
[*]─[vm4]─[~/clab-workshop_snr/10-basics]
└──> sudo containerlab inspect
15:05:06 INFO Parsing & checking topology file=basic.clab.yml
╭──────────────────┬───────────────────────┬─────────┬───────────────────╮
│       Name       │       Kind/Image      │  State  │   IPv4/6 Address  │
├──────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-basic-srl   │ nokia_srlinux         │ running │ 172.20.20.2       │
│                  │ ghcr.io/nokia/srlinux │         │ 3fff:172:20:20::2 │
├──────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-basic-srsim │ nokia_srsim           │ running │ 172.20.20.3       │
│                  │ nokia_srsim:25.7.R1   │         │ 3fff:172:20:20::3 │
╰──────────────────┴───────────────────────┴─────────┴───────────────────╯
```

If the topology file is located in a different directory, you can specify the path to the topology file:

```bash
[*]─[vm4]─[~/clab-workshop_snr/10-basics]
└──> containerlab inspect -t ~/clab-workshop_snr/10-basics/
15:05:17 INFO Parsing & checking topology file=basic.clab.yml
╭──────────────────┬───────────────────────┬─────────┬───────────────────╮
│       Name       │       Kind/Image      │  State  │   IPv4/6 Address  │
├──────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-basic-srl   │ nokia_srlinux         │ running │ 172.20.20.2       │
│                  │ ghcr.io/nokia/srlinux │         │ 3fff:172:20:20::2 │
├──────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-basic-srsim │ nokia_srsim           │ running │ 172.20.20.3       │
│                  │ nokia_srsim:25.7.R1   │         │ 3fff:172:20:20::3 │
╰──────────────────┴───────────────────────┴─────────┴───────────────────╯
```

You can also list all running labs regardless of where their topology files are located:

```bash
[*]─[vm4]─[~/clab-workshop_snr/10-basics]
└──> containerlab inspect --all
╭────────────────┬──────────┬──────────────────┬───────────────────────┬─────────┬───────────────────╮
│    Topology    │ Lab Name │       Name       │       Kind/Image      │  State  │   IPv4/6 Address  │
├────────────────┼──────────┼──────────────────┼───────────────────────┼─────────┼───────────────────┤
│ basic.clab.yml │ basic    │ clab-basic-srl   │ nokia_srlinux         │ running │ 172.20.20.2       │
│                │          │                  │ ghcr.io/nokia/srlinux │         │ 3fff:172:20:20::2 │
│                │          ├──────────────────┼───────────────────────┼─────────┼───────────────────┤
│                │          │ clab-basic-srsim │ nokia_srsim           │ running │ 172.20.20.3       │
│                │          │                  │ nokia_srsim:25.7.R1   │         │ 3fff:172:20:20::3 │
╰────────────────┴──────────┴──────────────────┴───────────────────────┴─────────┴───────────────────╯
```

The output will contain all labs and their nodes.

Shortcuts:

* `clab ins` == `containerlab inspect`
* `clab ins -a` == `containerlab inspect --all`

## Lab directory

Lab directory stores the artifacts generated by containerlab that are related to the lab:

* tls certificates
* startup configurations
* inventory files
* topology export json file
* bind mounted directories

To list the contents of the lab directory, run:

```
[*]─[vm4]─[~/clab-workshop_snr/10-basics]
└──> tree -L 3 clab-basic/
```

## Destroying the lab

When you are done with the lab, you can destroy it. Containerlab can try and find the `*.clab.yml` file in the current directory and use it so that you don't have to type it out.  
Try it:

```bash
clab des --cleanup
```

Alternatively, you could specify the topology file explicitly:

```bash
clab des -t basic.clab.yml --cleanup
```

The `--cleanup` flag ensures that the lab directory gets removed as well.

You finished the basics lab exercise!
