# Create a Mininet / FRR VM
These are instructions to create an Ubuntu based Mininet/FRR VM image with:
- Ubuntu 22.04.4 LTS
- Mininet 2.3.0
- FRR 9.1

Only these versions have been tested, other combinations could have problems.

The following are the steps to download the Ubuntu iso image, use git to fetch Mininet and FRR 
sources from github.com, and build and install Mininet and FRR.  Building from source is a bit
more effort, but ensures you get exactly what you want.

## Step 1: Create a Ubuntu based VM

There are multiple options for working with VMs. 
[Virtual Box](https://www.virtualbox.org/) is probably the easiest and a good
place to start if you are not familiar with working with VMs. I use 
[virt-manager](https://virt-manager.org/) on Linux to work with qemu/KVM based
virtual machines, and these instructions refer to virt-manager commands. 
Virtual Box has operations similar to virt-manager.

### Download Ubuntu 22.04.4 LTS ISO Image

We use the Ubuntu server image instead of the desktop image since the server is more lightweight and 
all access is done though the terminal. You can use the desktop image if you want to run
tests that include a web browser and such. But make sure you use the version [22.04.4](https://releases.ubuntu.com/22.04/)

Get the file: ubuntu-22.04.4-live-server-amd64.iso

### Create the VM and Run the Ubuntu Install

Run virt-manager and use the GUI to step through creating a new VM and
installing Ubuntu Linux:

File -> New VM
- Select 'local install media'
- Choose the downloaded ubuntu iso file: **ubuntu-22.04.4-live-server-amd64.iso**
- 8192 memory
- 4 CPUs
- 32 GB storage
- set the name to **vm-frr**
- Go

After the VM starts, select "Try or Install Ubuntu". Walk through the install,
select the default options.

When prompted for names and passwords, I use the following:
- Your name: *anything*
- Your server's name: **frr-system**
- Pick a user name: **ubuntu**
- Choose a password: **ubuntu**
- Confirm your password: **ubuntu**

You may pick different values for the user name and password, just do not forget 
what you picked and do not choose **frr** as a user name.
That user will be created later, dedicated to running FRR.

Select the option to install the OpenSSH server and allow password access.
An SSH client is the easiest way to run multiple terminal sessions in the
VM at one time.  You can also import an SSH identity to avoid typing a 
password when logging in.
(Importing the identity from Github works great if you have that setup on
github.com).

Once the install completes, select Reboot now. If the screen pauses and 
complains about the cdrom, just press ENTER.

Once the reboot is complete, you can login on the VM terminal with the username ubuntu and the password ubuntu.
However, I feel the best interaction with the VM is by logging in over SSH.

If you are using VirtualBox defaults, you will need to add a port forwarding rule
for ssh access. See these [instructions](https://nsrc.org/workshops/2014/sanog23-virtualization/raw-attachment/wiki/Agenda/ex-virtualbox-portforward-ssh.htm) for port forwarding.

When using virt-manager, I login into the VM by running ssh connecting to the IP address of the VM.
I find the IP address to connect to through the virt-manager:

  - View -> Details
  - NIC: :xx:xx:xx  -> IP address


The IP address can also be found by connecting to the VM terminal with the user name and password and running the 
ip command:

```
ip a
```

Look for inet *address* under the enp1so: (or similar) device

At this point, you can ssh into the system using the ip address with the username
ubuntu and password ubuntu (substitute your own IP address):

```
ssh ubuntu@192.168.122.109
```

The rest of this process will be done inside the VM.

Upgrade Ubuntu to the latest packages and install additional packages:
```
sudo apt update
```

```
sudo apt upgrade
```


## Step 2: Get the mininet_frr repo

Log into the VM. 
These instructions assume everything is done in the home directory of 
your user account on the VM.

Use git to copy the Mininet/FRR integration scripts and configs:

```
git clone https://github.com/jmwanderer/mininet_frr
```

## Step 2: Build and Install Mininet

Here I roughly follow Option 2 of the [Mininet download instuctions](https://mininet.org/download/):

Clone the mininet git repository and checkout version 2.3.0

```
git clone https://github.com/mininet/mininet 
```

```
cd mininet
git checkout -b mininet-2.3.0 2.3.0
```

Apply a basic patch included in this repo to fix the 
Mininet source installation (this is fixed in later versions of Mininet):

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

This is the most complex task, but if you follow the instructions, FRR will build and 
install correctly in the VM.

I use the instructions for 
[installing FRR from source](https://docs.frrouting.org/en/latest/installation.html)
on the 
[Ubuntu 22.04 platform](https://docs.frrouting.org/projects/dev-guide/en/latest/building-frr-for-ubuntu2204.html)

### Install FRR Build Dependencies

```
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
```

```
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
```

```
make
```

```
sudo make install
```

### Install FRR Configuration Files

While the mininet scripts do not run an FRR instance in the default namespace,
we need to install the FRR configuration files to provide a complete install of FRR to
allow the integration to function.


From the frr directory, run the following:

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
If you want to use MPLS, there are more steps found in the FRR instructions. 
(Background note: because we include an ip forwarding directive in the FRR configs, we do not
need to modify the system sysctls.conf file.)


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














