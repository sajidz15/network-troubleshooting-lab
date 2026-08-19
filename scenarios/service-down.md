# Incident 2 — Web Service Down

## Problem

Users were unable to access the application and received a connection refused error.

## Customer Report

> "The application is refusing connections. Please investigate and restore service."

## Investigation

1. Test hostname resolution

```bash
getent hosts app.lab.local
Result: The hostname resolved successfully.

2. Test network connectivity
ping -c 5 172.22.236.124
Result: 0% packet loss.
The server was reachable.

3. Test TCP port 80
nc -zv 172.22.236.124 80
Result: Connection refused.
This indicated that the server was reachable, but the TCP connection to port 80 was being refused.

4. Check whether port 80 was listening
ss -lntp | grep ':80'
Result: No output.
Nothing was listening on TCP port 80.

5. Check the web service
systemctl status nginx --no-pager
Result: Nginx was inactive/stopped.
Root Cause
The Nginx web service was stopped, so nothing was listening on TCP port 80.

Resolution
Started Nginx:
sudo systemctl start nginx
Verify the service
systemctl status nginx --no-pager
Nginx was active and running.
Verify port 80
ss -lntp | grep ':80'
Port 80 was listening again.
Verify HTTP access
curl http://app.lab.local
The application responded successfully.

Final Status
Resolved ✅
Troubleshooting Lesson
The server was reachable, but the application port was refusing connections.
The investigation followed:
DNS → Connectivity → Port → Listening Process → Service → Fix → Verification
The service was restarted only after evidence confirmed that Nginx was stopped.
