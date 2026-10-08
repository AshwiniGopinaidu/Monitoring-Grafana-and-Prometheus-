# EC2 Infrastructure Monitoring Stack

This project automates and documents the installation and integration of an automated monitoring environment on an **AWS EC2 Linux (Ubuntu)** instance using **Prometheus**, **Node Exporter**, and **Grafana**.

---

## 🏗️ Architecture Flow
`Linux System Kernel` ➡️ `Node Exporter (9100)` ➡️ `Prometheus Time-Series DB (9090)` ➡️ `Grafana Visualizer (3000)`

---

## 📋 Security Group Configuration Requirements

Ensure your active AWS EC2 instance has an associated **Inbound Security Group Rule** mapping open for the following protocols:

| Protocol | Port | Source Traffic | Purpose Description |
| :--- | :--- | :--- | :--- |
| **TCP** | `22` | Admin IP | Secure SSH Management Terminal |
| **TCP** | `9100` | `127.0.0.1` | Node Exporter Hardware Metrics Stream |
| **TCP** | `9090` | Public/Private IP | Prometheus Metric Scraper UI |
| **TCP** | `3000` | Public Internet | Grafana Web Management Analytics |

---

## 🚀 Execution & Deployment Commands

Connect via SSH to your instance terminal and run the following deployment blocks sequentially:

### 1. System Synchronization and Core Native Installation
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install prometheus prometheus-node-exporter -y
```

### 2. Overwrite Scrape File Configurations (Fixes Indentation and Spacing Errors)
```bash
sudo tee /etc/prometheus/prometheus.yml << 'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
EOF
```

### 3. Configure Verified Grafana Repository Keys and Packages
```bash
sudo rm -f /etc/apt/sources.list.d/grafana.list
sudo mkdir -p /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/grafana.asc https://grafana.com
sudo chmod 644 /etc/apt/keyrings/grafana.asc

echo "deb [signed-by=/etc/apt/keyrings/grafana.asc] https://grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update && sudo apt install grafana -y
```

### 4. Enable Background System Automation Daemons
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus prometheus-node-exporter grafana-server
```

---

## ⚙️ Visual UI Integration Map

### 1. Connect Prometheus to Grafana Engine
1. Launch the web client inside your browser at `http://<YOUR_EC2_PUBLIC_IP>:3000`.
2. Access the console via default administrative authorizations (**User:** `admin` | **Pass:** `admin`).
3. Traverse down to **Connections** ➡️ **Data Sources** ➡️ **Add Data Source**.
4. Choose the **Prometheus** processing database backend option.
5. Map the network entry URL config to: `http://localhost:9090`.
6. Finalize execution by clicking **Save & Test**.

### 2. Provision Pre-built Node Exporter Analytical Metrics
1. Browse to the dashboard explorer grid: **Dashboards** ➡️ **New** ➡️ **Import**.
2. Type **`1860`** inside the dashboard catalog ID block configuration array field and click **Load**.
3. Set your target data source selector path connection object to **Prometheus**.
4. Select **Import** to start tracking live infrastructure performance statistics.
