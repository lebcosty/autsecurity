Attack Chain Summary

1. Recon — Nmap found ports 22 (SSH) and 80 (HTTP) running nginx with "CipherState" web app
2. Web Enumeration — Discovered /encrypt and /decrypt endpoints; the decrypt endpoint base64-decodes input and renders it through render_template_string
3. SSTI Exploitation — {{7*7}} → 49 confirmed Jinja2 SSTI in Flask. Used {{ lipsum.__globals__['os'].popen('cmd').read() }} for RCE as www-data
4. Credential Discovery — Read /opt/cipherstate/app.py which contained SSH credentials: operator:operator1
5. SSH Access — Logged in as operator and read user.txt at /home/operator/user.txt
6. Privilege Escalation — Found /usr/bin/python3.12 had cap_setuid=ep capability. Used os.setuid(0) to escalate to root and read root.txt at /root/root.txt
