# Packet Filtering Project

## Introduction
This project involves filtering incoming packets to a network card using Linux systems. The implementation is performed on three Linux systems, with one acting as a router ("linux router") placed between the other two systems. The filtered traffic is forwarded between the systems.

## System Configuration

### Router (Linux Mint)
- Interface `ens33`
  - **IP Address**: `192.168.88.138`
  - **Netmask**: `255.255.255.0`
- Interface `ens38`
  - **IP Address**: `192.168.126.203`
  - **Netmask**: `255.255.255.0`

### Device 1 (Ubuntu 19.10)
- **IP Address**: `192.168.88.137`
- **Netmask**: `255.255.255.0`
- **Gateway**: `192.168.88.138`

### Device 2 (Ubuntu 18.04)
- **IP Address**: `192.168.126.200`
- **Netmask**: `255.255.255.0`
- **Gateway**: `192.168.126.203`

## System Configuration Commands

For each system, configure the IP address, netmask, and gateway using the following commands:

```bash
# Set IP address
ifconfig <interface> <ip_address>

# Set netmask
ifconfig <interface> netmask <subnet_mask>

# Set gateway
route add default gw <gateway_ip>
```

Replace `<interface>`, `<ip_address>`, `<subnet_mask>`, and `<gateway_ip>` with the respective values for each system.

## Implementation Steps

### Step 1: Enable IP Forwarding on the Router
Run the following command on the Linux Mint router to enable IP forwarding:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

### Step 2: Configure Traffic Control on Interface `ens38`
Run the following commands to configure traffic control for filtering traffic on the `ens38` interface:

```bash
# Add a root qdisc
 tc qdisc add dev ens38 handle 1: root htb r2q 1700

# Add a class to the qdisc
tc class add dev ens38 parent 1: classid 1:1 htb rate 100Mbps ceil 100Mbps

tc class add dev ens38 parent 1:1 classid 1:20 htb rate 100Mbps

# Add network emulation rules
tc qdisc add dev ens38 parent 1:20 handle 12: netem trace test.bin 10
```

### Step 3: Define Filters
Run the following commands to create filters that match specific packet attributes:

```bash
# Add filters
tc filter add dev ens38 parent 1:0 prio 1 protocol ip u32

tc filter add dev ens38 parent 1:0 prio 1 handle 1: u32 divisor 1

# Match traffic for HTTP (port 80)
tc filter add dev ens38 parent 1:0 prio 1 u32 ht 1: match tcp dst 80 0xffff match ip protocol 6 0xff match ip src 192.168.88.137/24 match ip dst 192.168.126.200 flowid 1:20

# Match traffic for FTP (ports 20, 21)
tc filter add dev ens38 parent 1:0 prio 1 u32 ht 1: match tcp dst 20 0xfffe match ip protocol 6 0xff match ip src 192.168.88.137/24 match ip dst 192.168.126.200 flowid 1:20
```

## References

- [Linux Advanced Routing and Traffic Control HOWTO](http://lartc.org/howto)
- [Netem Documentation](http://linux-net.osdl.org/index.php/Netem)
