# Run multiple Tor instances with different configuration files (torrc) to host different hidden services on the same machine. 
Make sure you have the Tor service installed, as described in the previous response.
- Create a new directory for each hidden service configuration:
```bash
sudo mkdir -p /etc/tor/instances/hidden_service_1
sudo mkdir -p /etc/tor/instances/hidden_service_2
```
Replace hidden_service_1 and hidden_service_2 with the desired names for your hidden services.

- Create new configuration files for each hidden service:
```bash
sudo nano /etc/tor/instances/hidden_service_1/torrc
sudo nano /etc/tor/instances/hidden_service_2/torrc
```
- Add the following configuration to each torrc file, adjusting the HiddenServiceDir and HiddenServicePort values as needed:
```bash
RunAsDaemon 1
DataDirectory /var/lib/tor/instances/hidden_service_1
PidFile /var/run/tor/instances/hidden_service_1.pid
SocksPort 0
HiddenServiceDir /var/lib/tor/hidden_service_1/
HiddenServicePort 80 127.0.0.1:8080
```
- Replace hidden_service_1 with the appropriate hidden service name, and adjust the HiddenServicePort value to point to the correct local address and port for the associated web service (e.g., Nginx reverse proxy listening on a different port for each hidden service).

Save the changes and exit the editor.

- Create the required directories and set the correct ownership for the data directories:
```bash
sudo mkdir -p /var/lib/tor/instances/hidden_service_1
sudo mkdir -p /var/lib/tor/instances/hidden_service_2
sudo chown -R debian-tor:debian-tor /var/lib/tor/instances
```
- Create a new systemd service file for each Tor instance:
```bash
sudo nano /etc/systemd/system/tor@hidden_service_1.service
sudo nano /etc/systemd/system/tor@hidden_service_2.service
```
Add the following content to each systemd service file, adjusting the hidden_service_1 and hidden_service_2 values as needed:
```bash
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
Save the changes and exit the editor.

Reload the systemd configuration:
```bash
sudo systemctl daemon-reload
```
Enable and start the new Tor instances:
```bash
sudo systemctl enable tor@hidden_service_1.service
sudo systemctl start tor@hidden_service_1.service
sudo systemctl enable tor@hidden_service_2.service
sudo systemctl start tor@hidden_service_2.service
```
Check the status of the new Tor instances to ensure they are running correctly:
```bash
sudo systemctl status tor@hidden_service_1.service
sudo systemctl status tor@hidden_service_2.service
```
Retrieve the .onion addresses for your hidden services:
```bash
sudo cat /var/lib/tor/hidden_service_1/hostname
sudo cat /var/lib/tor/hidden_service_2/hostname
```

Note the .onion addresses for each hidden service, as you will use them to access the respective services.

Now you have multiple Tor instances running with different torrc files, each hosting a separate hidden service. To access each hidden service, use the Tor Browser and navigate to the corresponding .onion address obtained in step befor
