commands-reference.md

```markdown
# Linux Network Troubleshooting Commands Reference

A practical reference for the commands used throughout the
Network Troubleshooting Lab.

---

# 1. Network Interface

## Show IP addresses

```bash
ip addr

Short form:

ip a

Use when:

Checking IP configuration

Checking whether an interface has an address

Identifying the active interface



---

Show interface state

ip link

Use when checking whether an interface is UP or DOWN.


---

2. Routing

Show routing table

ip route

Useful for identifying:

Default gateway

Network routes

Interfaces

Source addresses



---

Determine route to a destination

ip route get 8.8.8.8

This shows which route Linux would use for the destination.


---

3. Connectivity

Ping a destination

ping -c 4 8.8.8.8

Use to test basic IP connectivity.


---

Ping a gateway

ping -c 4 GATEWAY

Useful for separating local network problems from external connectivity problems.


---

4. Neighbor / ARP Information

Show neighbor table

ip neigh

More specific:

ip neigh show dev eth0

Useful when investigating local network or gateway reachability.

States such as:

REACHABLE
STALE
FAILED

can provide useful evidence.


---

5. DNS

Resolve a hostname

getent hosts example.com


---

Query DNS

dig example.com


---

Inspect local hostname mappings

cat /etc/hosts


---

Inspect DNS resolver configuration

cat /etc/resolv.conf


---

6. TCP Ports

Show listening TCP ports

ss -lntp

Options:

-l = listening
-n = numeric addresses/ports
-t = TCP
-p = process information


---

Check a specific port

ss -lntp | grep :8080


---

Identify a process using a port

lsof -i :8080


---

7. HTTP / HTTPS

Check HTTP response headers

curl -I http://example.com


---

Verbose HTTP/HTTPS troubleshooting

curl -v http://example.com

HTTPS:

curl -Iv https://example.com

Useful for seeing:

Connection establishment

HTTP status

TLS information

Certificate verification

Headers



---

8. TLS

Inspect TLS connection

openssl s_client -connect example.com:443

Useful for investigating:

TLS handshake

Certificate chain

Protocol

Certificate information



---

9. Linux Services

Check service status

systemctl status SERVICE

Example:

systemctl status nginx


---

Start a service

sudo systemctl start SERVICE


---

Stop a service

sudo systemctl stop SERVICE


---

Restart a service

sudo systemctl restart SERVICE


---

Check whether a service is enabled

systemctl is-enabled SERVICE


---

10. Logs

View service logs

journalctl -u SERVICE


---

Show recent service logs

journalctl -u SERVICE -n 50


---

Follow service logs

journalctl -u SERVICE -f


---

11. Firewall

For systems using UFW:

Check firewall status

sudo ufw status


---

Show numbered rules

sudo ufw status numbered


---

Allow a port

sudo ufw allow 8080/tcp


---

Remove a rule

sudo ufw delete allow 8080/tcp

Only change firewall configuration after collecting evidence and confirming that the firewall is actually responsible for the failure.


---

12. SSH

Connect using SSH

ssh user@HOST


---

Test SSH port

ss -lntp | grep :22


---

Check SSH service

systemctl status ssh


---

View SSH logs

journalctl -u ssh


---

13. Latency

Basic latency test

ping -c 10 example.com

Look at:

packet loss
min
avg
max

High average latency or packet loss can indicate a network-path problem.


---

14. Useful Combined Checks

Check interface + routing

ip addr
ip route


---

Check routing + gateway

ip route
ip neigh
ping -c 4 GATEWAY


---

Check service + port

systemctl status SERVICE
ss -lntp


---

Check backend

ss -lntp
curl http://127.0.0.1:PORT


---

Check HTTPS

curl -Iv https://HOST
openssl s_client -connect HOST:443
