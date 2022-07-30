# Blue Team: Summary of Operations

## Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior
- Suggestions for Going Further

### Network Topology


The following machines were identified on the network:

- Name of VM 1: Hyper V Host Manager
	- **Operating System**: Windows 10
	- **Purpose**: Contains the vulnerable machines and the attacking machine
	- **IP Address**: 192.168.1.1
- Name of VM 2: Kali
	- **Operating System**: Linux 5.4.0
	- **Purpose**: Used as attacking machine
	- **IP Address**: 192.168.1.90
- Name of VM 3: Capstone
	- **Operating System**: Linux (Ubuntu 18.04.1 LTS)
	- **Purpose**: Used as a testing system for alerts
	- **IP Address**: 192.168.1.100
- Name of VM 4: ELK
	- **Operating System**: Linux (Ubuntu 18.04.1 LTS)
	- **Purpose**: Used for gathering information from the victim machine using Metricbeat, Filebeats, and Packetbeats
	- **IP Address**: 192.168.1.100
- Name of VM 5: Target 1
	- **Operating System**: Linux 3.2 - 4.9
	- **Purpose**: The VM with WordPress as a vulnerable server
	- **IP Address**: 192.168.1.110
- Name of VM 6: Target 2
	- **Operating System**: Linux 3.2 - 4.9
	- **Purpose**: The VM with WordPress as a vulnerable server
	- **IP Address**: 192.168.1.115

### Description of Targets

The target of this attack was: `Target 1` **IP Address**: 192.168.1.100

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### Monitoring the Targets

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

#### Excessive HTTP Errors
- Alert 1 is implemented as follows:
  - **Metric**: `Packetbeat:` http.response.status_code > 400
  - **Threshold**: When count() GROUPED OVER top5 ‘http.response.status_code’ is above 400 for the last 5 minutes
  - **Vulnerability Mitigated**: Used intrusion detection/prevention for attacks | 
IPS would block any suspicious IP’s | 
Utilize Account Management to lock or request user accounts to change the passwords every 60 days |
Filter and disable or close port 22
  - **Reliability**: This alert will not generate an excessive amount of false positives identifying brute force attacks. `Medium`

#### CPU Usage Monitor
- CPU Usage Monitor is implemented as follows:
  - **Metric**: `Metricbeat:` system.process.cpu.total.pct
  - **Threshold**: WHEN max() OF system.process.cpu.total.pct OVER all documents IS ABOVE 0.5 FOR THE LAST 5 minutes
  - **Vulnerability Mitigated**: Controlling the CPU usage percentage at 50%, it will trigger a memory alert only if the CPU remains at or above 50% consistently for 5 minutes. Virus or Malware
  - **Reliability**: Yes, this alert can generate a lot of false positives due to CPU spikes occurring when specific integrations are initiated at the start of processing. `High`

#### HTTP Request Size Monitor
- HTTP Request Size Monitor is implemented as follows:
  - **Metric**: `Packetbeat:` http.request.bytes
  - **Threshold**: When sum() of http.request.bytes OVER all documents is ABOVE 3500 for the LAST 1 minute
  - **Vulnerability Mitigated**: By controlling the number of http request sizes through a filter, protection is enabled to detect or prevent DDOS attacks for IPS/IDS.
  - **Reliability**: No, this alert doesn't generate an excessive amount of false positives because DDOS attacks submit requests within seconds, not within minutes. `Medium` 