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


## Stage 6: Linux System Hardening

The next stage of the project focused on establishing a basic security baseline for the Raspberry Pi and reducing unnecessary network exposure.

The approach used throughout this stage was:

**Observe → Identify → Assess → Change → Verify**

Rather than immediately disabling services or changing configuration, the system was first inspected to understand what was running and why.

---

### 6.1 Listening Port Enumeration

The Raspberry Pi's listening network sockets were inspected using:

```bash
ss -tuln
```

A more detailed enumeration was then performed using:

```bash
sudo ss -tulpn
```

This identified the processes responsible for the listening services.

The initial results included:

* **TCP 22** - SSH
* **TCP/UDP 111** - `rpcbind`
* **UDP 5353** - Avahi/mDNS
* **TCP 631** - CUPS, bound only to localhost

This provided an initial view of the Raspberry Pi's network attack surface.

---

### 6.2 Running Service Enumeration

The running system services were reviewed using:

```bash
systemctl list-units --type=service --state=running
```

Several services were identified, including:

* SSH
* NetworkManager
* Avahi
* Bluetooth
* CUPS
* `rpcbind`
* NFS-related services
* System logging and authentication services

The purpose of this review was to distinguish between services required for the current lab and services that could potentially increase the attack surface.

---

### 6.3 Investigating NFS and RPC Services

The installed packages were investigated after identifying `rpcbind` listening on port 111:

```bash
dpkg -l | grep -E 'nfs|rpcbind'
```

This identified NFS-related packages and `rpcbind`.

An attempt was made to remove the packages using:

```bash
sudo apt remove nfs-common rpcbind
```

However, the package manager proposed removing additional desktop-related packages.

The operation was cancelled rather than blindly accepting the proposed changes.

This highlighted an important system administration principle:

> Package dependencies should be understood before removing system components.

The packages were therefore left installed, while the unnecessary services were disabled instead.

---

### 6.4 Disabling Unnecessary Services

The NFS block mapping service was disabled:

```bash
sudo systemctl disable --now nfs-blkmap.service
```

The RPC binding service was then disabled:

```bash
sudo systemctl disable --now rpcbind.service
```

The associated socket was also disabled:

```bash
sudo systemctl disable --now rpcbind.socket
```

The listening ports were then checked again:

```bash
sudo ss -tulpn
```

Port **111**, previously associated with `rpcbind`, was no longer listening.

This provided verification that the change had successfully reduced the Raspberry Pi's network exposure.

---

### 6.5 Reviewing Localhost Services

Port 631 was identified as belonging to CUPS.

The service was found to be listening only on:

```text
127.0.0.1
[::1]
```

This means the service was bound to the local system rather than exposed directly to other devices on the network.

It was therefore left enabled rather than unnecessarily removing it.

This reinforced the principle that a service should be assessed based on its actual exposure and purpose before being disabled.

---

### 6.6 SSH Key Authentication

SSH was then hardened by moving from password authentication to public-key authentication.

An Ed25519 key pair was generated on the Windows administration machine:

```powershell
ssh-keygen
```

The public key was installed on the Raspberry Pi in:

```text
~/.ssh/authorized_keys
```

The SSH connection was then tested from a new PowerShell session.

The Raspberry Pi successfully authenticated using the private key and its associated passphrase.

The private key remains stored on the administration machine and is **not included in this repository**.

---

### 6.7 Disabling SSH Password Authentication

Once key-based authentication had been successfully verified, SSH password authentication was disabled.

The SSH configuration was edited using:

```bash
sudo nano /etc/ssh/sshd_config
```

The following configuration was applied:

```text
PasswordAuthentication no
```

Before applying the configuration, the SSH daemon configuration was validated:

```bash
sudo sshd -t
```

No errors were returned.

The SSH service was then reloaded:

```bash
sudo systemctl reload ssh
```

A new SSH connection was established successfully using the Ed25519 key.

This confirmed that SSH remained accessible after password authentication was disabled.

---

### 6.8 Security Improvements

The changes made during this stage resulted in several improvements to the Raspberry Pi's security posture:

* Unnecessary NFS-related functionality was disabled
* `rpcbind` was disabled
* TCP/UDP port 111 was removed from the exposed listening services
* SSH public-key authentication was configured
* SSH password authentication was disabled
* SSH configuration was validated before being reloaded
* Changes were independently verified after implementation
* The system was changed incrementally to reduce the risk of losing remote access

The current SSH access model therefore requires possession of the configured private key and its passphrase rather than relying on a traditional account password.

---

### 6.9 Lessons Learned

This stage demonstrated that system hardening is not simply a matter of disabling as many services as possible.

The process involved:

1. Identifying what was running
2. Understanding why it was running
3. Assessing whether it was required
4. Making the smallest appropriate change
5. Testing the result
6. Confirming that legitimate administration still worked

