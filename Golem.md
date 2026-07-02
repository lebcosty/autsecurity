Step-by-step roadmap:

1. Recon — ports 22 (SSH) and 3000 (Node.js Express). curl on port 3000 reveals JSON API with /api/exec endpoint.
2. Cmd exec (restricted) — POST to /api/exec with {"cmd":"id"} returns whitelist: ls, cat, ping, curl, wget, python3, node. Using {"cmd":"ls -la /"} works.
3. User flag — python3 is allowed. Direct cat /home/golem/user.txt via API returns AUT{****************}.
4. SUID discovery — /usr/local/bin/golemctl is setuid-root. strings shows: unsetenv("LD_PRELOAD") → setuid(0) → execve("/usr/bin/python3", ["python3", "/opt/golem/scripts/healthcheck.py"]).
5. Plugin hijack — healthcheck.py imports all .py files from /opt/golem/plugins/ (writable by golem). Write pwn.py with def run(): import os; print(os.popen("cat /root/root.txt").read()).
6. Trigger → Root — Execute golemctl via python3 subprocess → unsets LD_PRELOAD, escalates to root, runs healthcheck.py → loads our plugin → outputs AUT{****************}.


Breaking it down:

- API command injection with whitelisted python3 for arbitrary code execution
- /home/golem/user.txt readable via python3 → user flag
- golemctl SUID binary unsets LD_PRELOAD and escalates to root
- healthcheck.py plugin loader imports arbitrary code from writable /opt/golem/plugins/
- Plugin injected via API → loaded by root process → root flag

Full chain: Recon → API Discovery → CMD Exec (python3) → User Flag → Privilege Escalation → Root Flag
