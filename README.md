# ThingsBoard Dashboard Project

## 📌 Overview
This repository documents the complete process of setting up a **ThingsBoard** dashboard for IoT device monitoring. The setup includes **ThingsBoard, MQTT, SSL (Nginx), Docker, and device integration**.

## 🏗️ System Architecture

### Components:
- **ThingsBoard Server** (Handles data storage and visualization)
- **ThingsBoard Gateway (Docker)** (MQTT integration with ThingsBoard)
- **Nginx (SSL Proxy)** (Secure access via HTTPS)
- **IoT Devices** (Sending data via MQTT)
- **Robustel Gateway** (Edge device for data transfer)

### Data Flow:
1. IoT devices publish telemetry data via **MQTT**.
2. The **ThingsBoard Gateway (Docker)** receives and processes data.
3. **ThingsBoard Server** stores data and provides a UI for visualization.
4. **Nginx** secures access using **SSL (Let's Encrypt)**.
5. The dashboard displays real-time & historical data.

## 🔧 Installation & Configuration

### 1️⃣ Install & Configure ThingsBoard
- Download and install ThingsBoard:
  ```bash
  sudo apt update && sudo apt install thingsboard
  ```
- Edit the configuration file `/etc/thingsboard/conf/thingsboard.yml`:
  ```yaml
  server:
    ssl:
      enabled: true
      key_store: "/etc/letsencrypt/live/delta.acandia.se/keystore.p12"
      key_store_password: "your_password"
  ```
- Start ThingsBoard:
  ```bash
  sudo systemctl start thingsboard
  ```

### 2️⃣ Set Up ThingsBoard Gateway (Docker)
- Create a `docker-compose.yml` file:
  ```yaml
  version: '3.4'
  services:
    tb-gateway:
      image: thingsboard/tb-gateway
      container_name: tb-gateway
      restart: always
      ports:
        - "5000:5000"
      environment:
        - host=10.11.12.4
        - port=8883
        - MQTT_SSL_ENABLED=true
        - MQTT_SSL_CREDENTIALS_TYPE=PEM
        - MQTT_SSL_PEM_CERT=/thingsboard_gateway/certs/fullchain.pem
        - MQTT_SSL_PEM_KEY=/thingsboard_gateway/certs/privkey.pem
      volumes:
        - /etc/letsencrypt/live/delta.acandia.se:/thingsboard_gateway/certs
  ```
- Deploy with Docker:
  ```bash
  docker compose up -d
  ```

### 3️⃣ Configure Nginx as Reverse Proxy (SSL)
- Install **Nginx** and get SSL certificates using Let's Encrypt:
  ```bash
  sudo apt install nginx certbot python3-certbot-nginx
  sudo certbot --nginx -d delta.acandia.se
  ```
- Edit `/etc/nginx/sites-available/thingsboard.conf`:
  ```nginx
  server {
      listen 443 ssl;
      server_name delta.acandia.se;
      ssl_certificate /etc/letsencrypt/live/delta.acandia.se/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/delta.acandia.se/privkey.pem;

      location / {
          proxy_pass http://127.0.0.1:8082/;
          proxy_set_header Host $host;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "Upgrade";
      }
  }
  ```
- Enable and reload Nginx:
  ```bash
  sudo ln -s /etc/nginx/sites-available/thingsboard.conf /etc/nginx/sites-enabled/
  sudo systemctl reload nginx
  ```

## 📊 Dashboard Setup
- Login to ThingsBoard UI (`https://delta.acandia.se`)
- Navigate to **Dashboards** → **Create New Dashboard**
- Add **Telemetry Widgets** (Charts, Tables, Maps, etc.)
- Connect widgets to MQTT telemetry data

## 🛠️ Troubleshooting
### 1️⃣ Check ThingsBoard Logs
```bash
sudo tail -f /var/log/thingsboard/thingsboard.log
```

### 2️⃣ Check Nginx Logs
```bash
sudo tail -f /var/log/nginx/error.log
```

### 3️⃣ Verify WebSockets are Working
```bash
wscat -c "wss://delta.acandia.se/api/ws/plugins/telemetry?token=YOUR_TOKEN"
```

### 4️⃣ Check MQTT Connectivity
```bash
mosquitto_sub -h delta.acandia.se -t 'v1/devices/me/telemetry' -u 'device_token'
```

## 📌 Summary
This guide documents the entire setup for a ThingsBoard dashboard with SSL and MQTT integration. Future improvements include:
- Optimizing performance
- Adding alerts & automation
- Expanding device compatibility
