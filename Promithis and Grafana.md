# Prometheus & Grafana Setup Guide

## Introduction
This guide covers the installation and setup of **Prometheus** and **Grafana** for monitoring system metrics such as CPU, Memory, and Disk Usage.

---

## 1. Install Prometheus

### **Step 1: Download Prometheus**
```bash
wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-linux-amd64.tar.gz
```

### **Step 2: Extract and Move Files**
```bash
tar xvf prometheus-linux-amd64.tar.gz
sudo mv prometheus-linux-amd64 /etc/prometheus
cd /etc/prometheus
```

### **Step 3: Create Prometheus Service**
```bash
sudo nano /etc/systemd/system/prometheus.service
```
**Paste the following content:**
```ini
[Unit]
Description=Prometheus Monitoring System
After=network.target

[Service]
User=root
ExecStart=/etc/prometheus/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/etc/prometheus/data
Restart=always

[Install]
WantedBy=multi-user.target
```

### **Step 4: Start Prometheus Service**
```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

### **Step 5: Verify Installation**
Check if Prometheus is running on **http://localhost:9090**
```bash
systemctl status prometheus
```

---

## 2. Install Node Exporter (for System Metrics)
```bash
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-linux-amd64.tar.gz
```
```bash
tar xvf node_exporter-linux-amd64.tar.gz
sudo mv node_exporter-linux-amd64/node_exporter /usr/local/bin/
```

### **Create a Node Exporter Service**
```bash
sudo nano /etc/systemd/system/node_exporter.service
```
**Paste the following content:**
```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=root
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

### **Update Prometheus Configuration to Scrape Node Exporter**
```bash
sudo nano /etc/prometheus/prometheus.yml
```
**Add the following under `scrape_configs`:**
```yaml
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```
Restart Prometheus:
```bash
sudo systemctl restart prometheus
```

---

## 3. Install Grafana
```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt install -y grafana
```

### **Start and Enable Grafana**
```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```
Grafana runs on **http://localhost:3000** (default login: `admin/admin`).

---

## 4. Configure Grafana with Prometheus
1. Open **http://localhost:3000** in a browser.
2. Log in with `admin/admin`.
3. Go to **Configuration → Data Sources** → Add **Prometheus**.
4. Set **URL:** `http://localhost:9090`
5. Click **Save & Test**.

---

## 5. Import Grafana Dashboard
Instead of creating a new dashboard, you can **import a pre-built one**:
1. Go to **Grafana** → **Dashboards** → **Import**.
2. Enter **Dashboard ID: `1860`** (Node Exporter Full Dashboard).
3. Click **Load** → Select **Prometheus** as the data source → **Import**.

---

## 6. Verify Metrics
After importing the dashboard, you should see **CPU, Memory, Disk Usage, and Network Traffic** metrics.

---

## 7. Troubleshooting
- **Check Prometheus Logs:**
  ```bash
  sudo journalctl -u prometheus -f
  ```
- **Check Node Exporter Logs:**
  ```bash
  sudo journalctl -u node_exporter -f
  ```
- **Check Grafana Logs:**
  ```bash
  sudo journalctl -u grafana-server -f
  ```

---

## 8. Conclusion
You have successfully set up **Prometheus and Grafana** for system monitoring! 🚀

For alerting, configure **Alertmanager** and set up email notifications.

Let me know if you need further assistance! 😊

