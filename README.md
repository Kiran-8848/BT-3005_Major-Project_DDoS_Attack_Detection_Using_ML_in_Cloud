# 🛡️ DDoS Shield — Real-Time Detection & Blocking System

A full-stack DDoS detection system using Machine Learning (3 models), Flask WebSocket dashboard,
IP auto-blocking, and Locust load testing. Built on CIC-IDS-2017 dataset.

---

## 📁 Project Structure

```
ddos-project/
│
├── app.py              ← Flask server + WebSocket (flask-socketio)
├── train_model.py      ← Train 3 ML models, saves best to models/
├── detector.py         ← ML prediction engine (singleton)
├── blocker.py          ← IP blocking (iptables + in-memory fallback)
├── locustfile.py       ← Locust load testing (500 concurrent users)
│
├── templates/
│   └── index.html      ← Dark mode security dashboard
│
├── models/             ← Auto-created by train_model.py
│   ├── model.pkl       ← Best model (auto-selected by F1 score)
│   ├── meta.json       ← Model metadata + all results
│   ├── RandomForest.pkl
│   ├── GradientBoosting.pkl
│   └── SGD_SVM.pkl
│
└── DDoS/               ← Put your CSV dataset files here
    ├── DrDoS_LDAP.csv
    ├── DrDoS_MSSQL.csv
    ├── DrDoS_UDP.csv
    └── Monday-WorkingHours.pcap_ISCX.csv
```

---

## ⚡ Quick Start

### Step 1 — Install Dependencies
```bash
pip install flask flask-socketio scikit-learn pandas numpy joblib locust
```

### Step 2 — Place Dataset
Put your CSV files inside the `DDoS/` folder:
```
DDoS/DrDoS_LDAP.csv
DDoS/DrDoS_MSSQL.csv
DDoS/DrDoS_UDP.csv
DDoS/Monday-WorkingHours.pcap_ISCX.csv
```
> If no CSVs found, synthetic data is used automatically for testing.

### Step 3 — Train Models
```bash
python train_model.py
```
Trains 3 models and saves the best one:
- **RandomForest** — ensemble of 100 decision trees
- **GradientBoosting** — gradient boosted trees
- **SGD/SVM** — linear SVM with SGD optimizer

### Step 4 — Start Dashboard
```bash
python app.py
```
Open browser: **http://localhost:5000**

### Step 5 — Run Load Test (optional)
```bash
# Interactive UI at http://localhost:8089
locust -f locustfile.py --host=http://localhost:5000

# Headless: 500 users, 10/sec spawn rate
locust -f locustfile.py --headless -u 500 -r 10 --host=http://localhost:5000
```

---

## 🖥️ Dashboard Features

| Section | Features |
|---------|---------|
| 📈 Traffic | Real-time Chart.js chart, benign vs attack |
| 🚨 Alerts | Attack alerts, IP block notifications with sound |
| 👥 Live Feed | Per-request log: IP, flow type, prediction, confidence |
| 🤖 ML Prediction | Last prediction, confidence bar, detection rate |
| 🗺️ Map | Real-time IP origin map with attack dots |
| 🚫 Blocked IPs | Table with auto-unblock timer, manual unblock |
| 🤖 ML Models | Compare all 3 models, accuracy/F1 bar chart |
| ⚡ Simulate | Inject attack traffic from dashboard |
| ⚙️ Settings | Sound, auto-block, thresholds |

---

## 🤖 ML Models

### Dataset Features (68)
The model is trained using a combination of datasets from CIC-DDoS2019 and CIC-IDS2017:

*DrDoS_LDAP.csv → LDAP-based DDoS attacks
*DrDoS_MSSQL.csv → MSSQL reflection attacks
*DrDoS_UDP.csv → UDP-based amplification attacks
*Monday-WorkingHours.pcap_ISCX.csv → BENIGN (normal traffic)

### Label Encoding
- `BENIGN` → 0 (normal traffic)
- All DDoS variants → 1 (attack)

### Model Selection
Best model is auto-selected by weighted F1 score and saved as `models/model.pkl`.

---

## 🔧 Auto-Blocking Logic

```
IP sends traffic
     ↓
ML predicts ATTACK
     ↓
Attack streak counter +1
     ↓
streak >= 3 ?
     ↓ YES
Block IP (iptables if root, in-memory otherwise)
Auto-unblock after 5 minutes
```

---

## 📡 WebSocket Events

| Event | Direction | Description |
|-------|-----------|-------------|
| `init` | server→client | Full state on connect |
| `traffic_event` | server→client | Per-flow prediction |
| `traffic_stats` | server→client | Per-second aggregate |
| `alert` | server→client | Attack / block notification |
| `blocked_update` | server→client | Updated blocked list |
| `attack_detected` | server→client | Attack trigger |
| `request_simulate` | client→server | Trigger attack simulation |

---

## 🛠️ Requirements

```
flask>=2.0
flask-socketio>=5.0
scikit-learn>=1.0
pandas>=1.3
numpy>=1.21
joblib>=1.0
locust>=2.0
```

---

## 🔐 iptables Blocking

Runs automatically if script has root privileges:
```bash
sudo python app.py
```
Without root, falls back to in-memory blocking (dashboard shows correctly, no actual firewall rule).

### winget install Python.Python.3.11

###. Remove-Item -Recurse -Force venv

### py -3.11 -m venv venv

### venv\Scripts\activate

### python -m pip install -r requirements.txt
