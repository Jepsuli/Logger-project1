# Logger project overview


## Abstract
The project aimed to improve and upgrade an existing Raspberry Pi–based logging system that was used to monitor data from iLOQ S5 door modules. The original system parsed serial data and transmitted it to a server; however, it lacked remote connectivity and experienced stability issues.

The primary objectives were to implement remote access via AWS IoT Core, resolve identified issues, and enhance overall usability. A Raspberry Pi 4 Model B running Debian Linux was used as the core platform. The system included a C#-based serial parser that filtered and logged data from the Device Under Test (DUT). An existing and functional C# codebase was utilized as a foundation, and several improvements were implemented. These enhancements included the introduction of a new filtering mechanism to remove duplicate or irrelevant data and the addition of error handling to manage device disconnection scenarios.

The project integrated AWS IoT Core to enable secure remote SSH access using device-specific certificates. Docker containers were used to isolate the AWS client environment, ensuring a clean and conflict-free setup. Additionally, Datadog was implemented to collect and visualize system metrics and log data from the Raspberry Pi.

System automation was managed through Linux system services and shell scripts. These scripts handled startup procedures, monitored service availability, and periodically cleared log files using Anacron to ensure consistent system maintenance.

Overall, the project outcome was a stable and remotely accessible logging system with improved filtering, enhanced reliability, and automated maintenance processes. The upgraded solution provided iLOQ with a more robust device for monitoring and analyzing system data.


## System overview
The system architecture consisted of a Debian Linux server running on the Raspberry Pi 4 Model B. The device hosted the C#-based serial parser application along with several system services and supporting shell scripts. The existing serial parser code was extended and refined with additional features and stability improvements. The DUT, in this case the iLOQ S5 module, communicated directly with the serial parser application.

A Datadog agent was installed on the Raspberry Pi, and the serial parser transmitted collected data to the Datadog server through this agent. The Raspberry Pi used individual certificates obtained from AWS IoT Core to establish secure authentication. Docker was used within the Linux environment to create a secure SSH connection between AWS and the Raspberry Pi, enabling remote access and management.

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
