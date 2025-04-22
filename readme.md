# ChirpStack Network Server Setup and Adding Gateways and Devices 

This document outlines the steps taken to install and configure the ChirpStack Network Server on Ubuntu in an EC2 server. It also describes how to connect the gateways and devices to the Network and integrating the Network Server with Application Server.

--- 

# Prerequisites
    All the ports used need to be open in the aws security group.

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
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00  
# Add ChirpStack repo to APT sources
echo "deb https://artifacts.chirpstack.io/packages/4.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list

# Refresh package index
sudo apt update
```

---

## 4. Installing ChirpStack Gateway Bridge
Install the Gateway Bridge service:

```bash
sudo apt install chirpstack-gateway-bridge     
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

## 7. Verifications if error occurs

- PostgreSQL DSN in `chirpstack.toml` matches your DB user/pass/port:
  ```toml
  dsn = "postgres://chirpstack:chirpstack@localhost:5433/chirpstack?sslmode=disable" 
  ## postgresql's port can be found by: sudo grep port /etc/postgresql/*/main/postgresql.conf
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

## 8. Access ChirpStack Web UI

Open a browser and visit:
```
http://<your-ec2-public-ip>:55555   # the port it's listening to
```

Default login:
- Username: `admin`
- Password: `admin`

---

## 9. Adding Gateway in ChirpStack

1. Navigate to the "Gateways" tab on the left panel.
2. Click "+ Create Gateway".
3. Fill in:
   - Gateway ID (match the ID of your gateway)
   - Gateway name
   - Network Server (usually only one option)
   - Frequency Plan (e.g., `US915_0`)
   - Add location coordinates (optional)
4. Click "Create gateway"

---

## 10. Creating Device Profile

1. Navigate to "Device Profiles"
2. Click "+ Create"
3. Fill in:
   - Name: `Your device type`
   - LoRaWAN MAC Version: `1.0.2` or what your device supports
   - Regional Parameters: e.g., `US_915` and `US915(Channels 0-7 + 64)`
   - Set Default ADR algorithm as ADR algorithm 
   - Expected uplink interval: 10 secs
4. Click "Create device-profile"

---

## 11. Adding a Device

1. Navigate to "Applications" → select or create one
2. Go to "Devices" → Click "+ Create"
3. Fill in:
   - Device name
   - Generate DevEUI and JoinEUI/AppEUI and update them in the code of the end nodes
   - Device profile: select the one just created
4. Click "Create device"
5. After creation, go into the device and go to "OTAA Keys"
6. Generate AppKey and update them in the code of the end nodes

---

Check Live LoRaWAN Frames and Device Data tabs to debug and verify communication.