The failed package removal attempt was also useful practical experience. It demonstrated how Linux package dependencies can affect apparently simple security changes and why changes should be reviewed before being applied.

---

## Next Stage

The next stage will focus on **host-based firewall configuration**.

Planned work includes:

* Installing UFW
* Creating an SSH allow rule
* Enabling the firewall safely
* Verifying firewall status and rules
* Rechecking listening ports
* Testing SSH connectivity after firewall activation
* Documenting the resulting security baseline

The goal is to establish a simple defensive firewall policy while maintaining remote administrative access.

## Stage 7: Host-Based Firewall Configuration

The next stage of the project focused on implementing a host-based firewall on the Raspberry Pi.

The objective was to restrict unsolicited inbound network traffic while maintaining the SSH access required for remote administration.

UFW (Uncomplicated Firewall) was selected because it provides a straightforward interface for configuring the Linux firewall while still using the underlying netfilter framework.

---

### 7.1 Installing UFW

UFW was installed using:

```bash id="j7j4pd"
sudo apt install ufw
```

The installation completed successfully.

UFW was initially left disabled so that the firewall rules could be configured and reviewed before activation.

---

### 7.2 Configuring SSH Access

Because the Raspberry Pi is administered remotely over SSH, an SSH allow rule was created before enabling the firewall:

```bash id="b1m2k3"
sudo ufw allow ssh
```

UFW confirmed that rules were added for both IPv4 and IPv6.

The firewall status was then checked:

```bash id="c4d5e6"
sudo ufw status
```

The result showed:

```text id="f7g8h9"
Status: inactive
```

This confirmed that the SSH rule had been configured while the firewall remained disabled.

---

### 7.3 Enabling the Firewall

Once SSH access had been explicitly permitted, UFW was enabled:

```bash id="i1j2k3"
sudo ufw enable
```

The firewall was then inspected using:

```bash id="l4m5n6"
sudo ufw status verbose
```

The resulting configuration was:

```text id="o7p8q9"
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
```

This establishes a default-deny policy for incoming traffic while allowing normal outbound connections.

SSH access is explicitly permitted over TCP port 22 for both IPv4 and IPv6.

---

### 7.4 Verifying Remote Administration

A new SSH connection was established from the Windows administration machine after enabling UFW:

```powershell id="r1s2t3"
ssh mcctrl@ccmm-GITS001.local
```

Key-based authentication successfully completed using the configured Ed25519 SSH key.

This confirmed that enabling the firewall had not disrupted legitimate remote administration.

Maintaining the existing SSH session while performing this test provided an additional safeguard against accidental loss of access.

---

### 7.5 Final Network Exposure Check

The listening network sockets were checked again using:

```bash id="u4v5w6"
sudo ss -tulpn
```

The final listening services identified were:

```text id="x7y8z9"
TCP 22     SSH
UDP 5353   Avahi/mDNS
```

The previously identified `rpcbind` listener on port 111 was no longer present following the service hardening performed during Stage 6.

SSH remained available on both IPv4 and IPv6:

```text id="a1b2c3"
0.0.0.0:22
[::]:22
```

Avahi remained active for local network discovery:

```text id="d4e5f6"
0.0.0.0:5353
*:5353
```

---

### 7.6 Security Improvements

The Raspberry Pi now has a basic host-based firewall policy providing the following protections:

* UFW is enabled and active
* Incoming traffic is denied by default
* Outgoing traffic is allowed by default
* SSH access is explicitly permitted
* IPv4 and IPv6 SSH access are both configured
* Firewall logging is enabled at a low level
* Existing SSH administration was tested after firewall activation
* Network listening services were re-enumerated after the change

Combined with the service hardening performed during Stage 6, the Raspberry Pi now has a significantly smaller and more controlled network attack surface.

---

### 7.7 Lessons Learned

This stage demonstrated the importance of configuring firewall rules before enabling the firewall itself.

Because the Raspberry Pi is administered remotely, enabling a firewall without first permitting SSH could have resulted in the loss of remote access.

The process followed was:

1. Install the firewall
2. Configure the required SSH rule
3. Confirm the rule was present
4. Enable the firewall
5. Establish a new SSH connection
6. Re-enumerate listening services
7. Verify the final configuration

This provided practical experience with host-based network access control and the importance of verifying security changes after implementation.

---

## Next Stage

The next stage of the project will focus on **security monitoring and log analysis**.

Planned work includes:

* Reviewing SSH authentication logs
* Understanding Linux security-related log entries
* Monitoring failed authentication attempts
* Generating controlled security events
* Investigating the resulting logs
* Establishing a basic security monitoring workflow
* Documenting findings as incident-style reports

The goal is to move from simply hardening the system to actively monitoring and investigating security events occurring on the host.

