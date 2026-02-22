# Logger-project


## Abstract
This project aimed to improve and upgrade an existing Raspberry Pi-based logging system used to monitor data from iLOQ S5 door modules. The original system parsed serial data and sent it to
a server but lacked remote connectivity and had stability issues.

The main goals were to add remote access via AWS IoT Core, fix some issues, and improve usability. A Raspberry Pi 4 running Debian Linux was used as the core platform. The system includes
a C#-based serial parser that filters and logs data from the DUT. Key updates to the parser included removing duplicate or irrelevant data via a new filter and adding error handling for device
disconnection.

The project integrated AWS IoT Core for remote SSH access using certificates and Docker containers to isolate the AWS client, ensuring a clean and conflict-free environment. Datadog was used to collect and display metrics and logs from the Raspberry Pi.
System automation was handled using Linux services and shell scripts. These scripts managed startup processes, monitored service uptime, and regularly cleared log files using Anacron.

Overall, the project result was a success, remotely accessible logging system with improved stability, filtering and automation, providing ILOQ with a better device to monitor and analyze data.


## System overview
We had a Debian Linux server on Raspberry Pi 4 Model B, which contained the serial parser code
written with C# and several system services and shell scripts. We used an already existing and
working C# code and fixed and added new features to it. The DUT, which in this case is iLOQ S5
module, works with the serial parser code.

Our Raspberry Pi had a Datadog-agent installed on it and serial parser sent data to Datadog
server through the agent. Raspberry Pi had individual certificates downloaded from AWS Iot Core
and therefore AWS got the SSH remote connection to Raspberry Pi. We used Docker on Linux to
establish a connection between AWS and Raspberry Pi via SSH.

<img width="1881" height="1252" alt="iLOQ Raspberry Pi logger" src="https://github.com/user-attachments/assets/df879898-259d-462e-afb0-87a2c38cb049" />

## System changes
