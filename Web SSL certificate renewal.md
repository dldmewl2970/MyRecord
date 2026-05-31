# WAS Server SSL Certificate Renewal Guide

> Follow these steps when an SSL certificate expires on a Nginx-based WAS server.

---

## Prerequisites

- VPN connected (local PC must be on the internal network)
- New SSL file (`.zip`) received from the person in charge
- WinSCP installed and server connection details ready

---

## Step 1 — Connect to VPN

Connect to the company VPN from your local PC.

---

## Step 2 — Upload SSL File via WinSCP

Open WinSCP, connect to the server, and upload the new SSL `.zip` file to the following path:

```
Upload path: /home/onerpadm
```

---

## Step 3 — SSH into the Server

Open CMD on your local PC and connect to the server via SSH:

```bash
ssh onerpadm@172.30.8.161
```

Enter the password when prompted.

---

## Step 4 — Switch to Root

```bash
sudo su -
```

---

## Step 5 — Navigate to the Upload Directory

```bash
cd /home/onerpadm
```

---

## Step 6 — Unzip the SSL File

> ⚠️ Always unzip on the server (CMD), NOT on Windows.
> Unzipping on Windows and copying the files may corrupt them.
> Do NOT wrap the new folder name in quotes.

```bash
unzip uploaded_filename.zip -d new_cert_folder_name
```

**Folder name example:**
```
wild.example.co.kr_ssl_20230824
```

---

## Step 7 — Change File Ownership

Change the ownership of the extracted folder to the `onerp` account:

```bash
chown onerp:onerp -R new_cert_folder_name
```

---

## Step 8 — Move the Certificate Folder to the Nginx SSL Path

```bash
mv new_cert_folder_name /etc/nginx/ssl
```

**Example:**
```bash
mv wild.example.co.kr_ssl_20250822 /etc/nginx/ssl
```

---

## Step 9 — Confirm the pem File Path

Navigate into the new certificate folder and check the path:

```bash
cd /etc/nginx/ssl/new_cert_folder_name/Nginx/pem
pwd
```

Copy the path from `/ssl` onwards — you will need it in the next step.

---

## Step 10 — Update the Nginx Config File

Navigate to the Nginx config directory:

```bash
cd /etc/nginx/conf.d
```

Open the config file with vi and update the two SSL lines with the new folder name:

```bash
vi onerpservice.conf
```

**Lines to update (2 lines):**

```nginx
# Domain certificate
ssl_certificate     ssl/new_cert_folder_name/Nginx/pem/cert.pem;

# Private key
ssl_certificate_key ssl/new_cert_folder_name/Nginx/pem/key.pem;
```

> Press `i` to enter edit mode → make changes → press `ESC` → type `:wq` to save and exit.

---

## Step 11 — Test the Nginx Config

```bash
nginx -t
```

This checks whether the config file is valid. You should see `syntax is ok` if everything is correct.

---

## Step 12 — Reload Nginx (Zero Downtime)

```bash
nginx -s reload
```

> `reload` applies the new config without stopping Nginx. Downtime is near zero.

---

## Step 13 — (If reload doesn't work) Restart Nginx

```bash
nginx -s stop
nginx
```

Verify the service is running:

```bash
curl https://your-service-domain
```

---

## Step 14 — Verify the Certificate in the Browser

1. Open the browser in **incognito mode** (to avoid cache issues) and go to the service URL.
2. Click the **padlock icon** next to the address bar.
3. Click **"Connection is secure"**.
4. Click **"Certificate is valid"**.
5. Confirm the expiry date has been updated to the new certificate's date.

---

## Quick Summary

```
Connect VPN
  → Upload zip via WinSCP (/home/onerpadm)
  → SSH into server
  → sudo su -
  → unzip on server (never on Windows)
  → chown onerp:onerp to change ownership
  → mv → /etc/nginx/ssl
  → Edit onerpservice.conf (update 2 lines)
  → nginx -t to validate
  → nginx -s reload
  → Verify certificate date in incognito browser
```
