# raspberry-pi-home-security-lab
A practical home cybersecurity lab built on a Raspberry Pi 4, documenting Linux administration, networking, system hardening, monitoring and security testing. 

# Raspberry Pi Home Security Lab

## Project Overview

This project uses a Raspberry Pi 4 as the foundation for a small home cybersecurity and IT lab.

The aim is to build a practical environment for learning Linux administration, networking, system hardening, monitoring, and defensive cybersecurity techniques. The lab will be developed incrementally, with each stage documented and tested.

### Hardware

* Raspberry Pi 4 Model B
* MicroSD card
* Laptop/desktop used for remote administration
* Home Wi-Fi network

### Hostname

```text
ccmm-GITS001
```

---

## Stage 1: Raspberry Pi Installation

The Raspberry Pi OS operating system was installed using Raspberry Pi Imager.

The hostname was configured as:

```text
ccmm-GITS001
```

SSH was enabled during the initial setup so the Raspberry Pi could be administered remotely without requiring a monitor, keyboard, or mouse.

The initial connection was established using an Ethernet cable between the Raspberry Pi and laptop.

---

## Stage 2: Network Interface Verification

The Pi's network interfaces were inspected using:

```bash
ip link
```

The following interfaces were identified:

```text
lo      Loopback
eth0    Ethernet
wlan0   Wi-Fi
```

The Raspberry Pi 4 has built-in Wi-Fi, so no additional wireless adapter was required.

---

## Stage 3: Wi-Fi Configuration

The available wireless networks were scanned using:

```bash
sudo nmcli dev wifi list
```

The Raspberry Pi successfully detected the home wireless network.

The Wi-Fi interface was then verified using:

```bash
ip addr show wlan0
```

The Pi received a private IPv4 address through DHCP:

```text
<PI_IP>/24
```

The interface was confirmed as active:

```text
wlan0 ... state UP
```

---

## Stage 4: Internet Connectivity Testing

Connectivity to an external IPv4 address was tested using Google's public DNS server:

```bash
ping -c 4 8.8.8.8
```

Result:

```text
4 packets transmitted
4 packets received
0% packet loss
```

This confirmed that the Raspberry Pi could successfully communicate with the internet.

DNS resolution was then tested:

```bash
ping -c 4 google.com
```

The hostname successfully resolved and responded to ICMP requests.

This confirmed that both:

* Internet connectivity
* DNS resolution

were functioning correctly.

---

## Stage 5: SSH Over Wi-Fi

A second SSH session was established from the laptop using the Raspberry Pi's Wi-Fi address:

```bash
ssh <PI_USER>@<PI_IP>
```

Once the Wi-Fi SSH connection was confirmed, the Ethernet cable was disconnected.

The Raspberry Pi remained accessible over SSH.

A final connectivity test was performed:

```bash
ping -c 4 8.8.8.8
```

The Pi returned:

```text
4 packets transmitted
4 packets received
0% packet loss
```

The Wi-Fi interface was also checked again:

```bash
ip addr show wlan0
```

The Pi retained its private IPv4 address.

This confirmed that the Raspberry Pi was independently connected to the home network via Wi-Fi and no longer depended on the physical Ethernet connection.

---

## Current Network Architecture

The current lab configuration is:

```text
                    INTERNET
                        │
                  ┌─────▼─────┐
                  │ Home Router│
                  └─────┬─────┘
                        │
                     Wi-Fi
                        │
               ┌────────▼────────┐
               │  Raspberry Pi 4  │
               │  ccmm-GITS001    │
               │    <PI_IP>       │
               └────────▲─────────┘
                        │
                     Wi-Fi
                        │
                     Laptop
```

The Raspberry Pi is currently operating as a normal host on the home network. The existing home router remains responsible for routing and internet connectivity.

The Raspberry Pi is **not currently acting as a firewall, router, or network gateway**.

---

## Skills Demonstrated

This initial stage provided practical experience with:

* Raspberry Pi OS installation
* Linux command-line administration
* SSH remote administration
* Linux network interfaces
* Ethernet vs Wi-Fi networking
* IPv4 addressing
* CIDR notation
* DHCP
* DNS
* ICMP
* Basic network troubleshooting
* Wireless network configuration
* Remote administration without a physical display

---

## Next Stage

The next stage of the project will focus on **Linux system hardening**.

Planned areas include:

* Reviewing running services
* Updating the operating system
* SSH security
* User and privilege management
* Firewall configuration
* Removing unnecessary services
* Reviewing network exposure
* Basic system logging
* Establishing a security baseline

The lab will then be expanded into a practical environment for cybersecurity experimentation and defensive monitoring.
