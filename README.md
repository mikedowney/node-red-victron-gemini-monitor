# Node-RED Victron Gemini Monitor

An automated Node-RED workflow that functions as a virtual chief engineer for marine power systems. It queries the Victron VRM API daily, passes operational telemetry logs to Google's Gemini AI for deep safety and performance analysis, and emails alerts only when anomalies are detected.

## Features
* **Automated Daily Audits:** Runs on a scheduled cron trigger (default: 6:00 AM daily).
* **Multi-Source Telemetry Integration:** Pulls live diagnostics, 24-hour time-series history, and active alarm configurations from Victron VRM.
* **AI-Powered Diagnostics:** Leverages Gemini to evaluate battery health, inverter states, solar array production, and charging coordination.
* **Smart Noise Filtering:** If all parameters are operating within normal engineering tolerances, Gemini returns `NOMINAL` and the flow remains quiet, avoiding notification fatigue.
* **Rich HTML Email Reports:** If an anomaly is detected, it emails a thorough engineering breakdown complete with direct links to your VRM portal dashboard.

---

## Prerequisites & Dependencies
* A running **Node-RED** instance (typically hosted on a Raspberry Pi, marine server, or Cerbo GX Large OS).
* Node-RED Email Node (`node-red-node-email`).
* A **Victron VRM account** with an API Token and Installation ID.
* A **Google AI Studio API Key** (for Gemini access).

---

## Installation & Deployment

1. **Import the Flow:** Copy the contents of `flows.json`, open Node-RED, go to the menu -> Import, and paste the JSON.
2. **Configure Victron API Nodes:** 
   * Update the three HTTP Request nodes (`1. Fetch Diagnostics`, `2. Fetch 24H Stats`, `3. Fetch Alarm Settings`) with your specific Victron `INSTALLATION_ID` in the URL.
   * Add your Victron VRM API token (`Token YOUR_VICTRON_TOKEN_HERE`) into the HTTP header credentials.
3. **Configure Gemini API Node:** 
   * Open the `Build Comprehensive Prompt` function node and replace `YOUR_API_KEY_HERE` with your Google AI Studio API key.
   * Tailor the system baseline rules inside the function node to match your specific hardware setup (house bank capacity, solar array configuration, charger models, etc.).
4. **Configure Email Node:**
   * Double-click the email node and enter your SMTP server settings, user ID, app password, and recipient address.
   * Update the `Format Alert Email` function node with your actual VRM dashboard URL.

---

## Example Email Output

When an anomaly or system alert triggers an email, the report looks like this:

> **⚠️ Power System Engineering Alert**
> 
> **System Status Report (Snapshot: 2026-09-21 06:36:41 AM PDT)**
> 
> As chief engineer and safety officer, I have thoroughly reviewed the provided Victron operational logs. Based on the system profile and baseline rules, the vessel's power systems are currently operating within expected parameters, and no critical anomalies or safety concerns are identified in this snapshot.
> 
> **1. House Bank:** Stable 13.61V, 99.4% SoC, holding steady on shore power float charge.
> **2. Inverter/Charger:** MultiPlus operating correctly in Float mode; no overloads or VE.Bus errors.
> **3. Alternators:** Engines shut down; Orion XS DC-to-DC chargers reporting expected standby states.
> **4. Solar Array:** Inactive as expected during early morning hours while connected to grid power.
> **5. Shore Power & DVCC:** Grid input active and stable; DVCC safety coordination properly enabled.
> 
> **Summary:** All monitored systems are operating normally. No anomalies detected.
> 
> ---
> [Open VRM Portal Dashboard](https://vrm.victronenergy.com/)