
# 🔴 Brute Force Attack Simulation & Detection
(Active Directory + Windows Logs + Splunk SIEM)

---

# 🎯 LAB OBJECTIVE

In this lab, we will:

1️⃣ Generate multiple failed login attempts  
2️⃣ Detect Event ID 4625 (Failed Logon)  
3️⃣ Identify repeated failures from same IP  
4️⃣ Detect brute force followed by successful login  
5️⃣ Investigate using Splunk  

---

# 🧩 LAB ENVIRONMENT

| Component | Description |
|------------|-------------|
| Attacker Machine | Your Laptop |
| Target Server | Active Directory Server |
| Monitoring Tool | Splunk SIEM |
| Protocol Used | RDP (Remote Desktop Protocol) |

---

# 🚨 PHASE 1 — ATTACK SIMULATION

## ✅ Step 1 — Perform Failed Login Attempts

From your laptop:

1. Open Remote Desktop (RDP)
2. Connect to AD Server IP
3. Enter:
   - Username: `Administrator`  
   - OR fake user (example: `nadhil`)
4. Enter wrong password
5. Repeat 6–10 times

---

## 🎯 Expected Result

Each failed login generates:

- **Event ID: 4625**
- Log Source: Security
- Failure Reason: Bad Password

📸 —  Failed Login Attempt  


![Failed Login](images/failed_login_4625.png)

---

# 🔍 PHASE 2 — DETECTION IN SPLUNK

## ✅ Detect Failed Logins

```spl
index=windows EventCode=4625
| stats count by Account_Name, Source_Network_Address
| sort - count
```

### 🔎 What This Shows:

- Which account is targeted
- From which IP
- Number of failed attempts

📸  — Splunk Detection of 4625  


![Splunk Failed Logins](images/splunk_4625_detection.png)

---

# 🧠 DETECTION SCENARIO 1  
## Multiple Failed Attempts from Single Source

### 📌 What This Means (Simple Explanation)

If one IP address is trying many passwords repeatedly for the same account, it indicates a **brute force attack**.

### 🚩 Indicators:

- Same `Source_Network_Address`
- Multiple Event ID 4625
- High count in short time

---

# 🔥 DETECTION SCENARIO 2  
## Brute Force Followed by Successful Login

This is more dangerous.

### Attack Flow:

1. Attacker tries wrong password multiple times → Event ID 4625  
2. Eventually enters correct password → Event ID 4624  
3. Gains access to system  

📸  — Successful AD Login  


![Successful Login](images/successful_login_4624.png)

---

## ✅ SPL Query to Detect Brute Force + Success

```spl
index=windows (EventCode=4625 OR EventCode=4624)
| stats count by Account_Name, Source_Network_Address, EventCode
| sort - count
```

### 🔎 What This Helps Identify:

- Failed attempts (4625)
- Successful login (4624)
- Same source IP performing both

---

# 🕵️ SOC ANALYST PERSPECTIVE

When investigating brute force, analyst should check:

✔ Source IP reputation  
✔ Time pattern (rapid attempts?)  
✔ Target account (Administrator?)  
✔ Was login successful after failures?  
✔ Any lateral movement after success?  

---

# 🚨 RISK ASSESSMENT

| Factor | Value |
|---------|--------|
| Attack Type | Brute Force |
| Failed Log Event | 4625 |
| Successful Log Event | 4624 |
| Impact | Unauthorized Access |
| Severity | High (if success occurs) |

---

# 🧾 INVESTIGATION SUMMARY 

A brute force attack was simulated against an Active Directory account using RDP.

Multiple failed login attempts generated **Event ID 4625** in Windows Security logs.

Splunk detected:

- High number of failed attempts
- Same Source IP responsible
- Potential correlation with successful login (Event ID 4624)

This confirms detection capability for brute force attacks in the SOC environment.

---

# 🎯 FINAL OUTCOME

✅ Successfully simulated brute force  
✅ Generated Event ID 4625  
✅ Detected multiple failed logins  
✅ Identified attacking source IP  
✅ Correlated failed + successful login  

---

# 🎓 LEARNING OUTCOME

You now understand:

✔ How brute force attacks work  
✔ What Event ID 4625 means  
✔ What Event ID 4624 means  
✔ How to detect brute force in Splunk  
✔ How SOC analysts investigate authentication attacks  

---

🏁 End of Brute Force Detection Lab
