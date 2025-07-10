
# Multiple Tor Instances for Hidden Services
Tor is cool but sometimes it makes you crazy, sorry folks will not sleep till uptodate!

![Tor Logo](https://upload.wikimedia.org/wikipedia/commons/1/15/Tor-logo-2011-flat.svg)

Run multiple Tor instances with separate configs to host different hidden services on the same machine. Perfect for advanced setups or security testing.

## Table of Contents

* [Prerequisites](#prerequisites)
* [Setup Instructions](#setup-instructions)

  * [1. Install Tor](#1-install-tor)
  * [2. Create Config & Data Directories](#2-create-config--data-directories)
  * [3. Write Tor Config Files](#3-write-tor-config-files)
  * [4. Set Permissions](#4-set-permissions)
  * [5. Create Systemd Service Files](#5-create-systemd-service-files)
  * [6. Enable and Start Services](#6-enable-and-start-services)
  * [7. Verify Services](#7-verify-services)
  * [8. Get .onion Addresses](#8-get-onion-addresses)
* [Ethical Use](#ethical-use)
* [Support](#support)
* [License](#license)

---

## Prerequisites

* Tor installed (`sudo apt-get install tor` or equivalent)
* Basic Linux command line knowledge

---

## Setup Instructions

### 1. Install Tor

```bash
sudo apt-get update
sudo apt-get install tor
````

### 2. Create Config & Data Directories

```bash
sudo mkdir -p /etc/tor/instances/hidden_service_1
sudo mkdir -p /etc/tor/instances/hidden_service_2

sudo mkdir -p /var/lib/tor/instances/hidden_service_1/hidden_service
sudo mkdir -p /var/lib/tor/instances/hidden_service_2/hidden_service
```

### 3. Write Tor Config Files

Create `/etc/tor/instances/hidden_service_1/torrc` with:

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

Create `/etc/tor/instances/hidden_service_2/torrc` with:

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
sudo chown -R debian-tor:debian-tor /var/lib/tor/instances
sudo chmod 700 /var/lib/tor/instances/hidden_service_1
sudo chmod 700 /var/lib/tor/instances/hidden_service_2
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

Once running, find your onion URLs here:

```bash
sudo cat /var/lib/tor/instances/hidden_service_1/hidden_service/hostname
sudo cat /var/lib/tor/instances/hidden_service_2/hidden_service/hostname
```

# 9. Setting up 

1. **Ports frei­geben**
   In `/etc/apache2/ports.conf` ergänzen:

   ```
   Listen 9000
   Listen 9001
   ```

2. **VirtualHosts anlegen**
   **/etc/apache2/sites-available/hidden1.conf**
   
   Ordner erstellen mkdir /var/www/html/hiddne_service_1

   ```apache
   <VirtualHost *:9000>
     DocumentRoot /var/www/html/hiddne_service_1
     <Directory /var/www/html/hiddne_service_1>
       Require all granted
     </Directory>
   </VirtualHost>
   ```

   **/etc/apache2/sites-available/hidden2.conf**
   
     Ordner erstellen mkdir /var/www/html/hiddne_service_1

   ```apache
   <VirtualHost *:9001>
     DocumentRoot /var/www/html/hiddne_service_2
     <Directory /var/www/html/hiddne_service_2>
       Require all granted
     </Directory>
   </VirtualHost>
   ```

4. **Aktivieren & neustarten**

   ```bash
   sudo a2ensite hidden1 hidden2
   sudo systemctl restart apache2
   ```

5. **Berechtigungen**

   ```bash
   sudo chown -R www-data:debian-tor /var/www/html/hiddne_service_*
   sudo chmod -R 750               /var/www/html/hiddne_service_*
   ```

6. **Tor neu starten**

   ```bash
   sudo systemctl restart tor@hidden_service_1
   sudo systemctl restart tor@hidden_service_2
   ```

7. **Test**
   .onion im Tor‑Browser öffnen.
   Bei Krach:

   ```bash
   journalctl -u tor@hidden_service_1.service -n10 --no-pager
   ```

   Kopier die Fehlermeldungen hierher – wir knacken das! 🦇





## Oops, you bricked it again?

No worries, it happens to the best of us. Here’s how to clean up your mess and start fresh:

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

echo "Cleanup complete. Now you can start fresh! No? "
echo "ByeBYe TOR "

sudo apt-get purge --auto-remove tor
sudo rm -rf /etc/tor
sudo rm -rf /var/lib/tor
sudo rm -rf /var/log/tor
```

---

## Ethical Use

Use responsibly and legally. No hacking or attacks. Only research, education, or authorized testing.

---

## Support

If this helps, ⭐ the repo and share!
More cool stuff here: [Volkan Sah GitHub](https://github.com/volkansah)
Support me: [GitHub Sponsors](https://github.com/sponsors/volkansah)

---

## License

MIT License — see LICENSE file.

---

**Credits:** Powered by Batman’s grind and ChatGPT wizardry. 🦇🔥




