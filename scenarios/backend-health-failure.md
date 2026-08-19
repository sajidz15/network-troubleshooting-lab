
# Incident 7 — Backend Application Down

## Customer Report

> "The application isn't working."

---

## Incident Objective

Troubleshoot the application layer by layer without assuming the root cause.

The goal is to identify whether the failure is caused by:

- DNS
- Network connectivity
- Frontend service
- Nginx configuration
- Backend connectivity
- Backend application

---

# Investigation

## 1. Check DNS Resolution

### Command

```bash
getent hosts app.lab.local

Why?

Checks whether the application hostname resolves to an IP address.

Evidence

app.lab.local → 172.22.236.124

Result

DNS is working. ✅


---

2. Check Network Connectivity

Command

ping -c 4 172.22.236.124

Why?

Checks whether the server is reachable over the network.

Evidence

0% packet loss

Result

Network connectivity is working. ✅


---

3. Check Which Service Uses Port 80

Command

sudo ss -tulpn | grep :80

Why?

Shows whether anything is listening on port 80 and identifies the process using it.

Result

Nginx was listening on port 80.

Frontend port is healthy. ✅


---

4. Check Nginx Service

Command

sudo systemctl status nginx --no-pager

Why?

Confirms whether the Nginx service is running.

Result

Nginx was active and running.

Nginx is healthy. ✅


---

5. Test the Application Through Nginx

Command

curl -i http://app.lab.local

Why?

Tests the actual HTTP application path instead of only checking whether the server is reachable.


---

Strong Hypothesis

At this point:

DNS                 ✅
Network             ✅
Nginx                ✅

The next logical layer is the upstream/backend.

However, we should not assume that port 8080 is the backend.

> Port numbers do not automatically tell us what service is running.



We need evidence.


---

6. Check Port 8080

Command

sudo ss -tulpn | grep :8080

Why?

Checks whether anything is listening on port 8080 and, if so, which process owns the port.

Evidence

No output.

Result

Nothing is listening on port 8080. ❌

This is evidence that the expected upstream endpoint is unavailable, but it does not yet prove why.


---

7. Check Nginx Configuration

Command

sudo nginx -T 2>/dev/null | grep -i proxy_pass

Why?

We need to know where Nginx is configured to send application requests.

Evidence

proxy_pass http://127.0.0.1:8080;

Result

Nginx is configured to forward requests to:

127.0.0.1:8080

Now we have a strong connection between the frontend and the unavailable port.


---

8. Check the Backend Process

Command

ps aux | grep '[a]pp.py'

Why?

Checks whether the backend application process is actually running.

Evidence

No output.

Result

Backend process is not running. ❌


---

Root Cause

The backend application process was down.

Nginx itself was healthy and correctly configured to proxy requests to:

127.0.0.1:8080

However, no process was listening on that port because the backend application had stopped.

Root Cause Chain

Customer
   ↓
DNS                         ✅
   ↓
Network                     ✅
   ↓
Nginx :80                   ✅
   ↓
Nginx proxy_pass            ✅
   ↓
127.0.0.1:8080              ❌
   ↓
Backend process              ❌ DOWN


---

Resolution

Start the backend application:

python3 ~/network-troubleshooting-lab/backend/app.py &

Verify the backend port

sudo ss -tulpn | grep :8080

Expected: Python listening on 127.0.0.1:8080.

Verify the backend directly

curl http://127.0.0.1:8080

Expected:

Backend is healthy

Verify the complete customer-facing path

curl http://app.lab.local

Expected:

Backend is healthy


---

Final Status

Resolved ✅


---

Troubleshooting Lesson

Do not jump from a symptom directly to a root cause.

The correct troubleshooting flow was:

Customer symptom
      ↓
DNS
      ↓
Network connectivity
      ↓
Frontend service
      ↓
Frontend configuration
      ↓
Upstream port
      ↓
Backend process
      ↓
Confirm root cause
      ↓
Fix
      ↓
End-to-end verification
