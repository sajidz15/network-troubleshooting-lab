# Incident 5 — Connectivity Failure

## Problem

The customer reported that the application server could no longer be reached, although the application had been working earlier.

## Customer Report

> "I can't reach the application server. The application was working earlier, but now connections are failing."

## Investigation

1. Verify DNS resolution

```bash
getent hosts app.lab.local

Result: app.lab.local resolved successfully to 172.22.236.124.

DNS was not the cause of the application connectivity problem.

2. Check network/application connectivity

ping -c 4 172.22.236.124

The application server responded.

3. Check the application port

ss -lntp | grep ':80'

Port 80 was listening.

4. Identify the service

sudo ss -lntp | grep ':80'

Nginx was listening on port 80.

5. Check Nginx health

systemctl status nginx --no-pager

Nginx was active and running.

6. Check routing

ip route

The default route was missing.

Expected route:

default via 172.22.224.1 dev eth0

The missing default route was identified as the network connectivity problem.

Root Cause

The server's default network route had been removed.

Without the default route, the system could not properly route traffic outside its directly connected network.

Resolution

The default route was restored:

sudo ip route add default via 172.22.224.1 dev eth0

Verification

Verify the route:

ip route

Expected:

default via 172.22.224.1 dev eth0

Then verify application access:

curl http://app.lab.local

The application responded successfully.

Additional Investigation

An external DNS test was also performed:

nslookup example.com 10.255.255.254

The request timed out.

This was not used as the root cause because app.lab.local resolved successfully and the application itself was reachable after restoring the route. The external DNS behavior was treated separately from the customer's application connectivity issue.

Final Status

Resolved ✅

Troubleshooting Lesson

A connectivity problem does not automatically mean the application or service is down.

The investigation followed the network path:

DNS → Connectivity → Port → Service → Routing → Fix → Verification

The missing default route was identified as the root cause.
