# Splunk-SSH-Log-Analysis

A security monitoring project using Splunk to analyze SSH authentication activity, detect failed login attempts, and identify potentially malicious behavior such as brute-force attacks.

In this project, I used Splunk to ingest and analyze SSH authentication logs. The objectives were to:
- Upload and index Zeek-style JSON SSH logs
- Detect failed vs. successful SSH authentication attempts
- Identify unusual SSH activity and potential brute-force behavior
- Use SPL queries to extract security insights

## Data Set

Data Source: JSON-formatted Zeek SSH logs (uploaded manually into Splunk).

I used this JSON file: 

**[SSH Log File](./data/ssh_logs.json)**


Since I did not fetch logs through a server endpoint, I accessed the Splunk Web interface through:
```ccp
http://<your-server-ip>:8000
```

I manually downloaded the SSH JSON log file (provided separately) and uploaded it into Splunk using:

Add Data → Upload → Select JSON file → Index: main

## SPL Queries and Findings

Using SPL queries, I listed the top 10 endpoints with failed SSH login attempts:
```spl
index=main sourcetype="_json" auth_success=false
| stats count by "id.orig_h"
| sort -count
| head 10
```

![Top endpoints](./Images/TopEndpoints.png)

Then, I found the total number of SSH connections:
```spl
index=ssh_lab sourcetype="json"
| stats count as total_ssh_connections
```

![Total connections](./Images/TotalSSHConnections.png)


Finall, I counted all of the event types (successful, failed, no-auth, multiple-failed) seen in the logs:
```spl
index=ssh_lab sourcetype="json"
| stats count by event_type
```

![All events](./Images/AllEvents.png)

## Findings
- The dataset contains 1,200 SSH-related events, which provides a sufficient volume to analyze authentication patterns and detect anomalies. This volume suggests consistent SSH activity over the logging period, with a mix of successful, failed, and unauthenticated connection attempts.
- The distribution of failed login attempts shows that there were a small number of IP addresses account for a large share of failures, which is a classic sign of targeted brute-force attempts.
- The high ratio of failed logins to successes strongly suggests password spraying or brute-force attempts

