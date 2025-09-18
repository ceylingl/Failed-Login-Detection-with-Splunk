# 🛡️ SIEM Project: Failed Login Detection with Splunk  

## 📌 Overview  
This project simulates brute force login attempts and demonstrates how to detect them using **Splunk**. It includes:  
- Running Splunk Enterprise in Docker  
- Ingesting simulated logs (`failed_logins.log`)  
- Writing SPL queries to detect failed login patterns  
- Building dashboards for visualization  

---

## ⚙️ 1. Setup Environment  

### Install Docker  
- Download and install **Docker Desktop for Mac** from the official website:  
  👉 https://www.docker.com/products/docker-desktop/  
- Once installed, open **Docker.app** from Applications.  
- Verify installation:  
  ```bash
  docker --version
    ```

---

## 🐳 2. Run Splunk in Docker  

Run the following command to start Splunk (works on both Intel and M1/M2 Macs):  

```bash
docker run -d --name splunk   -p 8000:8000 -p 8088:8088   -e SPLUNK_START_ARGS="--accept-license"   -e SPLUNK_PASSWORD=YourPass123   -e SPLUNK_GENERAL_TERMS=--accept-sgt-current-at-splunk-com   --platform linux/amd64   splunk/splunk:latest
```

Check logs to confirm Splunk is running:  
```bash
docker logs -f splunk
```

---

## 🔑 3. Access Splunk  
Open your browser:  
```
http://localhost:8000
```

Login with:  
- Username: `admin`  
- Password: `YourPass123`  

---

## 📂 4. Ingest Test Log Data  

Create a test log file:  
```bash
cat > failed_logins.log <<EOL
2025-09-16 10:00:01 FAILED_LOGIN user=alice src_ip=192.168.1.50
2025-09-16 10:01:15 FAILED_LOGIN user=bob src_ip=203.0.113.12
2025-09-16 10:02:02 FAILED_LOGIN user=alice src_ip=192.168.1.50
2025-09-16 10:03:47 FAILED_LOGIN user=tom src_ip=203.0.113.12
2025-09-16 10:04:21 FAILED_LOGIN user=alice src_ip=192.168.1.50
2025-09-16 10:05:12 FAILED_LOGIN user=charlie src_ip=198.51.100.25
2025-09-16 10:06:33 FAILED_LOGIN user=alice src_ip=192.168.1.50
2025-09-16 10:07:45 FAILED_LOGIN user=bob src_ip=203.0.113.12
2025-09-16 10:08:59 FAILED_LOGIN user=charlie src_ip=198.51.100.25
2025-09-16 10:09:30 FAILED_LOGIN user=alice src_ip=192.168.1.50
EOL
```

In Splunk Web:  
- Go to **Settings > Add Data > Upload File**  
- Upload `failed_logins.log`  
- Create index: `test_logs`  
- Source type: `failed_logins`  

---

## 🔍 5. Query Data with SPL  

Find all failed login events:  
```spl
index=test_logs FAILED_LOGIN
```

Count failed logins per IP:  
```spl
index=test_logs FAILED_LOGIN
| stats count by src_ip
```

Failed logins over time:  
```spl
index=test_logs FAILED_LOGIN
| timechart count by src_ip
```

---

## 📊 6. Create Dashboard  

- Go to **Save as > Dashboard Panel**  
- Dashboard ID: `failed_logins_dashboard`  
- Panels added:  
  - **Top IPs with Failed Logins (Bar Chart)**  
  - **Failed Logins Over Time (Line Chart)**

---

## 🖼️ 7. Export Dashboard as PDF  

Splunk lets you export dashboards for reporting purposes.  

1. Open your dashboard (`Failed Logins Dashboard`).  
2. Click the **Export → Export PDF** in the top-right.

### ⚠️ Known Issue  
When exporting dashboards in Docker-based Splunk, sometimes you may see this error in the PDF:  

```
'bool' object has no attribute 'replace'
```  

This is a **known Splunk bug** with the built-in PDF renderer.  

### ✅ Workarounds  
- **Option 1:** Click the **Export → Print** in the top-right.
- **Option 2:** Use browser print → Save as PDF (captures the dashboard as displayed).  
- **Option 3:** Take screenshots of your panels and include them in your project.  
- **Option 4:** Install Splunk’s **PDF Export App** from Splunkbase (requires setup).  


### Example Screenshots  

<p align="center">
Top IPs Bar Chart <br/>
<img src="https://i.imgur.com/0gabJfk.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

<p align="center">
Failed Logins Over Time Chart <br/>
<img src="https://i.imgur.com/BBFNOJT.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

---

## ✅ Final Result  

The dashboard shows:  
- The **top attacking IPs**  
- The **pattern of failed logins over time**  

---

## 🔍 Findings
- One IP (`192.168.1.50`) showed repeated failed login attempts, indicating brute force activity.  
- Other IPs (`203.0.113.12`, `198.51.100.25`) also contributed but less frequently.  

---

## 🚨 If This Were a Real SOC Case
- Escalate suspicious IP addresses to Threat Intel for validation.  
- Block attacker IPs on the firewall.  
- Investigate targeted user accounts (password resets, MFA enforcement).  
- Create an alert in Splunk to trigger if an IP generates >50 failed logins within 5 minutes.

---

## 🧑‍💻 Key Skills Learned  
- Deploying Splunk using Docker
- Ingesting and indexing log data
- Writing SPL queries (e.g., stats, timechart)
- Building dashboards for security monitoring 
