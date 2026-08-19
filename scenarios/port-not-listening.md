# Incident 8 — Port Not Listening

## Customer Report

> "The application is running, but I cannot connect to it on port 8080."

---

## Incident Objective

Determine why the expected application port is not accepting connections.

Unlike the previous backend-health incident, the focus here is specifically on the **network port/listener**.

---

# Investigation

## 1. Check Port 8080

### Command

```bash
sudo ss -tulpn | grep :8080

Why?

Checks whether anything is listening on TCP port 8080 and identifies the process using it.

Evidence

No output.

Result

Nothing is listening on port 8080.


---

2. Test the Port Directly

Command

curl -i http://127.0.0.1:8080

Why?

Tests whether an HTTP client can actually establish a connection to the application on port 8080.

Evidence

curl: (7) Failed to connect to 127.0.0.1 port 8080

Result

The application cannot be reached through port 8080.


---

3. Check the Application Process

Command

ps aux | grep '[a]pp.py'

Why?

Determines whether the application process is running.

The important distinction is:

> A process can be running while the expected port is not listening.



Therefore, process state and port state should be checked separately.


---

4. Identify the Port Mismatch

The application was deliberately started on port 9090 instead of its expected port 8080.

Result

Application process → RUNNING
Port 9090            → LISTENING
Port 8080            → NOT LISTENING

This explains why the customer cannot connect to port 8080.


---

Root Cause

The application was running, but it was listening on the wrong port (9090) instead of the expected port (8080).


---

Resolution

Stop the incorrectly configured HTTP server:

pkill -f "HTTPServer"

Start the application on the expected port:

python3 ~/network-troubleshooting-lab/backend/app.py &


---

Verification

Check that port 8080 is now listening:

sudo ss -tulpn | grep :8080

Then test the application:

curl http://127.0.0.1:8080

Expected:

Backend is healthy


---

Final Status

Resolved ✅


---

Troubleshooting Lesson

A running application does not automatically mean the expected port is available.

Always distinguish between:

Process state
     ↓
Is the application running?

Port state
     ↓
Is the application listening on the expected port?

Application state
     ↓
Does it actually respond?

The troubleshooting chain was:

Port 8080
    ↓
ss → no listener
    ↓
curl → connection refused
    ↓
Process → running
    ↓
Check actual listening port
    ↓
9090 → listening
    ↓
8080 → not listening
    ↓
Port mismatch identified
    ↓
Correct application port
    ↓
Verify 8080
