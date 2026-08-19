README.md

# Network Troubleshooting Lab

A hands-on Linux networking troubleshooting lab built using Ubuntu on WSL.

This project simulates common production-style networking, Linux service,
backend, DNS, HTTP/HTTPS, and connectivity incidents.

The goal is to demonstrate practical troubleshooting rather than simply
memorizing commands.

The investigation process used throughout the project is:

**Symptom → Evidence → Hypothesis → Investigation → Root Cause → Fix → Verification**

---

## Environment

- Ubuntu 22.04 LTS
- Windows Subsystem for Linux (WSL)
- Bash
- Linux networking utilities
- curl
- ping
- iproute2
- ss
- systemctl
- journalctl
- DNS utilities
- Firewall utilities

---

## Project Structure

```text
network-troubleshooting-lab/
├── README.md
├── troubleshooting-flow.md
├── commands-reference.md
└── scenarios/
    ├── connectivity-failure.md
    ├── backend-health-failure.md
    ├── dns-failure.md
    ├── firewall-block.md
    ├── high-latency.md
    ├── https-502.md
    ├── port-not-listening.md
    ├── routing-problem.md
    ├── service-down.md
    ├── ssh-connectivity.md
    └── tls-https-failure.md


---

Incidents Covered

1. Connectivity Failure

Investigates a situation where network connectivity is unavailable.

Areas investigated:

Network interface

IP address

Default route

Gateway

Neighbor/ARP state

External connectivity


Key commands:

ip addr
ip link
ip route
ip neigh
ping


---

2. Backend Health Failure

Investigates an unavailable backend application.

Areas investigated:

Backend process

Listening port

Service status

Application response

Service logs


Key commands:

curl
ss
systemctl
journalctl


---

3. DNS Failure

Investigates hostname resolution problems.

Areas investigated:

/etc/hosts

DNS resolver configuration

Hostname resolution

DNS server availability


Key commands:

getent hosts
dig
cat /etc/hosts
cat /etc/resolv.conf


---

4. Firewall Block

Investigates traffic being blocked by firewall rules.

Areas investigated:

Firewall status

Firewall rules

Listening ports

Network connectivity


Key commands depend on the firewall implementation, for example:

sudo ufw status
sudo ufw status numbered
ss -lntp


---

5. High Latency

Investigates unusually slow network responses.

Areas investigated:

Round-trip time

Packet loss

Network path

Gateway response


Key commands:

ping
ip route
ip route get


---

6. HTTPS 502

Investigates an HTTP 502 Bad Gateway response.

Typical architecture:

Client
  |
  v
Reverse Proxy
  |
  v
Backend Application

Areas investigated:

Reverse proxy

Backend availability

Backend listening port

Proxy-to-backend communication

Application logs


Key commands:

curl
ss
systemctl
journalctl


---

7. Port Not Listening

Investigates a service that is not accepting connections because the expected port is not listening.

Key commands:

ss -lntp
lsof -i :PORT
curl


---

8. Routing Problem

Investigates incorrect or unavailable routes.

Areas investigated:

Routing table

Default gateway

Destination route

Network interface

Neighbor state


Key commands:

ip route
ip route get
ip addr
ip neigh
ping


---

9. Service Down

Investigates a stopped or failed Linux service.

Areas investigated:

Service state

Service logs

Failed startup

Recovery


Key commands:

systemctl status
systemctl start
systemctl restart
journalctl


---

10. SSH Connectivity Failure

Investigates failure to establish an SSH connection.

Areas investigated:

Network connectivity

Port 22

SSH service

Firewall

SSH logs


Key commands:

ping
ss
systemctl
journalctl
ssh


---

11. TLS/HTTPS Failure

Investigates HTTPS/TLS connection and certificate problems.

Troubleshooting path:

DNS
 |
 v
TCP connection
 |
 v
TLS handshake
 |
 v
Certificate validation
 |
 v
HTTPS request

Key commands:

curl -Iv https://HOST
openssl s_client -connect HOST:443


---

Troubleshooting Philosophy

The project follows an evidence-driven troubleshooting approach.

Instead of immediately restarting a service or changing configuration, the investigation first attempts to determine where the failure occurs.

The basic process is:

Observe
   |
   v
Reproduce
   |
   v
Collect Evidence
   |
   v
Identify Failing Layer
   |
   v
Form Hypothesis
   |
   v
Test Hypothesis
   |
   v
Determine Root Cause
   |
   v
Apply Fix
   |
   v
Verify Recovery


---

Skills Demonstrated

Linux

Linux command line

Processes and services

systemd

journal logs

Configuration files

Service troubleshooting


Networking

IPv4 addressing

Routing

Default gateways

DNS

TCP/IP

Ports

Connectivity

Latency

Packet loss

Neighbor/ARP resolution


Web and Application Troubleshooting

HTTP

HTTPS

HTTP 502

Reverse proxy concepts

Backend services

TLS

SSH


Troubleshooting

Evidence collection

Hypothesis testing

Root-cause analysis

Layer-by-layer troubleshooting

Service recovery

Post-fix verification



---

Project Objective

The purpose of this lab is to demonstrate the ability to investigate infrastructure problems systematically.

For every incident, the investigation should answer:

1. What is failing?


2. Where is it failing?


3. Why is it failing?


4. What evidence proves the cause?


5. What was changed?


6. Does the system work after the fix?
