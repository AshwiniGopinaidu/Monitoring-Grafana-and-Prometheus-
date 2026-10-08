# EC2 Infrastructure Monitoring Stack

This project automates and documents the deployment of a complete infrastructure monitoring pipeline on an **AWS EC2 Linux (Ubuntu)** instance using **Prometheus**, **Node Exporter**, and **Grafana**.

---

## 🏗️ Architecture Flow
`EC2 Metrics` ➡️ `Node Exporter (Port 9100)` ➡️ `Prometheus Scraper (Port 9090)` ➡️ `Grafana Dashboard (Port 3000)`

---

## 📋 Prerequisites & Security Group Rules

Ensure your AWS EC2 instance has an assigned **Security Group** allowing the following inbound traffic:

| Protocol | Port | Source | Description |
| :--- | :--- | :--- | :--- |
| **TCP** | `22` | Your IP | SSH Admin Access |
| **TCP** | `9100` | `127.0.0.1` or EC2 IP | Node Exporter Metrics |
| **TCP** | `9090` | Public / Private IP | Prometheus Web UI |
| **TCP** | `3000` | Public IP | Grafana Web UI Dashboard |

---

## 🚀 Step-by-Step Deployment Guide

SSH into your Ubuntu EC2 instance and run the following commands sequentially.

### 1. System Update
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Node Exporter (Metrics Agent)
```bash
# Download and extract binary
wget https://github.com
tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz
sudo mv node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/

# Create systemd service
sudo nano /etc/systemd/system/node_exporter.service
```

*Paste the following service configuration inside the file:*
```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

*Enable and start the service:*
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
```

---

### 3. Install and Configure Prometheus (TSDB)
```bash
# Download and extract binary
wget https://github.com
tar -xvf prometheus-2.54.1.linux-amd64.tar.gz
sudo mv prometheus-2.54.1.linux-amd64 /etc/prometheus

# Configure Scrape Targets
sudo nano /etc/prometheus/prometheus.yml
```

*Overwrite or adjust the file to include your node exporter configurations:*
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

*Create systemd service:*
```bash
sudo nano /etc/systemd/system/prometheus.service
```

*Paste the following service configuration:*
```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=root
ExecStart=/etc/prometheus/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/etc/prometheus/data

[Install]
WantedBy=multi-user.target
```

*Enable and start the service:*
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```

---

### 4. Install Grafana Server (Visualization Frontend)
```bash
# Add Grafana GPG Keys and Repositories
sudo apt-get install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://grafana.com | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

# Install, start and enable Grafana
sudo apt update
sudo apt install grafana -y
sudo systemctl enable --now grafana-server
```

---

## ⚙️ Configuration & Integration

### 1. Linking Prometheus Data Source
1. Access your web interface at `http://<YOUR_EC2_PUBLIC_IP>:3000`.
2. Log in with the default credentials: **Username:** `admin` | **Password:** `admin` (Update the password when prompted).
3. Navigate to **Connections** ➡️ **Data Sources** ➡️ **Add Data Source**.
4. Select **Prometheus**.
5. Set the Connection URL to: `http://localhost:9090`.
6. Scroll down and click **Save & Test**.

### 2. Importing the Node Exporter Dashboard
1. Go to **Dashboards** via the left menu navigation sidebar.
2. Click **New** ➡️ **Import**.
3. Under the **Find and import dashboards** text box, enter ID **`1860`** (Official Node Exporter Full Dashboard) and click **Load**.
4. Select **Prometheus** from your data source selection dropdown.
5. Click **Import**.

---

## 📊 Verification Checklist
- [ ] Prometheus Status UI active at `http://<EC2_IP>:9090/targets` showing target states as **UP**.
- [ ] Node exporter generating raw metrics at `http://<EC2_IP>:9100/metrics`.
- [ ] Grafana server processing live system health visual analytics panels (CPU, Disk, RAM).
