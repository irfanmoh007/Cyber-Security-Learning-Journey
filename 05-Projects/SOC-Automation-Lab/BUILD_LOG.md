# Build Log - SOC Automation Lab

This document logs the step-by-step installation, configuration, and setup of the SOC Automation Lab.

## Step 1: Endpoint Logging (Sysmon Setup)
- Installed Sysmon on target endpoints.
- Configured local logs using `configs/sysmonconfig.xml`.

## Step 2: SIEM Deployment (Wazuh Setup)
- Configured Wazuh Manager rules in `configs/local_rules.xml`.
- Configured agent settings in `configs/ossec.conf`.

## Step 3: SOAR Automation (Shuffle Integration)
- Setup webhook triggers and integrations in `configs/shuffle-workflow.json`.

## Step 4: Case Management (TheHive Setup)
- Integrated alert feeds with incident tickets.
