# Create a Mininet / FRR VM
These instructions produce an Ubuntu based Mininet/FRR VM image for:
- Ubuntu 22.04.4 LTS
- Mininet 2.3.0
- FRR 9.1

We use git to fetch Mininet and FRR from github and build from source.

## Step 1: Create a Ubuntu based VM

There are multiple options for working with VMs. 
[Virtual Box](https://www.virtualbox.org/) is probably the easiest and a good
place to start if you are not familiar with working with VMs. I use 
[virt-manager](https://virt-manager.org/) on Linux to work with qemu/KVM based
virtual machines, and these instructions refer to virt-manager commands. 

### Download Ubuntu 22.04.4 LTS ISO Image

I use the [Ubuntu Server](https://ubuntu.com/download/server) image for the 
Mininet/FRR system since all of my access is terminal based and it is a bit
more lightweight. You can also use the desktop verison if you want to run
tests that include browers, etc.

Get the file: ubuntu-22.04.4-live-server-amd64.iso

### Run the Ubuntu Install

I run virt-manager and use the GUI to step through creating a new VM and
installing Ubuntu Linux. There are various ways to do this. I create a VM
with 8 GB of RAM, 32 GB of storage, and 4 CPUs.

File -> New VM
- Select local install media
- Choose the ubuntu iso file
- 8192 memory
- 4 CPUs
- 32 GB storage
- set the name to vm-frr
- Go

After the VM starts, select Try or Install Ubuntu. Walk through the install,
I use the defaults.

When prompted for names and passwords, I use the following:
- Your name: *anything*
- Your servers name: frr-system
- Pick a user name: ubuntu
- Choose a password: ubuntu
- Confirm your password: ubuntu

You may select different values, just do not forget the user name and
password. Also, do not choose frr as a user. That user will be created
later, dedicated to running FRR.

Select the option to install the OpenSSH server and import an SSH identity.
(Importing the identify from Github works great if you have that setup).

If you do not use a secure password (as I did above), you probably should
not allow password auth over SSH.

Once the install completes, select Reboot now. If the screen pauses and 
complains about the cdrom, just press ENTER.

Once the reboot is complete, 
you can find the IP address either through the virt-manager or by
logging the VM terminal with the user name and password and running the 
ip command:

- virt-manager
  - View -> Details
  - NIC: :xx:xx:xx  -> IP address

- login
```
ip a
```

Look for inet *address* under the enp1so: device


From here, you can ssh into the system using the ip address and your
user name.  I find using my native terminal windows and SSH to be the best experience.

```
ssh ubuntu@192.168.122.109
```

Upgrade Ubuntu to the latest packages and install additional packages:
```
sudo apt update
sudo apt upgrade
```

The rest of this process will be done inside the VM.

## Step 2: Get the mininet_frr repo

These instructions assume everything is done in the home directory of 
your user account on the VM.

Use git to pull in this repository:

```
git clone https://github.com/jmwanderer/mininet_frr
```

## Step 2: Build and Install Mininet

I roughly followed the Option 2 of the [Mininet download instuctions](https://mininet.org/download/)

Clone the mininet git repository and checkout version 2.3.0

```
git clone https://github.com/mininet/mininet 
cd mininet
git checkout -b mininet-2.3.0 2.3.0
```

Apply a basic patch included in this repo to fix the 
Mininet source installation:

```
git apply ../mininet_frr/make_vm/mininet.patch
```

Run the mininet install script:

```
cd ..
mininet/util/install.sh -a
```

After this completes, mininet is installed on the system. You can test the 
functionality  by running:

```
sudo mn --switch ovsbr --test pingall
```


## Step 3: Build FRR

We use the instructions for 
[installing FRR from source](https://docs.frrouting.org/en/latest/installation.html)
on the 
[Ubuntu 22.04 platform](https://docs.frrouting.org/projects/dev-guide/en/latest/building-frr-for-ubuntu2204.html)

### Install Dependencies

```
sudo apt update
sudo apt-get install \
   git autoconf automake libtool make libreadline-dev texinfo \
   pkg-config libpam0g-dev libjson-c-dev bison flex \
   libc-ares-dev python3-dev python3-sphinx \
   install-info build-essential libsnmp-dev perl \
   libcap-dev libelf-dev libunwind-dev \
   protobuf-c-compiler libprotobuf-c-dev
```

Install pytest:

```
sudo apt install python3-pytest
```

An extra step for the libyang dependency (can be included in the previous,
but separated here to match the FRR instructions):

```
sudo apt install libyang2-dev
```

### Create the FRR User and Groups

```
sudo groupadd -r -g 92 frr
sudo groupadd -r -g 85 frrvty
sudo adduser --system --ingroup frr --home /var/run/frr/ \
   --gecos "FRR suite" --shell /sbin/nologin frr
sudo usermod -a -G frrvty frr
```

### Get the FRR 9.1 Source Code

Get the source from github and check out the 9.1 branch:
```
git clone https://github.com/FRRouting/frr.git
cd frr
git checkout stable/9.1
```

### Update the Build System

From the frr directory, run the FRR bootstrap script:

```
./bootstrap.sh
```

### Compile and Install

From the frr directory, run the commands to build and install FRR:

```
./configure \
    --prefix=/usr \
    --includedir=\${prefix}/include \
    --bindir=\${prefix}/bin \
    --sbindir=\${prefix}/lib/frr \
    --libdir=\${prefix}/lib/frr \
    --libexecdir=\${prefix}/lib/frr \
    --sysconfdir=/etc/frr \
    --localstatedir=/var/frr \
    --with-moduledir=\${prefix}/lib/frr/modules \
    --enable-configfile-mask=0640 \
    --enable-logfile-mask=0640 \
    --enable-snmp=agentx \
    --enable-multipath=64 \
    --enable-user=frr \
    --enable-group=frr \
    --enable-vty-group=frrvty \
    --with-pkg-git-version \
    --with-pkg-extra-version=-MyCustomFRR
make
sudo make install
```

### Install FRR Configuration Files

While the mininet scripts do not run an FRR instance in the default namespace,
we can install the FRR configuration files to provide a complete install of FRR.

```
sudo install -m 775 -o frr -g frr -d /var/frr
sudo install -m 775 -o frr -g frr -d /var/log/frr
sudo install -m 775 -o frr -g frrvty -d /etc/frr
sudo install -m 640 -o frr -g frrvty tools/etc/frr/vtysh.conf /etc/frr/vtysh.conf
sudo install -m 640 -o frr -g frr tools/etc/frr/frr.conf /etc/frr/frr.conf
sudo install -m 640 -o frr -g frr tools/etc/frr/daemons.conf /etc/frr/daemons.conf
sudo install -m 640 -o frr -g frr tools/etc/frr/daemons /etc/frr/daemons
```

At this point, the FRR/Mininet system is complete. 
There are more steps to enable MPLS found in the instructions. 
Because we include an ip forwarding directive in the FRR configs, we do not
need to modify the system sysctls.conf file.


### Run a Topology

Run the simple two_router topology to test.

```
cd ..
cd mininet_frr/two_router
./config_frr.sh
sudo ./mn_topo.py
```

Try the pingall command to verify connectivity.

Enter exit or ^D to stop mininet.














