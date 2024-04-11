# mininet_frr
Run FRR on Mininet topologies

# Purpose

The Mininet network emulator enables the configuration and testing of routing 
protocols on various topologies. Mininet runs on Linux and uses network namespaces
and Linux kernel routing to emulate host, switches, and routers. These examples include
FRR configurations and topologies to exercise FRR network protocols.

# Overview

This repository contains various topologies and configurations that 
run on a Linux system with both FRR and Mininet installed. While not strictly
necessary, running Mininet and FRR in a VM is recommended. Clone the repostory
on the FRR/Mininet system to access the examples.

## Running a Toplogy

Each example is standalone. To run, open a Linux terminal and:
- cd into the example directory
- configure FRR with the configure script (note: uses sudo and erases the previous configurations)
- start mininet with the sudo command

```
cd two_router
./config_frr.sh
sudo mn_topo.py
```

This starts the mininet emulator and the relevant FRR routing deamons, and brings up the 
mininet command line. Mininet provides access to the shell on each emulated node
and connectivity tests. To stop the emulation and shutdown the FRR daemons, enter exit or press ^D.

At the mininet> prompt:
```
pingall
r1 ip route
```

Run the FRR shell from a Linux termimal for specific nodes: vtysh -N "node name"

```
sudo vtysh -N r1
```
## Examples

- two_router: 2 routers configured static routes 
- ospf:  routers running OSPF
- bpg: 4 routers in 3 AS regions running BGP 

Each of the examples workds in that the routing configuration provides connectivity, but are likely
not good examples of how to do things in the right way. There are many improvements to be made.


# Requirements
It is assumed that FRR uses the following directories:

- /etc/frr : configuration
- /var/frr : run state
- /usr/lib/frr/frrinit.sh : start script

The scripts in this project create a directory for each routing node under /etc/frr, /var/frr, and /var/log/frr

You can build your own FRR/Mininet VM image by building from source. Instructions are WIP



