
# Multiple Tor Hidden Services Setup Guide

![Tor Logo](https://upload.wikimedia.org/wikipedia/commons/1/15/Tor-logo-2011-flat.svg)

A comprehensive guide to setting up multiple Tor hidden services on a single system using separate configurations.

## Prerequisites

- Linux system (Debian/Ubuntu recommended)
- Tor service installed
- Root/sudo access
- Basic terminal knowledge

## Installation

1. Install Tor:
```bash
sudo apt-get update && sudo apt-get install tor
```

## Configuration

### 1. Directory Setup
Create configuration directories:
```bash
sudo mkdir -p /etc/tor/instances/{hidden_service_1,hidden_service_2}
sudo mkdir -p /var/lib/tor/{hidden_service_1,hidden_service_2}
sudo chown -R debian-tor:debian-tor /var/lib/tor/hidden_service_*
```

### 2. Service Configuration
Create config files for each service:

`/etc/tor/instances/hidden_service_1/torrc`:
```ini
RunAsDaemon 1
DataDirectory /var/lib/tor/hidden_service_1
HiddenServiceDir /var/lib/tor/hidden_service_1/
HiddenServicePort 80 127.0.0.1:8080
SocksPort 0
ExitRelay 0
```

`/etc/tor/instances/hidden_service_2/torrc`:
```ini
RunAsDaemon 1
DataDirectory /var/lib/tor/hidden_service_2
HiddenServiceDir /var/lib/tor/hidden_service_2/
HiddenServicePort 80 127.0.0.1:8081  # Different port than service 1
SocksPort 0
ExitRelay 0
```

### 3. Systemd Service Setup
Create service files (`/etc/systemd/system/tor@.service`):
```ini
[Unit]
Description=Tor Hidden Service %I
After=network.target

[Service]
User=debian-tor
Type=simple
ExecStart=/usr/sbin/tor -f /etc/tor/instances/%I/torrc
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## Starting Services

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now tor@hidden_service_1
sudo systemctl enable --now tor@hidden_service_2
```

## Verification

1. Check service status:
```bash
sudo systemctl status tor@hidden_service_1 tor@hidden_service_2
```

2. View onion addresses:
```bash
sudo cat /var/lib/tor/hidden_service_1/hostname
sudo cat /var/lib/tor/hidden_service_2/hostname
```

## Troubleshooting

### Common Issues

1. **Service fails to start**:
   - Verify permissions: `sudo chown -R debian-tor:debian-tor /var/lib/tor/hidden_service_*`
   - Check logs: `sudo journalctl -u tor@hidden_service_2 -n 50`

2. **Missing hostname files**:
   - Ensure directories exist and are writable
   - Wait 1-2 minutes for Tor to generate addresses

3. **Port conflicts**:
   - Verify no other services are using ports 8080/8081: `sudo ss -tulnp | grep '8080\|8081'`

4. **Configuration testing**:
```bash
sudo -u debian-tor tor -f /etc/tor/instances/hidden_service_2/torrc --verify-config
```

## Ethical Considerations

⚠️ **Important Notice**  
This guide is provided for educational purposes only. Always:
- Obtain proper authorization before testing
- Comply with all applicable laws
- Respect others' privacy
- Use this knowledge responsibly

Unauthorized use of hidden services for malicious purposes is strictly prohibited.

## License

[Creative Commons Zero v1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/)

