# Incident 4 — SSH Service Down

## Problem

Users were unable to connect to the Linux server over SSH.

## Customer Report

> "I can't connect to the Linux server over SSH. Please investigate and restore access."

## Investigation

1. Check SSH service status

```bash
systemctl status ssh --no-pager
Result: The SSH service was inactive/stopped.
This indicated that the SSH server was not running.

2. Start the SSH service
sudo systemctl start ssh

3. Verify SSH service health
systemctl status ssh --no-pager
Result: SSH was active and running.

4. Verify SSH port
ss -lntp | grep ':22'
Result: SSH was listening on TCP port 22.

5. Test an actual SSH connection
ssh localhost
The SSH connection was successful after accepting the host key and authenticating with the local user's credentials.
Root Cause
The SSH service was stopped, causing the server to reject SSH connection attempts.

Resolution
The SSH service was started:
sudo systemctl start ssh
The service and port were then verified, followed by a successful SSH connection test.

Verification
systemctl status ssh --no-pager
ss -lntp | grep ':22'
ssh localhost
All verification checks succeeded.

Final Status
Resolved ✅
Troubleshooting Lesson
When an SSH-specific access issue is reported, verify the SSH service before performing unnecessary troubleshooting.
The investigation followed:
SSH Service → Port 22 → Actual SSH Connection → Fix → Verification
A running service, a listening port, and a successful connection are separate verification points.
