# NexusDev: CI/CD Pipeline Poisoning by Rishi

## Challenge Metadata
- **Challenge Name**: NexusDev: CI/CD Pipeline Poisoning (by Rishi)
- **Difficulty**: Medium
- **Category**: Web / DevOps / Supply Chain
- **Flags**: 
  - **User**: `HTB{c1cd_p1p3l1n3_p01s0n1ng_1s_r34l}`
  - **Root**: `HTB{d0ck3r_s0ck3t_4bus3_cl4ss1c_pr3v3sc}`
- **Challenge Caption**: 
  > NexusDev has just migrated to a state-of-the-art CI/CD pipeline. However, in their haste to automate everything, they might have left some "legacy" features exposed. Can you infiltrate their supply chain and gain full control over their build server?

## Challenge Overview
This challenge focuses on a supply chain attack via a misconfigured internal artifact store.

### Vulnerabilities
1. **Unauthenticated WebDAV PUT**: The `/artifacts/` directory allows unauthenticated users to upload files via the `PUT` method.
2. **Insecure Pipeline Execution**: A simulated CI/CD cron job downloads and executes scripts from the artifact store without any integrity checks (no checksums or signatures).
3. **Privilege Escalation**: The `devops` user has access to the Docker socket, allowing a container breakout to the host root.

## Setup Instructions
1. Ensure you have Docker installed.
2. Build the challenge:
   ```bash
   docker build -t nexusdev-rishi .
   ```
3. Run the challenge:
   ```bash
   docker run -p 8080:80 -v /var/run/docker.sock:/var/run/docker.sock nexusdev-rishi
   ```
4. Access the web interface at `http://localhost:8080`.

## Solution Walkthrough
Refer to the [walkthrough.md](walkthrough.md) file for detailed exploitation steps.

https://github.com/rikvendra/HTB-NexusDev-CI-CD-Pipeline-Poisoning/blob/main/walkthrough.md
