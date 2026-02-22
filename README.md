# Logger-project

## Abstract
This project aimed to improve and upgrade an existing Raspberry Pi-based logging system used to monitor data from ILOQ S5 door modules. The original system parsed serial data and sent it to
a server but lacked remote connectivity and had stability issues.

The main goals were to add remote access via AWS IoT Core, fix some issues, and improve usability. A Raspberry Pi 4 running Debian Linux was used as the core platform. The system includes
a C#-based serial parser that filters and logs data from the DUT. Key updates to the parser included removing duplicate or irrelevant data via a new filter and adding error handling for device
disconnection.

The project integrated AWS IoT Core for remote SSH access using certificates and Docker containers to isolate the AWS client, ensuring a clean and conflict-free environment. Datadog was
used to collect and display metrics and logs from the Raspberry Pi.

System automation was handled using Linux services and shell scripts. These scripts managed
startup processes, monitored service uptime, and regularly cleared log files using Anacron.

Overall, the project result was a success, remotely accessible logging system with improved stability, filtering and automation, providing ILOQ with a better device to monitor and analyze data.
