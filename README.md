# mininet_frr
Free Range Routing (FRR) on Mininet topologies

# Purpose

Network emulators enable hands-on learning, experimenting, and testing network 
protocols on virtual toplogies. Mininet uses Linux network name spaces for 
lightweight emulation of routers, switches, and hosts.

This package provides a basic integration of Mininet and FRR with a variety
of example topologies and configurations.

# Overview

This repository contains various topologies and configurations that 
run on a Linux system with FRR and Mininet installed. While not strictly
necessary, running Mininet and FRR in a VM is strongly recommended.

To run these examples, clone this repostory on an FRR/Mininet system and
run the individual Mininet Python scripts.

## Topologies

- two_router: 2 routers configured static routes 
- ospf:  4 routers using OSPF
- bpg: 4 routers in 3 AS regions using BGP 


## Running a Toplogy

Each scenario is standalone. To run a scenario, open a Linux terminal and
do the following:

- cd into the directory of the example that you want to run.
- configure FRR by running configure script.
- start the eumlation by running the python topology script

Example:
```
cd two_router
./config_frr.sh
sudo ./mn_topo.py
```

This starts the mininet emulator and the relevant FRR routing deamons, and brings up the 
mininet command line. Mininet provides access to the linux shell on each emulated node
and connectivity tests. To stop the emulation and shutdown the FRR daemons, enter exit or press ^D.

At the mininet> prompt, you can run tests:

For example:
```
r1 ping r3
pingall
```

To run Linux shell commands on individual nodes:
```
r1 ip route
```

To access the FRR command and configuration shell, run vtysh from a 
a Linux termimal for a specific node: vtysh -N "node name"

Example:
```
sudo vtysh -N r1
show running-config
```

## Notes

These examples focus on routing protocols and protocol configuration. 
We use Mininet switches
to connect multiple hosts in a submit.
Mininet supports Openflow by running a controller, however these examples do not
use Openflow, beyond whatever Mininet may be doing to make 
the switches "just work".

The configuration scripts config_frr.sh create subdirectories under the 
FRR configuration, run state, and logging directories and populates the configuration
files. Note that any existing configuration files in these directories are erased
as new ones are applied. Copy any updated configurations out of 
/etc/frr/*nodename* before reconfiguring.

While each of these scenarios "work" in that the routing configuration 
provides connectivity, they are not good examples of how to do things 
"in the right way". There are many improvements that can be made to the FRR configurations..

# Requirements
It is assumed that FRR uses the following directories, which are the default 
under Ubuntu Linux:

- /etc/frr : configuration
- /var/frr : run state
- /usr/lib/frr/frrinit.sh : start script

The scripts in this project create a directory for each routing node under 
/etc/frr, /var/frr, and /var/log/frr

You can build your own FRR/Mininet VM image.
Instructions for building from source can be found at:
[https://github.com/jmwanderer/mininet_frr/blob/main/make_vm/README.md](make_vm/README.md)


#  Additional Resources

An excellent description of network name spaces and mininet can be found
[Network Emulation using Network Namespaces and Mininet](https://www.inf.usi.ch/faculty/carzaniga/edu/adv-ntw/mininet.html).

Kudos to [edoardesd](https://stackoverflow.com/users/7892067/edoardesd) for an answer on Stack Overflow:
[(mininet) How to create a topology with two routers and their respective hosts
](https://stackoverflow.com/questions/46595423/mininet-how-to-create-a-topology-with-two-routers-and-their-respective-hosts) 

