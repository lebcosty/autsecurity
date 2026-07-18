Attack Chain

1. SSRF Discovery — PDF generator on port 8080 (exposed to all interfaces) accepts logo_url which requests.get() fetches. Supports file:// protocol — direct file reads as www-data.
2. Admin Panel Path Traversal — Found internal admin panel at 127.0.0.1:5000 (port 8080 exposed via systemd oversight, Bloque 2). JWT secret membrete2019 from source comments. GET /plantillas/previsualizar?archivo=../../../../../../../path&token=<JWT> reads any file as rsolis user via path traversal.
3. Exfiltrated rsolis SSH key — id_rsa extracted despite PDF rendering corruption (reconstructed base64 body).
4. Privilege Escalation to root — membrete-backup.timer runs run_backup.sh as root every 2 minutes with WorkingDirectory=/opt/membrete/backup/. The backup dir is group-writable by membrete group (rsolis is a member). The script executes ./cleanup if present — replaced it with a script that added my SSH key to /root/.ssh/authorized_keys.


SSRF → JWT forge → path traversal → SSH key → backup cleanup → root

Walkthrough

1. **Login** → any user/pass works, access `/factura/generar`
2. **SSRF** → `logo_url=file:///etc/passwd` reads local files
3. **Discover internal admin** → 127.0.0.1:5000 (JWT auth)
4. **Forge JWT** → secret `membrete2019`, role=admin
5. **Path traversal** → `/plantillas/previsualizar?archivo=../../../../../../../path`
6. **Read user flag** → `/home/rsolis/user.txt` → `AUT{REDACTED}`
7. **Exfil SSH key** → `/home/rsolis/.ssh/id_rsa` → SSH as rsolis
8. **Find backup timer** → `membrete-backup.timer` runs as root every 2min from `/opt/membrete/backup/`
9. **Writable dir** → backup dir is group-writable by `membrete` group (rsolis is in it)
10. **Hijack cleanup** → create `./cleanup` with SSH key adder, wait for timer
11. **Root SSH** → `AUT{REDACTED}`
