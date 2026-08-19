Incident Documentation — 502 Bad Gateway

Incident

Application returned 502 Bad Gateway

Symptom

curl http://app.lab.local

Result:

502 Bad Gateway

Investigation

1. Check Nginx upstream

sudo grep -n "proxy_pass" /etc/nginx/sites-enabled/default

Found:

proxy_pass http://127.0.0.1:9999;

2. Check port 9999

sudo ss -lntp | grep :9999

Result: Nothing listening.

3. Check backend process

ps aux | grep '[a]pp.py'

Found:

python3 .../backend/app.py

4. Find where Python is listening

sudo ss -lntp | grep python

Found:

127.0.0.1:8080

5. Test backend directly

curl -i http://127.0.0.1:8080

Result:

HTTP/1.0 200 OK
Backend is healthy

Root Cause

Port mismatch.

Nginx was forwarding to:

127.0.0.1:9999 ❌

while the healthy backend was listening on:

127.0.0.1:8080 ✅

Therefore:

Client → Nginx ✅ → :9999 ❌ → 502

Fix

Changed:

proxy_pass http://127.0.0.1:9999;

to:

proxy_pass http://127.0.0.1:8080;

Validated:

sudo nginx -t

Result:

syntax is ok
test is successful

Reloaded Nginx:

sudo systemctl reload nginx

Verification

curl http://app.lab.local

Result:

Backend is healthy

Final Conclusion

> The 502 Bad Gateway was caused by Nginx pointing to the wrong backend port. The backend itself was healthy and listening on port 8080.
