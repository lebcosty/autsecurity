1. Web recon — Found  /login.php
2. SQL injection — ' or 1=1-- - bypassed auth; UNION SELECT extracted credentials from DB: admin:Admin@SweetByte!, baker:SweetByte2024!
3. SSH — Logged in as baker via SSH, got user.txt
4. Privesc — Found /opt/backup.sh (world-writable, runs every minute as root). Injected cp /root/root.txt /tmp/rootflag.txt → got root.txt
