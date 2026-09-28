# Diagnosing Two Nginx Page Failures

Both controlled failures affected only `/var/www/html/miliano-blarasin/`. I recorded the visible symptom and service status before reading the log, changed only Miliano's page file, and restored the working state after each test.

## Failure 1: the index page was unavailable

### Symptom recorded first

The browser displayed **403 Forbidden** for `/miliano-blarasin/`. A server-local request confirmed `HTTP_STATUS=403`. At the same time, the service check still said:

```text
$ systemctl is-active nginx
active
```

Because Nginx was active, restarting the service would not have explained or corrected the page-specific symptom.

### Log command and identifying line

I ran:

```bash
sudo grep -F '/var/www/html/miliano-blarasin/' /var/log/nginx/error.log | tail -n 1
```

The single identifying line was:

```text
2026/09/28 01:16:03 [error] 171736#171736: *4 directory index of "/var/www/html/miliano-blarasin/" is forbidden, client: 127.0.0.1, server: _, request: "GET /miliano-blarasin/ HTTP/1.1", host: "127.0.0.1"
```

### What the line named

The resource was the directory `/var/www/html/miliano-blarasin/`. The reason was that Nginx was asked to serve that directory but could not use an index page there, and directory listing was forbidden. The service itself was healthy; the expected `index.html` resource was unavailable.

### Fix and confirmation

I restored the page filename and requested the URL again:

```bash
sudo mv /var/www/html/miliano-blarasin/index.html.disabled /var/www/html/miliano-blarasin/index.html
curl -sS -o /dev/null -w 'HTTP_STATUS=%{http_code}\n' http://127.0.0.1/miliano-blarasin/
systemctl is-active nginx
```

The results were `HTTP_STATUS=200` and `active`. The page also displayed normally in the browser through the SSH tunnel.

**First check on an unfamiliar machine:** If one directory returns 403 while Nginx is active, I would first confirm that the configured directory contains a readable index file with a name allowed by the Nginx `index` directive.

## Failure 2: Nginx could not read the page

### Symptom recorded first

The browser again displayed **403 Forbidden**, and the local HTTP check returned `HTTP_STATUS=403`. Nginx still reported:

```text
$ systemctl is-active nginx
active
```

The identical browser status did not prove that the cause was identical, so I checked the new log entry rather than repeating the first fix.

### Log command and identifying line

I ran:

```bash
sudo grep -F '/var/www/html/miliano-blarasin/index.html' /var/log/nginx/error.log | tail -n 1
```

The single identifying line was:

```text
2026/09/28 01:16:21 [error] 171735#171735: *6 open() "/var/www/html/miliano-blarasin/index.html" failed (13: Permission denied), client: 127.0.0.1, server: _, request: "GET /miliano-blarasin/ HTTP/1.1", host: "127.0.0.1"
```

### What the line named

The resource was `/var/www/html/miliano-blarasin/index.html`. The reason was operating-system error 13, **Permission denied**: the Nginx worker could find the file but lacked permission to read it.

### Fix and confirmation

I restored normal read permissions and checked both HTTP and the service:

```bash
sudo chmod 0644 /var/www/html/miliano-blarasin/index.html
curl -sS -o /dev/null -w 'HTTP_STATUS=%{http_code}\n' http://127.0.0.1/miliano-blarasin/
systemctl is-active nginx
stat -c '%A %U %G %n' /var/www/html/miliano-blarasin/index.html
```

The results were `HTTP_STATUS=200`, `active`, and:

```text
-rw-r--r-- azureuser azureuser /var/www/html/miliano-blarasin/index.html
```

**First check on an unfamiliar machine:** If the error log says `Permission denied`, I would inspect the file and parent-directory permissions with `namei -l` or `stat` before changing the service or its configuration.
