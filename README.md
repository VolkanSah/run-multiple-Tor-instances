## Multiple Tor Instances for Hidden Services

Tor is cool but sometimes it makes you crazy—sorry folks, we won’t sleep until everything’s up to date!
Run multiple Tor instances with separate configs to host different hidden services on the same machine. Perfect for advanced setups or security testing.

![Tor Logo](https://upload.wikimedia.org/wikipedia/commons/1/15/Tor-logo-2011-flat.svg)

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Setup Instructions](#setup-instructions)

   1. [Install Tor](#1-install-tor)
   2. [Create Config & Data Directories](#2-create-config--data-directories)
   3. [Write Tor Config Files](#3-write-tor-config-files)
   4. [Set Permissions](#4-set-permissions)
   5. [Create Systemd Service Files](#5-create-systemd-service-files)
   6. [Enable and Start Services](#6-enable-and-start-services)
   7. [Verify Services](#7-verify-services)
   8. [Get .onion Addresses](#8-get-onion-addresses)
3. [Configure Apache VirtualHosts](#9-configure-apache-virtualhosts)
4. [Cleanup & Reset](#cleanup--reset)
5. [Ethical Use](#ethical-use)
6. [Support](#support)
7. [License](#license)

---

## Prerequisites

* Tor installed (`sudo apt-get install tor` or equivalent)
* Apache2 (LAMP) installed and running
* Basic Linux command-line knowledge

---

## Setup Instructions

### 1. Install Tor

```bash
sudo apt-get update
sudo apt-get install tor
```

### 2. Create Config & Data Directories

```bash
sudo mkdir -p /etc/tor/instances/hidden_service_1
sudo mkdir -p /etc/tor/instances/hidden_service_2
sudo mkdir -p /var/lib/tor/instances/hidden_service_1/hidden_service
sudo mkdir -p /var/lib/tor/instances/hidden_service_2/hidden_service
```

### 3. Write Tor Config Files

Create `/etc/tor/instances/hidden_service_1/torrc`:

```ini
RunAsDaemon 0
DataDirectory /var/lib/tor/instances/hidden_service_1
PidFile /run/tor/instances/hidden_service_1.pid
SocksPort 0
ExitRelay 0
HiddenServiceDir /var/lib/tor/instances/hidden_service_1/hidden_service/
HiddenServicePort 80 127.0.0.1:9000
Log notice syslog
```

Create `/etc/tor/instances/hidden_service_2/torrc`:

```ini
RunAsDaemon 0
DataDirectory /var/lib/tor/instances/hidden_service_2
PidFile /run/tor/instances/hidden_service_2.pid
SocksPort 0
ExitRelay 0
HiddenServiceDir /var/lib/tor/instances/hidden_service_2/hidden_service/
HiddenServicePort 80 127.0.0.1:9001
Log notice syslog
```

### 4. Set Permissions

```bash
sudo chown -R debian-tor:debian-tor /etc/tor/instances
sudo chown -R debian-tor:debian-tor /var/lib/tor/instances
sudo chmod 700 /var/lib/tor/instances/hidden_service_1/hidden_service
sudo chmod 700 /var/lib/tor/instances/hidden_service_2/hidden_service
```

### 5. Create Systemd Service Files

Create `/etc/systemd/system/tor@hidden_service_1.service` and `/etc/systemd/system/tor@hidden_service_2.service` with:

```ini
[Unit]
Description=Tor Hidden Service %i
After=network.target

[Service]
User=debian-tor
Type=simple
ExecStart=/usr/sbin/tor -f /etc/tor/instances/%i/torrc
Restart=on-failure
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/lib/tor/instances/%i
RuntimeDirectory=tor/instances/%i
RuntimeDirectoryMode=0750

[Install]
WantedBy=multi-user.target
```

### 6. Enable and Start Services

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now tor@hidden_service_1.service
sudo systemctl enable --now tor@hidden_service_2.service
```

### 7. Verify Services

```bash
sudo systemctl status tor@hidden_service_1.service
sudo systemctl status tor@hidden_service_2.service
```

For live logs:

```bash
journalctl -u tor@hidden_service_1.service -f
journalctl -u tor@hidden_service_2.service -f
```

### 8. Get .onion Addresses

```bash
sudo cat /var/lib/tor/instances/hidden_service_1/hidden_service/hostname
sudo cat /var/lib/tor/instances/hidden_service_2/hidden_service/hostname
```

---

## 9. Configure Apache VirtualHosts

1. **Open Ports** in `/etc/apache2/ports.conf`:

   ```
   Listen 9000
   Listen 9001
   ```

2. **Create Directories**:

   ```bash
   sudo mkdir -p /var/www/html/hidden_service_1
   sudo mkdir -p /var/www/html/hidden_service_2
   ```

3. **Create VirtualHost Files**:

   * `/etc/apache2/sites-available/hidden_service_1.conf`:

     ```apache
     <VirtualHost *:9000>
       DocumentRoot /var/www/html/hidden_service_1
       <Directory /var/www/html/hidden_service_1>
         Require all granted
       </Directory>
     </VirtualHost>
     ```
   * `/etc/apache2/sites-available/hidden_service_2.conf`:

     ```apache
     <VirtualHost *:9001>
       DocumentRoot /var/www/html/hidden_service_2
       <Directory /var/www/html/hidden_service_2>
         Require all granted
       </Directory>
     </VirtualHost>
     ```

4. **Enable & Restart Apache**:

```bash
sudo a2ensite hidden_service_1 hidden_service_2
sudo systemctl restart apache2
```

5. **Set Permissions**:

```bash
sudo chown -R www-data:debian-tor /var/www/html/hidden_service_*
sudo chmod -R 750             /var/www/html/hidden_service_*
```

6. **Restart Tor**:

```bash
sudo systemctl restart tor@hidden_service_1
sudo systemctl restart tor@hidden_service_2
```

7. **Test in Tor Browser** by visiting your .onion URLs.

If you encounter errors, check the last 10 lines of the log:

```bash
journalctl -u tor@hidden_service_1.service -n10 --no-pager
```

---

## Cleanup & Reset

If you need to wipe everything and start fresh:

```bash
sudo systemctl stop tor@hidden_service_1.service tor@hidden_service_2.service
sudo systemctl disable tor@hidden_service_1.service tor@hidden_service_2.service
sudo rm -rf /etc/tor/instances/hidden_service_1
sudo rm -rf /etc/tor/instances/hidden_service_2
sudo rm -rf /var/lib/tor/instances/hidden_service_1
sudo rm -rf /var/lib/tor/instances/hidden_service_2
sudo rm /etc/systemd/system/tor@hidden_service_1.service
sudo rm /etc/systemd/system/tor@hidden_service_2.service
sudo systemctl daemon-reload
sudo apt-get purge --auto-remove tor
sudo rm -rf /etc/tor /var/lib/tor /var/log/tor
echo "Cleanup complete. Start fresh whenever you’re ready!"
```

---

## Ethical Use

Use responsibly and legally. No hacking or attacks. Only research, education, or authorized testing.

---

## Support

If this helps, ⭐ the repo and share! More cool stuff here:

* [Volkan Sah GitHub](https://github.com/volkansah)
* [GitHub Sponsors](https://github.com/sponsors/volkansah)

---

## License

MIT License — see LICENSE file.

---

**Credits:** Powered by Batman’s grind and ChatGPT wizardry. 🦇🔥
