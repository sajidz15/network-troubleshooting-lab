# Incident 6 — High Latency Failure

## Problem

The customer reported that the application was extremely slow and requests were taking much longer than normal.

## Customer Report

> "The application is extremely slow. It eventually responds, but every request is taking much longer than normal."

## Investigation

### 1. Check application latency

```bash
ping -c 4 app.lab.local

The normal response time was very low.

2. Check HTTP response time

curl -o /dev/null -s -w '%{time_total}\n' http://app.lab.local

The HTTP response completed successfully.

3. Check traffic-control configuration

sudo tc qdisc show dev eth0

Output showed:

netem ... delay 500ms

A 500 ms artificial network delay had been configured on eth0.

Root Cause

A 500 ms network delay was intentionally introduced on the network interface, causing a high-latency condition in the lab.

Resolution

Remove the artificial delay:

sudo tc qdisc del dev eth0 root

Verify:

sudo tc qdisc show dev eth0

Then verify the application:

curl http://app.lab.local

The application responded normally after the delay was removed.

Final Status

Resolved ✅

Troubleshooting Lesson

When an application is slow, determine whether the delay is coming from the application or the network.

Useful checks include:

ping
curl
tc qdisc show

The important troubleshooting pattern is:

Symptom → measure latency → identify network condition → remove the cause → verify recovery.
