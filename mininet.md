

# Mininet VM Setup & Working Commands (Verified)

## 1. Login to Mininet VM

(Default credentials)

```text
Username: mininet
Password: mininet
```

---

## 2. Update system (optional but recommended)

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 3. Verify Mininet is installed

```bash
mn --version
```

If this works, Mininet is correctly installed.

---

## 4. Clean any previous Mininet state (IMPORTANT)

Always run before starting Mininet:

```bash
sudo mn -c
```

---

## 5. Start Mininet with a linear topology (4 hosts)

This command **must be run from Linux shell (`$`)**:

```bash
sudo mn --topo linear,4
```

Successful start ends with:

```text
mininet>
```

---

## 6. Mininet CLI commands (ONLY work inside `mininet>`)

### List all nodes

```bash
nodes
```

### Display network topology

```bash
net
```

### Test connectivity between all hosts

```bash
pingall
```

### Ping a specific host

```bash
h1 ping h4
```

---

## 7. Sending packets using iperf (TCP traffic)

### Start iperf server on destination host

```bash
h4 iperf -s
```

(or background it)

```bash
h4 iperf -s &
```

### Send packets from source host

```bash
h1 iperf -c h4
```

This confirms **real TCP packet transfer** inside the Mininet network.

---

## 8. Packet capture (proof of packet flow)

Capture packets on switch interface:

```bash
s1 tcpdump -i s1-eth1
```

Then generate traffic:

```bash
h1 ping h4
```

Live packets will be visible.

---

## 9. Exit Mininet safely

```bash
exit
```

---

## 10. Cleanup after exiting (MANDATORY)

```bash
sudo mn -c
```

Prevents controller and interface conflicts.

---

## 11. SSH into Mininet VM (from host machine – optional but useful)

```powershell
ssh mininet@<VM-IP>
```

Step 1: Start Mininet (from SSH or VM — same thing)
sudo mn --topo linear,4


You must see:

mininet>

Step 3: Understand what h1 really is

Inside mininet>:

h1 ifconfig


This shows:

h1 is a Linux system

With its own IP

Its own network interface

Step 4: Enter a host (this is important)
h1 bash


Now prompt changes.

You are inside h1.

Run:

ping (ip address of h4 .you can get it by running h4 ipconfig
exit


This is real Linux networking, just isolated.
