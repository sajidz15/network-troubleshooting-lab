Incident: HTTPS / TLS Failure

Scenario

Customer report:
The application is reachable over HTTP, but HTTPS is not working.


---

Initial Symptoms

HTTP worked:

curl -I http://127.0.0.1

HTTPS failed:

curl -k -I https://127.0.0.1

Output:

curl: (7) Failed to connect to 127.0.0.1 port 443


---

Investigation

1. Check whether anything is listening on HTTPS port 443

sudo ss -tulpn | grep :443

Result: Nothing appeared.

This means nothing was listening on port 443.


---

2. Check NGINX configuration

sudo nginx -T 2>/dev/null | grep -n "listen.*443"

The output showed:

# listen 443 ssl;
# listen [::]:443 ssl;

The # meant the HTTPS listener was commented out.


---

3. Enable HTTPS listener

Edit the NGINX configuration:

sudo nano /etc/nginx/sites-available/default

Enable:

listen 443 ssl default_server;
listen [::]:443 ssl default_server;


---

4. Test NGINX configuration

sudo nginx -t

Initially, NGINX reported a certificate problem:

cannot load certificate
"/etc/ssl/certs/ssl-cert-snakeoil.pem":
No such file or directory

The HTTPS configuration required a certificate that did not exist.


---

5. Create a self-signed certificate

Because the lab machine also had a routing problem at that point, installing the certificate package with apt failed.

Instead, generate a certificate locally:

sudo openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout /etc/ssl/private/ssl-cert-snakeoil.key \
-out /etc/ssl/certs/ssl-cert-snakeoil.pem \
-subj "/CN=localhost"


---

6. Validate NGINX again

sudo nginx -t

Result:

syntax is ok
test is successful


---

7. Restart NGINX

sudo systemctl restart nginx


---

8. Verify port 443

sudo ss -tulpn | grep :443

Port 443 was now listening.


---

9. Test HTTPS

curl -k -I https://127.0.0.1

HTTPS successfully responded.

-k is used because the lab uses a self-signed certificate, which curl does not normally trust.


---

Root Cause

HTTPS was unavailable because:

1. NGINX's listen 443 ssl configuration was disabled.


2. The required SSL certificate was missing.



Therefore, NGINX could not provide HTTPS on port 443.


---

Resolution

Enabled NGINX HTTPS listener on port 443.

Created the required self-signed certificate and private key.

Validated the NGINX configuration.

Restarted NGINX.

Verified port 443 was listening.

Confirmed HTTPS connectivity with curl.



---

Troubleshooting Flow

HTTPS fails
     ↓
Check port 443
     ↓
Nothing listening
     ↓
Inspect NGINX configuration
     ↓
listen 443 was commented out
     ↓
Enable HTTPS listener
     ↓
nginx -t
     ↓
Certificate missing
     ↓
Create certificate
     ↓
nginx -t succeeds
     ↓
Restart NGINX
     ↓
Verify :443
     ↓
curl HTTPS
     ↓
RESOLVED

Key Commands

sudo ss -tulpn | grep :443
sudo nginx -T
sudo nginx -t
sudo systemctl restart nginx
curl -k -I https://127.0.0.1
