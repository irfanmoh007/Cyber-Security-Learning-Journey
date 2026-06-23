# Troubleshooting - SOC Automation Lab

Common issues, diagnostics, and resolutions encountered during the lab setup.

## Issue 1: Wazuh Agent Not Connecting
- **Symptom**: Agent shows active but disconnected in dashboard.
- **Resolution**: Check firewall port 1514/1515 and verify key enrollment.

## Issue 2: Shuffle SOAR Workflow Fails to Trigger
- **Symptom**: Webhook calls return 500 error.
- **Resolution**: Verify API keys and json structure formatting.
