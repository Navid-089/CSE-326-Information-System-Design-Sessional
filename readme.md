# ChirpStack Network Server Setup and Adding Gateways and Devices 

This document outlines the steps taken to install and configure the ChirpStack Network Server on Ubuntu in an EC2 server. It also describes how to connect the gateways and devices to the Network and integrating the Network Server with Application Server.

---

## 1. Installing ChirpStack Requirements
Install the necessary dependencies:

```bash
sudo apt install \
    mosquitto \              # MQTT broker used for communication between components
    mosquitto-clients \      # Command-line tools to test MQTT publish/subscribe
    redis-server \           # In-memory data store for device queues and sessions
    redis-tools \            # CLI tools for managing Redis
    postgresql               # SQL database for ChirpStack data
```

---

## 2. PostgreSQL Setup
Launch PostgreSQL interactive shell:
```bash
sudo -u postgres psql
```

Execute the following SQL queries:
```sql
-- Create role for authentication
create role chirpstack with login password 'chirpstack';

-- Create database
create database chirpstack with owner chirpstack;

-- Change to chirpstack database
\c chirpstack

-- Create pg_trgm extension (used for searching and similarity matching)
create extension pg_trgm;

-- Exit psql
\q
```

---

## 3. Setting up the Software Repository
Install required tools and add ChirpStack package repository:

```bash
sudo apt install apt-transport-https dirmngr                 # Enable APT to fetch over HTTPS
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00  # Add GPG key

# Add ChirpStack repo to APT sources
echo "deb https://artifacts.chirpstack.io/packages/4.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list

# Refresh package index
sudo apt update
```

---

## 4. Installing ChirpStack Gateway Bridge
Install the Gateway Bridge service:

```bash
sudo apt install chirpstack-gateway-bridge      # Gateway Bridge receives LoRa packets from gateways
```

Edit its configuration:
```bash
sudo nano /etc/chirpstack-gateway-bridge/chirpstack-gateway-bridge.toml
```

Update the MQTT integration region and topics:
```toml
[integration.mqtt]
event_topic_template="us915_0/gateway/{{ .GatewayID }}/event/{{ .EventType }}"
command_topic_template="us915_0/gateway/{{ .GatewayID }}/command/#"
```

Start and enable the service:
```bash
sudo systemctl start chirpstack-gateway-bridge   # Start the service now
sudo systemctl enable chirpstack-gateway-bridge  # Start automatically on boot
```

---

## 5. Installing ChirpStack
Install the ChirpStack network server package:

```bash
sudo apt install chirpstack      # Main server that handles device sessions, LoRaWAN logic, etc.
```

Enable and start the ChirpStack service:
```bash
sudo systemctl start chirpstack      # Start ChirpStack network server
sudo systemctl enable chirpstack     # Start it on system boot
```

Check logs to ensure it runs correctly:
```bash
sudo journalctl -f -n 100 -u chirpstack   # Live logs from ChirpStack process
```

---

## 6. Config File Replacements
Replace the following files with your customized versions:

- `/etc/postgresql/16/main/pg_hba.conf` (for DB authentication config)
- `/etc/chirpstack/chirpstack.toml` (main ChirpStack config)

Make sure to verify:

- PostgreSQL DSN in `chirpstack.toml` matches your DB user/pass/port:
  ```toml
  dsn = "postgres://chirpstack:chirpstack@localhost:5433/chirpstack?sslmode=disable"
  ```
- Region name in `[network]` matches enabled gateway region:
  ```toml
  enabled_regions = ["us915_0"]
  ```
- MQTT server is accessible and set to:
  ```toml
  server="tcp://localhost:1883"
  ```

After updates, restart services:
```bash
sudo systemctl restart chirpstack
sudo systemctl restart chirpstack-gateway-bridge
```

---

