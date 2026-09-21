# Node-RED Victron Gemini Power Monitor

An automated Node-RED workflow that acts as a virtual chief engineer for marine electrical systems, conducting an intelligent daily audit of Victron energy components using Gemini AI.

## Overview

Monitoring a complex marine power system (parallel lithium banks, multiple MPPT solar charge controllers, alternators, and inverters) usually requires manually inspecting dense graphs on the Victron VRM portal. 

This workflow automates that process:
1. **Pulls Live Telemetry:** Automatically queries the Victron VRM API every morning for diagnostics, 24-hour time-series stats, and active alarms.
2. **AI-Powered Engineering Audit:** Compiles the logs and runs them through Google's Gemini AI using strict system-specific rules and baseline tolerances.
3. **Smart Filtering:** If everything is operating safely, the AI replies with `NOMINAL` and the flow remains quiet. If an anomaly is detected, it formats a rich HTML email report with direct links to your VRM portal and local dashboards.

---

## How It Works

* **Trigger:** A cron inject node fires every day at 6:00 AM.
* **API Requests:** Sequential HTTP request nodes gather JSON payloads from your Victron VRM installation endpoints.
* **Prompt Assembly:** A JavaScript function node injects current timestamps and raw datasets into a heavily tuned system prompt defining expected baseline behaviors (such as lithium state-of-charge tracking, solar wake-up voltage thresholds, and shore-power oscillations).
* **Evaluation & Delivery:** Gemini evaluates the snapshot. A switch node filters out routine `NOMINAL` responses, while alerts trigger an email node to dispatch structured insights straight to your inbox or satellite link.

---

## Deployment & Setup

### Prerequisites
* A running **Node-RED** instance onboard (e.g., Raspberry Pi, marine server, or Cerbo GX Large OS).
* Node-RED Email node (`node-red-node-email`).
* A Victron VRM account with an API token.
* A Google AI Studio API key for Gemini.

### Configuration Steps
1. Import `flows.json` into your Node-RED workspace.
2. Update the HTTP request nodes with your specific **Victron Installation ID** and **VRM API Token**.
3. Update the Gemini HTTP request node URL with your **Google AI Studio API key**.
4. Configure the Email node with your SMTP server details and recipient address.

---

## Tailoring to Your System Configuration

Because every boat's electrical architecture is unique, you will need to customize the system prompt inside the **"Build Comprehensive Prompt"** function node:
* **House Bank Capacity:** Update battery chemistry, total Amp-hour ratings, and parallel configurations.
* **Solar Layout:** Adjust MPPT controller models, panel wattages, and wiring layouts.
* **Charging Infrastructure:** Modify alternator specifications, DC-to-DC charger limits (like Orion XS units), and shore power configurations.

---

## Example Email Output

When an anomaly or system review triggers an alert, the output email resembles the complete engineering report below:

```text
Vessel Apogee - System Status Report (Snapshot: 2026-09-21 06:36:41 AM PDT)

As chief engineer and safety officer, I have thoroughly reviewed the provided Victron operational logs. Based on the system profile and baseline rules, the vessel's power systems are currently operating within expected parameters, and no critical anomalies or safety concerns are identified in this snapshot.

Here's a detailed breakdown of the analysis:

1. House Bank (1280Ah LiTime smart LiFePO4 bank):
*   Current Status: The main house bank (Apogee SmartShunt [288]) reports a healthy voltage of 13.61 V and a stable State of Charge (SoC) of 99.4%. The current is +2.70 A, indicating a slight charge/maintenance current.
*   24-Hour Trends (DATASET 2): Battery voltage (bv) has been stable between 13.58 V and 13.91 V, and SoC (bs) has remained high, fluctuating between 99.3% and 100%.
*   Alarms: All house bank alarms (Low/High Voltage, Low/High SoC, Low/High Temperature, Cell Imbalance, High Charge/Discharge Current, Internal Failure, Low/High Cell Voltage, High Current) are reported as "No alarm" or "OK".
*   Conclusion: The house bank is performing as expected, holding a stable high charge on shore power. No voltage drops, spikes, or sudden SoC changes are observed.

2. Inverter/Charger (VE.Bus network - MultiPlus 12/3000/120-50 120V [276]):
*   Current Status: The MultiPlus is reporting a "Float" charge state (VE.Bus state (S): Float), which is the normal operational mode when the battery bank is full and connected to shore power. AC input voltage is 124.5 V, and input frequency is 60.31 Hz, consistent with shore power. The DC charge current supplied by the MultiPlus is 9.6 A, with a net 2.7 A going into the battery after accounting for DC loads, which is normal.
*   Alarms: No "Overload" (eO) alarms are active, nor are there any other VE.Bus errors (ERR) reported in this snapshot.
*   Conclusion: The inverter/charger is operating correctly, maintaining the house bank on shore power without any faults or overload conditions.

3. Alternators (Stock Yanmar 125A with two Orion XS 1400 DC-to-DC chargers):
*   Current Status: Both the Port Engine Alternator (Orion XS [290]) and the Stbd Engine Alternator (Orion XS [291]) are in an "Off" state (State (als): Off). The Device off reason (alOR) for both is "Engine shutdown detected", with no active error codes. Input voltages are 13.42 V (Port) and 13.31 V (Stbd), which are normal for engines that are not running.
*   Conclusion: The alternators and DC-to-DC chargers are functioning as expected; they are not active because the engines are shut down. No sustained high input voltage or other anomalies are present.

4. Solar Array & Charge Controllers (3 MPPTs):
*   Current Context: The vessel is currently connected to shore power (AC-Input (AIS): Grid), and the current time is
