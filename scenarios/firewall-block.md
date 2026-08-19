# Incident 3 — Firewall Blocking Application Traffic

## Problem

Users were unable to access the application even though the server and web service were running.

## Customer Report

> "The application is running, but users cannot connect to it. Please investigate and restore access."

## Investigation

1. Test hostname resolution

```bash
getent hosts app.lab.local
Result: The hostname resolved successfully through the local host configuration.

2. Test IP connectivity
ping -c 5 172.22.236.124
Result: 0% packet loss.
The server was reachable.

3. Test TCP port 80
nc -zv 172.22.236.124 80
Result: The TCP connection to port 80 failed.
This indicated that the server was reachable, but TCP traffic to the application port was not getting through.

4. Check whether port 80 was listening
ss -lntp | grep ':80'
Result: Port 80 was listening.
This ruled out a stopped web service as the immediate cause.

5. Identify the service
sudo ss -lntp | grep ':80'
Nginx was listening on port 80.

6. Check Nginx health
systemctl status nginx --no-pager
Result: Nginx was active and running.

7. Check Nginx logs
sudo journalctl -u nginx -n 50
No relevant errors were found.

8. Investigate the firewall
sudo ufw status verbose
A rule was found denying TCP port 80:
80/tcp DENY
Root Cause
UFW was actively blocking inbound TCP traffic to port 80.
The web server itself was healthy and listening correctly; the firewall prevented clients from reaching it.

Resolution
Removed the incorrect deny rule:
sudo ufw delete deny 80/tcp
Allowed HTTP traffic:
sudo ufw allow 80/tcp
Verify the firewall
sudo ufw status
The HTTP rule was now allowed.
Verify TCP connectivity
nc -zv 172.22.236.124 80
The connection succeeded.
Verify the application
curl http://app.lab.local
The application responded successfully.

Final Status
Resolved ✅
Troubleshooting Lesson
The server was reachable, port 80 was listening, and Nginx was healthy, but clients could not establish a TCP connection.
The investigation followed:
DNS → Connectivity → Port → Service → Logs → Firewall → Fix → Verification
The firewall was treated as a hypothesis first and confirmed only after the blocking rule was found.
