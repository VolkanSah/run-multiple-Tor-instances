# Multiple Tor Instances for Hidden Services

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
```

### 2. Create Config & Data Directories

```bash
sudo mkdir -p /etc/tor/instances/hidden_service_1
sudo mkdir -p /etc/tor/instances/hidden_service_2

sudo mkdir -p /var/lib/tor/instances/hidden_service_1
sudo mkdir -p /var/lib/tor/instances/hidden_service_2
```

### 3. Write Tor Config Files

Create `/etc/tor/instances/hidden_service_1/torrc` with:

```ini
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

For each instance, create `/etc/systemd/system/tor@hidden_service_1.service` and `/etc/systemd/system/tor@hidden_service_2.service` with:

```ini
[Unit]
Description=Tor Hidden Service %I
After=network.target

[Service]
User=debian-tor
Type=notify
ExecStart=/usr/sbin/tor -f /etc/tor/instances/%I/torrc
Restart=on-failure
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=yes

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

Check logs for errors:

```bash
journalctl -u tor@hidden_service_1.service -f
journalctl -u tor@hidden_service_2.service -f
```

### 8. Get .onion Addresses

Once running, the onion address is in:

```bash
sudo cat /var/lib/tor/instances/hidden_service_1/hidden_service/hostname
sudo cat /var/lib/tor/instances/hidden_service_2/hidden_service/hostname
```

---

## Ethical Use

Use this setup responsibly and legally. Don’t hack, attack, or annoy anyone. Only for research, learning, or authorized testing.

---

## Support

If this helps you, drop a ⭐ on my GitHub and share!
More of my stuff here: [Volkan Sah GitHub](https://github.com/volkansah)
Support me if you like: [GitHub Sponsors](https://github.com/sponsors/volkansah)

---

## License



---

**Credits:** Built with help from ChatGPT, powered by Batman's eternal hustle.

