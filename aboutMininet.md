Mininet is a network emulator.
# It has
Real IP stack

Real TCP/UDP/ICMP

Real switches (Open vSwitch)

Real routing & forwarding

But all inside one Linux kernel.

# Mininet has only three real building blocks:

Hosts

Switches

Links

Hosts (h1, h2, h3…)
What a host is

A Mininet host is:

A Linux network namespace acting like a computer.

Each host has:

Its own IP address

Its own network interface

Its own routing table

Its own processes

But:

It shares the same kernel as the VM

It does NOT see other hosts unless connected

How hosts are created (internally)

When you run:

sudo mn --topo linear,4


Mininet:

Creates 4 Linux network namespaces

Names them h1, h2, h3, h4

Assigns IPs like:

h1 → 10.0.0.1

h2 → 10.0.0.2


What hosts can do

Hosts behave like real machines:

h1 ping h4
h1 iperf -c h4
h1 curl http://h4
h1 bash


They can:

Send packets

Run servers

Act as clients

Run applications

2️⃣ Switches (s1, s2…)
What a switch is

A Mininet switch is usually:

Open vSwitch (OVS) running in kernel space

This is the same software used in real data centers.

A switch:

Does packet forwarding

Does not have an IP (by default)

Works at Layer 2 (Ethernet

What is OVS?
Open vSwitch (OVS) is an open-source, multilayer virtual switch designed for high-scale network automation in virtualized environments. It functions similarly to a physical Ethernet switch but resides within a software layer, typically used to connect virtual machines (VMs) and containers within and across physical servers. 


How switches work internally

When Mininet creates a switch:

It creates an OVS bridge

Each connected link becomes an OVS port

Packets entering one port are forwarded out another

You can inspect (inside mininet>):

sh ovs-vsctl show


This command runs on the VM, but shows Mininet switches.

Controller interaction (basic idea)

Switches don’t decide everything alone.

They:

Ask a controller what to do (via OpenFlow)

Cache rules in a flow table

Forward packets based on rules

Default Mininet controller = simple learning switch.

3️⃣ Links (connections)
What a link really is

A Mininet link is:

A pair of virtual Ethernet interfaces (veth pair)

Example:

One end → h1-eth0

Other end → s1-eth1

These behave like a real Ethernet cable.

How links are created

Mininet:

Creates veth pairs

Moves each end into the correct namespace

Brings interfaces UP

You can see links:

net


And interfaces:

h1 ifconfig
s1 ifconfig

What links can simulate

Links are powerful. They can simulate:

Delay

Bandwidth limits

Packet loss

Queue size

Example:

sudo mn --topo linear,4 --link tc


This uses Linux Traffic Control (tc).

How packets actually flow (step-by-step)

Let’s say:

h1 ping h4

What really happens

h1 creates an ICMP packet

Packet goes out via h1-eth0

Enters switch s1

Switch checks flow table

Forwards to next switch

Reaches h4

Reply follows reverse path

Every step is real kernel networking.

Why Mininet is fast

Because:

No hardware

No virtual NIC emulation

No hypervisor networking

Everything runs inside one Linux kernel.

That’s why Mininet can create 1000+ nodes easily.

Functionalities Mininet provides (this is what you can DO)
✅ Topology creation

Linear

Tree

Custom Python topologies

✅ Traffic generation

ping (ICMP)

iperf (TCP/UDP)

HTTP, sockets, custom apps

✅ Network emulation

Delay

Loss

Bandwidth shaping

Congestion

✅ SDN experimentation

OpenFlow switches

External controllers (Ryu, POX)

Flow rule control

✅ Education & testing

Safe (no real network damage)

Reproducible experiments

Scriptable

Where VM and SSH fit (again, clearly)
You are typing in	What it controls
PowerShell	Just connects
SSH shell ($)	VM OS
mininet>	Virtual network

SSH does NOT change how Mininet works.
It’s just a better keyboard.

One killer analogy (remember this)

VM = Building

Mininet = Mini city inside the building

Hosts = Houses

Switches = Roads/intersections

Links = Roads

Controller = Traffic police

SSH = You entering the building through a different door.



# main points
Excellent — this is a **very real, practical question**, and the confusion is 100% justified.
Let’s clear it **cleanly and correctly**.

---

## What’s happening right now (your screenshot explained)

You are here:

```
mininet>
```

And you ran:

```bash
h2 iperf -s -u
```

This means:

* You started a **UDP iperf server** (video-like traffic)
* It is **running in the foreground**
* Mininet CLI is now **blocked** by that process

So you **cannot type another command** until it stops.

This is normal Linux behavior.

---

## The key concept you need (very important)

👉 **One terminal can run only one foreground process at a time**

To run **multiple servers simultaneously**, you have **two correct options**.

---

# OPTION 1 (BEST & SIMPLE): Run servers in the background

This is what you should do.

### Step 1: Stop the current UDP server

Press:

```
Ctrl + C
```

You should return to:

```
mininet>
```

---

### Step 2: Start UDP server in background

```bash
h2 iperf -s -u &
```

Notice the `&` at the end.

This means:

* “Run this in the background”
* Mininet CLI stays usable

You’ll see something like:

```
[1] 1234
```

That’s good.

---

### Step 3: Start TCP server (also background)

Now run:

```bash
h2 iperf -s &
```

Now **both servers are running on h2**:

* UDP → video traffic
* TCP → file download

---

### Step 4: Generate traffic from h1

```bash
h1 iperf -c h2 -u -b 10M
h1 iperf -c h2
```

Now you have:

* UDP stream competing with TCP stream
* Exactly what you wanted

---

# OPTION 2 (ADVANCED): Use two Mininet terminals

You can also:

* Open a **second SSH session**
* Or use another Mininet CLI window

Example:

* Terminal 1: `h2 iperf -s -u`
* Terminal 2: `h2 iperf -s`

This works, but it’s unnecessary right now.

---

## Why this models real life correctly

In real servers:

* One process handles video
* Another handles file downloads
* OS schedules both

Mininet hosts (`h2`) are **real Linux environments**, so this behavior is authentic.

---

## Important clarification (don’t miss this)

* UDP server and TCP server can **both listen on port 5001**
* Because:

  * One is UDP
  * One is TCP
* Different protocols → no conflict

This is correct networking behavior.

---

## Mental model (lock this in)

* `iperf -s` = service
* `&` = “run in background”
* Mininet hosts behave like **real Linux machines**

Nothing special. Nothing fake.

---

## You are now at an important milestone

You have successfully:

* Emulated **video traffic**
* Emulated **file transfer**
* Run **competing flows**
* Inside a **controlled network**

