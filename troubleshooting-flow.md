`troubleshooting-flow.md`

```markdown
# Troubleshooting Flow

This document describes the troubleshooting methodology used throughout
the Network Troubleshooting Lab.

The core principle is:

> Do not guess the cause. Collect evidence, form a hypothesis, test it,
> and verify the fix.

---

# General Flow

```text
                 INCIDENT
                    |
                    v
             Identify Symptom
                    |
                    v
             Reproduce Failure
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
             +------+------+
             |             |
          Confirm       Disprove
             |             |
             v             v
         Root Cause    New Hypothesis
             |
             v
             Fix
             |
             v
        Verify Recovery
             |
             v
        Document Result


---

Layer 1 — Application Symptom

Start with what the user or application is actually experiencing.

Examples:

curl -I http://HOST
curl -Iv https://HOST
ssh user@HOST
ping -c 4 HOST

Questions:

What exactly failed?

Can the failure be reproduced?

Is the failure consistent?



---

Layer 2 — Network Interface

Check the local interface and IP configuration.

ip addr
ip link

Questions:

Is the interface UP?

Does it have an IP address?

Is the expected interface being used?



---

Layer 3 — Routing

Check how Linux intends to reach the destination.

ip route
ip route get DESTINATION

Questions:

Is there a default route?

Which gateway is being used?

Which interface will carry the traffic?



---

Layer 4 — Gateway / Local Network

Test the next hop.

ping -c 4 GATEWAY
ip neigh

If the gateway cannot be reached, investigate:

Interface configuration

Routing

Neighbor/ARP state

Local network connectivity



---

Layer 5 — DNS

If IP connectivity works but a hostname does not resolve, investigate DNS.

getent hosts HOSTNAME
dig HOSTNAME
cat /etc/resolv.conf

Also inspect local hostname overrides:

cat /etc/hosts

Questions:

Does the hostname resolve?

Which DNS server is being used?

Is the DNS server reachable?



---

Layer 6 — TCP / Port

Once the network path appears functional, determine whether the expected service port is actually listening.

ss -lntp

For a specific port:

ss -lntp | grep :PORT

Another useful check:

lsof -i :PORT

Important distinction:

Port not listening
        !=
Firewall blocking traffic

Evidence must determine which condition exists.


---

Layer 7 — Firewall

If the service is listening but connections are being rejected or blocked, inspect firewall rules.

For UFW:

sudo ufw status
sudo ufw status numbered

Questions:

Is the firewall enabled?

Is the required port allowed?

Is traffic being denied?



---

Layer 8 — Service

If the expected port is not listening, investigate the service.

systemctl status SERVICE

Inspect logs:

journalctl -u SERVICE

Possible causes include:

Service stopped

Startup failure

Configuration error

Dependency failure

Process crash



---

Layer 9 — Backend

For an application behind a reverse proxy:

Client
  |
  v
Reverse Proxy
  |
  v
Backend Application

Test the backend directly when possible:

curl http://127.0.0.1:PORT

Then test the proxy:

curl -I http://HOST

This helps separate:

Reverse proxy failure
        from
Backend failure


---

Layer 10 — HTTP

Inspect HTTP responses directly.

curl -I http://HOST
curl -v http://HOST

For a 502 response, investigate the communication path:

Client
  |
  v
Reverse Proxy
  |
  v
Backend

Check:

Is the proxy running?

Is the backend running?

Is the backend listening?

Can the proxy reach the backend?

What do the logs say?



---

Layer 11 — HTTPS / TLS

For HTTPS problems, investigate in order:

DNS
 |
 v
TCP
 |
 v
TLS Handshake
 |
 v
Certificate
 |
 v
Hostname Verification
 |
 v
HTTPS

Useful commands:

curl -Iv https://HOST
openssl s_client -connect HOST:443

Questions:

Does DNS resolve?

Does TCP connect?

Does TLS negotiation succeed?

Is the certificate valid?

Does the certificate match the hostname?

Is the certificate trusted?



---

Layer 12 — Latency and Packet Loss

For slow or unstable connections:

ping -c 10 HOST

Look for:

High latency

Packet loss

Large latency variation

Unreachable destinations


Compare different points when possible:

Local host
    |
    v
Gateway
    |
    v
Remote destination

This helps determine where the degradation occurs.


---

Root Cause Process

Use:

Observation
     |
     v
Hypothesis
     |
     v
Test
     |
     v
Evidence
     |
     v
Root Cause

Example:

Symptom:
Backend connection refused

        |
        v

Hypothesis:
Backend service is not running

        |
        v

Evidence:
ss shows no listener
systemctl shows service inactive

        |
        v

Root Cause:
Backend service stopped

        |
        v

Fix:
Start the service

        |
        v

Verification:
Service is active
Port is listening
curl succeeds


---

Verification

A troubleshooting task is not complete when the configuration is changed.

The original failure must be tested again.

Examples:

systemctl status SERVICE
ss -lntp
curl -I http://HOST
ping -c 4 HOST

The final question is:

> Did the original problem actually disappear?




---

Evidence Standard

Each incident should contain:

1. Symptom


2. Reproduction


3. Initial evidence


4. Hypothesis


5. Investigation


6. Root cause


7. Fix


8. Verification



This keeps troubleshooting evidence-driven rather than trial-and-error.
