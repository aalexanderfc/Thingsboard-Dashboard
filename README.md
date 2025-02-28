# **ThingsBoard IoT Dashboard Implementation**
### **Author**: Alexander Flores

---

## **Introduction**
This repository documents the complete process of setting up a ThingsBoard dashboard to visualize IoT sensor data. It includes installation, configuration, troubleshooting, scalability considerations, and potential future improvements.

## System Architecture
The following diagram illustrates the flow of data from IoT devices to the ThingsBoard dashboard:

![System Architecture](images/SystemArchitecture.png)

### **Features:**
- ✅ **ThingsBoard Server** (running natively, not in Docker)
- ✅ **ThingsBoard Gateway (tb-gateway)** (Docker-based)
- ✅ **MQTT communication** for sensor data collection
- ✅ **Secure access with HTTPS & SSL/TLS**
- ✅ **Port forwarding & network setup**
- ✅ **Scalability considerations & future improvements**

---

## **System Architecture**
### **Components**
- **ThingsBoard Server**: Hosts the UI and backend logic.
- **ThingsBoard Gateway (tb-gateway)**: Handles MQTT communication with devices.
- **IoT Devices**: Send temperature and humidity data via MQTT.
- **Nginx Reverse Proxy**: Ensures HTTPS access.
- **Database**: Stores device telemetry.

### **Workflow**
1. IoT devices publish MQTT messages to **tb-gateway**.
2. The gateway forwards data to the **ThingsBoard server**.
3. The **dashboard** visualizes real-time sensor data.
4. **Nginx** ensures secure access via HTTPS.

---

## **Installation and Configuration**

### **1. ThingsBoard Server Setup**
- Installed on a dedicated machine.
- Configured to run on **port 8082**.
- Enabled **SSL** in `thingsboard.conf`.

### **2. ThingsBoard Gateway (Docker)**
#### **docker-compose.yml Configuration**
```yaml
version: '3.4'
services:
  tb-gateway:
    image: thingsboard/tb-gateway
    container_name: tb-gateway
    restart: always
    ports:
      - "5000:5000"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    environment:
      - host=10.11.12.4
      - port=8883  # Secure MQTT connection
      - accessToken=JT4RglYjwJ3LYJMRa31t
      - MQTT_SSL_ENABLED=true
      - MQTT_SSL_CREDENTIALS_TYPE=PEM
      - MQTT_SSL_PEM_CERT=/thingsboard_gateway/certs/fullchain.pem
      - MQTT_SSL_PEM_KEY=/thingsboard_gateway/certs/privkey.pem
    volumes:
      - tb-gw-config:/thingsboard_gateway/config
      - tb-gw-logs:/thingsboard_gateway/logs
      - tb-gw-extensions:/thingsboard_gateway/extensions
      - /etc/letsencrypt/live/delta.acandia.se:/thingsboard_gateway/certs

volumes:
  tb-gw-config:
  tb-gw-logs:
  tb-gw-extensions:
```

### **3. Nginx Reverse Proxy**
#### **nginx.conf (Excerpt)**
```nginx
server {
    listen 443 ssl;
    server_name delta.acandia.se;

    ssl_certificate /etc/letsencrypt/live/delta.acandia.se/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/delta.acandia.se/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8082/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## **Challenges and Troubleshooting**
### **1. Data Not Visible Over HTTPS**
- **Issue**: Data loaded in HTTP (`http://delta.acandia.se`) but not in HTTPS (`https://delta.acandia.se`).
- **Logs Showed**:
  ```
  [error] 6961#6961: *29 connect() failed (111: Connection refused) while connecting to upstream, client: 10.11.12.1, server: delta.acandia.se, request: "GET /api/ws/plugins/telemetry?token= HTTP/1.1"
  ```
- **Solution**:
  - Ensured ThingsBoard was accessible on **port 8082**.
  - Verified **SSL certificate paths** were correctly mapped.
  - Adjusted **Nginx proxy settings** to handle WebSockets (`/api/ws`).
  - Restarted `thingsboard.service` and `nginx.service`.

### **2. MQTT Connection Issues**
- **Issue**: tb-gateway could not connect securely.
- **Solution**:
  - Ensured **tb-gateway** used the same certificates as ThingsBoard.
  - Opened **port 8883** on the firewall and router.

### **3. Devices Not Updating**
- **Issue**: Devices appeared in the UI but were stuck in "loading" state.
- **Solution**:
  - Verified **MQTT logs** (`docker logs tb-gateway`).
  - Ensured devices were sending data to the correct **MQTT topic**.

---

## **Port Forwarding**
For external access, the following port forwarding was configured on the **Robustel Gateway**:

| Protocol | External Port | Internal IP | Internal Port | Purpose |
|----------|--------------|-------------|--------------|----------|
| HTTP     | 80           | 10.11.12.4  | 80           | Redirect to HTTPS |
| HTTPS    | 443          | 10.11.12.4  | 8443         | Secure UI access |
| MQTT     | 1883         | 10.11.12.4  | 1883         | MQTT without SSL |
| MQTT SSL | 8883         | 10.11.12.4  | 8883         | Secure MQTT |

---

## **Scalability and Future Improvements**
### **1. Load Balancing**
- Deploy **multiple ThingsBoard instances** with **HAProxy** or **NGINX load balancing** to distribute traffic efficiently.

### **2. Cloud Hosting**
- Move the setup to **AWS or Azure** for better availability, automated scaling, and redundancy.

### **3. Database Optimization**
- Transition from the default database to **PostgreSQL** with **TimescaleDB** to handle large volumes of time-series data.

### **4. Additional Sensor Integration**
- Expand beyond temperature and humidity to include **CO2, air quality, and motion sensors**.

### **5. AI-Powered Analytics**
- Use **machine learning** models to predict trends and detect anomalies in sensor data.

### **6. Edge Computing**
- Deploy edge processing to reduce latency and make local decisions before sending data to ThingsBoard.

---

## **Use Cases**
This implementation can be applied to various industries:
1. **Smart Agriculture** – Monitor temperature, humidity, and soil moisture.
2. **Industrial IoT** – Track machine health, detect anomalies.
3. **Smart Cities** – Manage environmental monitoring and air quality.
4. **Home Automation** – Automate lighting, heating, and security systems.
5. **Logistics & Cold Chain** – Ensure temperature-sensitive goods remain within required conditions.

---

## **ThingsBoard Dashboard Screenshots**
Here are screenshots showcasing the final dashboard:

### **1. Overview Dashboard**
![THX Dashboard](images/thx_dashboard.png)

### **2. Device Monitoring**
![SDG Dashboard](images/sdg_dashboard.png)

### **3. Available Dashboards**
![Dashboard List](images/dashboard_list.png)

### **4. Home Overview**
![Home Dashboard](images/home_dashboard.png)


---

## **Conclusion**
This documentation provides a complete guide to setting up, securing, and scaling a ThingsBoard IoT dashboard. By following these steps, users can visualize real-time data, troubleshoot common issues, and improve their system for future expansions.

---

