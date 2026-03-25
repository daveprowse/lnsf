# MTU Jumbo Frames Lab — KVM VM Network Performance Test

## Overview

Tests network throughput between two KVM VMs at MTU 1500 vs MTU 9000 (jumbo frames).
Uses `iperf3` directly — no scripts, no SSH keys required.

| VM | IP | OS | Role |
|---|---|---|---|
| debclient | 10.0.2.52 | Debian | iperf3 client |
| fedora-tester | 10.0.2.62 | Fedora | iperf3 server |
| KVM Host | — | — | Bridge (virbr1 / 10-network) |

> **Note:** Both VMs use `enp1s0` and NetworkManager connection name `"Wired connection 1"`

---

## Prerequisites

Install iperf3 on both VMs if not already done:

```bash
# Debian
sudo apt install iperf3 -y

# Fedora
sudo dnf install iperf3 -y
```

---

## Part 1 — Baseline Test at MTU 1500

### Step 1 — Verify both VMs are at MTU 1500

On Debian:
```bash
ip link show enp1s0 | grep mtu
```

On Fedora:
```bash
ip link show enp1s0 | grep mtu
```

Both should show `mtu 1500`.

### Step 2 — Start iperf3 server on Fedora

```bash
iperf3 -s -p 5201
```

Leave this running.

### Step 3 — Run test from Debian

```bash
iperf3 -c 10.0.2.62 -p 5201 -t 10
```

Note the final **sender** Mbps from the summary line at the bottom.

### Step 4 — Stop iperf3 server on Fedora

Press `Ctrl+C`

---

## Part 2 — Set MTU 9000 (Jumbo Frames)

### Step 1 — Set KVM Bridge to MTU 9000 (on KVM Host)

```bash
sudo virsh net-edit 10-network
```

Add the MTU line inside `<network>`:
```xml
<network>
  <name>10-network</name>
  <mtu size='9000'/>
  ...
```

Apply the change:
```bash
sudo virsh net-destroy 10-network
sudo virsh net-start 10-network
sudo systemctl restart libvirtd
```

Verify:
```bash
ip link show virbr1 | grep mtu
# Expected: mtu 9000
```

### Step 2 — Set MTU 9000 on Debian

```bash
sudo nmcli con mod "Wired connection 1" ethernet.mtu 9000
sudo nmcli con up "Wired connection 1"
ip link show enp1s0 | grep mtu    # verify: mtu 9000
```

### Step 3 — Set MTU 9000 on Fedora

```bash
sudo nmcli con mod "Wired connection 1" ethernet.mtu 9000
sudo nmcli con up "Wired connection 1"
ip link show enp1s0 | grep mtu    # verify: mtu 9000
```

### Step 4 — Verify connectivity

```bash
ping 10.0.2.62    # from Debian
```

---

## Part 3 — Jumbo Frames Test at MTU 9000

### Step 1 — Start iperf3 server on Fedora

```bash
iperf3 -s -p 5201
```

### Step 2 — Run test from Debian

```bash
iperf3 -c 10.0.2.62 -p 5201 -t 10
```

Note the final **sender** Mbps from the summary line.

### Step 3 — Stop iperf3 server on Fedora

Press `Ctrl+C`

---

## Part 4 — Restore MTU 1500

### Step 1 — Restore KVM Bridge to MTU 1500 (KVM Host)

```bash
sudo virsh net-edit 10-network
```

Change the MTU line:
```xml
<mtu size='1500'/>
```

Apply:
```bash
sudo virsh net-destroy 10-network
sudo virsh net-start 10-network
sudo systemctl restart libvirtd
```

Verify:
```bash
ip link show virbr1 | grep mtu
# Expected: mtu 1500
```

### Step 2 — Restore MTU 1500 on Debian

```bash
sudo nmcli con mod "Wired connection 1" ethernet.mtu 1500
sudo nmcli con up "Wired connection 1"
ip link show enp1s0 | grep mtu    # verify: mtu 1500
```

### Step 3 — Restore MTU 1500 on Fedora

```bash
sudo nmcli con mod "Wired connection 1" ethernet.mtu 1500
sudo nmcli con up "Wired connection 1"
ip link show enp1s0 | grep mtu    # verify: mtu 1500
```

---

## Results — What to Expect

| MTU | Sender Avg |
|-----|-----------|
| 1500 | _______ Gbits/sec |
| 9000 | _______ Gbits/sec |

### Important Caveat

Both VMs run on the **same physical host**, so traffic never touches real network
hardware. The virtio driver transfers data via shared memory, which means results
reflect RAM bandwidth rather than network performance — numbers in the 90-110
Gbits/sec range are normal and expected.

In production on **real physical hardware** across a switch, jumbo frames
typically yield 5-15% improvement on gigabit networks and more significant
gains on 10GbE for high-throughput workloads such as SANs, NFS, and
database replication — exactly the use case your attendee was asking about.