# Serving Miliano's CIT 386 Page with Nginx

## Result

Nginx serves the page from `/var/www/html/miliano-blarasin/index.html`. A server-local request returned HTTP 200, and the restored page also loaded in a Windows browser through an SSH tunnel at `http://127.0.0.1:18080/miliano-blarasin/`. The browser displayed **Service online**, **Miliano Blarasin**, and **CIT 386: Cloud Network Design — Fall A 2026**.

The course server's live public address and SSH host-key fingerprint are intentionally represented by placeholders in this public repository.

## Command sequence, in order

```text
PS> plink -i VM01_key.ppk azureuser@<course-server-ip>
Using username "azureuser".

azureuser@VM01:~$ pwd
/home/azureuser

azureuser@VM01:~$ dpkg-query -W nginx
nginx   1.24.0-2ubuntu7.18

azureuser@VM01:~$ systemctl is-active nginx
active

azureuser@VM01:~$ sudo apt-get update -qq

azureuser@VM01:~$ sudo DEBIAN_FRONTEND=noninteractive apt-get install -y nginx
nginx is already the newest version (1.24.0-2ubuntu7.18).
0 upgraded, 0 newly installed, 0 to remove and 20 not upgraded.

azureuser@VM01:~$ sudo systemctl start nginx

azureuser@VM01:~$ systemctl is-active nginx
active

azureuser@VM01:~$ sudo install -d -o azureuser -g azureuser -m 0755 /var/www/html/miliano-blarasin

azureuser@VM01:~$ sudo install -o azureuser -g azureuser -m 0644 /home/azureuser/miliano-blarasin/index.html /var/www/html/miliano-blarasin/index.html

azureuser@VM01:~$ stat -c '%A %U %G %s %n' /var/www/html/miliano-blarasin/index.html
-rw-r--r-- azureuser azureuser 742 /var/www/html/miliano-blarasin/index.html

azureuser@VM01:~$ curl -sS -o /dev/null -w 'HTTP_STATUS=%{http_code}\n' http://127.0.0.1/miliano-blarasin/
HTTP_STATUS=200
```

The first service check reported `active`: Nginx was already installed, enabled, and running before the explicit start command. `systemctl start` is idempotent, so running it against an active service left it running. The page was placed only in Miliano's subdirectory; no other student's directory or page was changed.

## Browser verification

The server does not expose TCP port 80 directly through its public network boundary. I therefore opened a local SSH tunnel to the server's loopback port 80:

```powershell
plink -N -L 18080:127.0.0.1:80 -i VM01_key.ppk azureuser@<course-server-ip>
```

With that tunnel open, I loaded this address in the browser:

```text
http://127.0.0.1:18080/miliano-blarasin/
```

The browser title was `Miliano Blarasin | CIT 386`, and the page content matched the committed source below.

## Page source

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Miliano Blarasin | CIT 386</title>
  <style>
    body { max-width: 48rem; margin: 4rem auto; padding: 0 1rem; font-family: system-ui, sans-serif; line-height: 1.6; }
    h1 { color: #8b0000; }
    .status { display: inline-block; padding: .3rem .65rem; border-radius: 999px; background: #e8f5e9; color: #1b5e20; font-weight: 700; }
  </style>
</head>
<body>
  <p class="status">Service online</p>
  <h1>Miliano Blarasin</h1>
  <p>CIT 386: Cloud Network Design — Fall A 2026</p>
  <p>This page is served by Nginx from Miliano's assigned directory on the shared course server.</p>
</body>
</html>
```
