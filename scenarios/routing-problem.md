Incident 9 — Routing Problem

Incident

Customer report:

> "The application server is running, but users cannot reach the application."



The application itself is assumed to be healthy. The goal is to determine whether the server has a valid route to the destination.


---

Initial Investigation

1. Check the routing table

ip route

Why?

The routing table tells us how the operating system decides where to send network traffic.

A healthy server normally has a default route similar to:

default via <gateway> dev eth0

During this incident, the default route was missing.

Only the directly connected network remained:

172.22.224.0/20 dev eth0 proto kernel scope link src 172.22.236.124

This means the server knows how to communicate with its local network, but it does not have a route for destinations outside that network.


---

2. Test external connectivity

ping -c 4 8.8.8.8

Result:

ping: connect: Network is unreachable

What does this tell us?

The server cannot find a route to the external destination.

This is stronger evidence of a routing problem than simply saying "the application isn't working."


---

3. Ask the kernel how it would route the traffic

ip route get 8.8.8.8

Result:

RTNETLINK answers: Network is unreachable

This confirms that the kernel has no valid route to the destination.


---

Hypothesis

Hypothesis:

> The server's default route is missing, preventing traffic from reaching networks outside the local subnet.




---

Root Cause

The default route was removed.

Without a default route, the server can communicate with destinations on its directly connected network, but it does not know where to send traffic destined for other networks.


---

Fix

Restore the correct default gateway for the system:

sudo ip route add default via <correct-gateway> dev eth0

Important: Do not blindly use a gateway copied from another machine. The gateway must belong to the server's actual network.

Verify the gateway and routing table with:

ip route

The default route should now appear:

default via <correct-gateway> dev eth0


---

Verification

Test external connectivity again:

ping -c 4 8.8.8.8

Then verify the route:

ip route get 8.8.8.8

Traffic should now have a valid route through the default gateway.


---

Final Diagnosis

Root cause: Missing default route.

Impact: The server could not reach destinations outside its local network.

Evidence:

ip route
→ No default route

ping 8.8.8.8
→ Network is unreachable

ip route get 8.8.8.8
→ Network is unreachable

Fix: Restored the correct default route.

Verification: External connectivity and route lookup succeeded after restoration.


---

Troubleshooting Lesson

When an application is unreachable, don't immediately assume the application is broken.

Think through the path:

Application
     ↓
Server
     ↓
Network interface
     ↓
Routing table
     ↓
Gateway
     ↓
Destination

A server can have a healthy application and a working network interface while still being unable to reach the destination because the routing table is incorrect.
