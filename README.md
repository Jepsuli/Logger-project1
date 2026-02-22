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

## System automation
### Automation services
On the Raspberry Pi, several Linux services were used to automate system functionality. At startup, dedicated services launched both the serial parser and the AWS connection. The AWS service executed a Docker command and included a timer that started the service one minute after boot. The serial parser was configured as a system service that executed a shell script to launch the parser application, ensuring that it became operational immediately when the system started.

A separate shell script was created to copy the required certificates and configuration files from the /boot/firmware directory, where they were initially placed. The script transferred the files to their designated directories and set the appropriate permissions. A corresponding system service was configured to execute this script once during the first boot.

Additionally, a monitoring system service was implemented to verify that both the AWS service and the serial parser were running. This service executed a helper shell script that checked the status of the services. If either service was not running, the script restarted it and recorded a message in a dedicated helper log file. The log file contained timestamped status messages indicating whether the services were running correctly. The monitoring service was controlled by a system timer that executed it hourly.

Finally, a maintenance shell script was developed to clear the log files of both the serial parser and the helper service. This script was scheduled using Anacron, a Linux task scheduler that ensured execution even if the system was powered off at the scheduled time. In this implementation, Anacron executed the script weekly. The script cleared both log files and recorded a timestamped message stating “log files cleared.”

### Other shell scripts and services
A system reboot counter message was also added to the same shell script that started the serial parser. The implementation was simple: the script used an echo command to print the message “Raspberry started reboot #count” into the serial parser’s log file. Each time the system rebooted, the script counted all previously recorded reboot messages from the log file and incremented the counter accordingly.

Another script was developed to analyze all messages in the log file, including the “adapter opened” entry, which indicated a reboot of the lock module. The script searched the log file for the correct serial number of the lock device by identifying the string “Main 540” and extracting the number that followed it. If the expected message had not yet appeared in the log file, the script waited until it was detected and then printed the device reboot message along with the updated counter value.

This script was configured to run as a system service that continuously monitored the log file for the “adapter opened” message. However, in the final implementation, the service was not left active because it was not compatible with the RAM disk configuration.
