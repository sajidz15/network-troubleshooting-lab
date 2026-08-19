# Incident 1 — DNS Resolution Failure

## Problem

The application `app.lab.local` became inaccessible.

## Customer Report

> "I can't access the application. The application server is reportedly running."

## Symptoms

- `app.lab.local` could not be resolved.
- The application could not be accessed using its hostname.

## Investigation

1. Test hostname resolution

```bash
getent hosts app.lab.local

Result: No output was returned.
This indicated that the hostname was not resolving.

2. Investigate the local hostname configuration
cat /etc/hosts
The expected entry was missing:
172.22.236.124 app.lab.local

Root Cause
The local hostname-to-IP mapping for app.lab.local had been removed from /etc/hosts.

Resolution
Restored the missing mapping:
echo "172.22.236.124 app.lab.local" | sudo tee -a /etc/hosts

Verification
Confirmed hostname resolution:
getent hosts app.lab.local
The hostname resolved to:
172.22.236.124
Then verified application access:
curl http://app.lab.local
The application responded successfully.

Final Status
Resolved ✅
Troubleshooting Lesson
The failure was isolated to hostname resolution rather than the web server itself.
The investigation followed:
DNS → Connectivity → Service → Application
Instead of restarting services unnecessarily, evidence was collected first and the actual root cause was corrected.
