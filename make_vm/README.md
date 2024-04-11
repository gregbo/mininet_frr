# Create a Mininet / FRR VM

These instructions produce an Ubuntu based Mininet/FRR VM image for:
- Ubuntu 22.04.4 LTS
- Mininet 2.3.0
- FRR 9.1

We use git to fetch Mininet and FRR from github and build from source.

## Step 1: Create a Ubuntu based VM

## Step 2: Start and Login to the VM

## Step 3: Get the mininet_frr repo

git clone htts://github.com/jmwanderer/mininet_frr

## Step 3: Build Mininet

<instructions reference>

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

## Step 4: Build FRR

<instructions reference>











