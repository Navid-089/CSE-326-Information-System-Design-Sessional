# ChirpStack Network Server Setup 

This document outlines the steps taken to install and configure the ChirpStack Network Server on Ubuntu.

---

## 1. Installing ChirpStack Requirements
Install the necessary dependencies:

```bash
sudo apt install \
    mosquitto \
    mosquitto-clients \
    redis-server \
    redis-tools \
    postgresql
```

---

## 2. PostgreSQL Setup
Launch PostgreSQL:
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

-- Create pg_trgm extension
create extension pg_trgm;

-- Exit psql
\q
```

---

## 3. Setting up the Software Repository
Install repository tools and add ChirpStack package repository:
```bash
sudo apt install apt-transport-https dirmngr
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00

echo "deb https://artifacts.chirpstack.io/packages/4.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list

sudo apt update
```

---

## 4. Installing ChirpStack Gateway Bridge
Install the gateway bridge:
```bash
sudo apt install chirpstack-gateway-bridge
```

Edit the configuration:
```bash
sudo nano /etc/chirpstack-gateway-bridge/chirpstack-gateway-bridge.toml
```

Update the `[integration.mqtt]` section:
```toml
[integration.mqtt]
event_topic_template="us915_0/gateway/{{ .GatewayID }}/event/{{ .EventType }}"
command_topic_template="us915_0/gateway/{{ .GatewayID }}/command/#"
```

Enable and start the service:
```bash
sudo systemctl start chirpstack-gateway-bridge
sudo systemctl enable chirpstack-gateway-bridge
```

---

## 5. Installing ChirpStack
Install the ChirpStack network server:
```bash
sudo apt install chirpstack
```

Enable and start the service:
```bash
sudo systemctl start chirpstack
sudo systemctl enable chirpstack
```

View logs:
```bash
sudo journalctl -f -n 100 -u chirpstack
```

---

## 6. Config File Replacements
Replace the following files with the versions from your repository:

- `/etc/postgresql/16/main/pg_hba.conf`
- `/etc/chirpstack/chirpstack.toml`

Ensure port numbers and database credentials match your setup.

---

> ✅ ChirpStack Network Server is now installed and configured.

Continue with gateway and device setup as required.
