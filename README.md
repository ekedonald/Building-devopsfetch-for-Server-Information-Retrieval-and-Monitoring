# Building `devopsfetch` for Server Information Retrieval and Monitoring

## Objective
 Developing a tool for devops named `devopsfetch` that collects and displays System Information, Active Ports, User Logins, Nginx Configurations, Docker Images and Container Statuses. The tool is also implemented as a `systemd` service to Monitor and Log these activities continuously.

## Features
### Information Retrieval
**1. Ports**
   - Displays all active ports and services (`-p`or `--port`).
   - Provides detailed information about a specific port (`-p` `<port_number>`).

**2. Docker**
   - Lists all Docker images and containers (`-d` or `--docker`).
   - Provides detailed information about a specific container (`-d` `<container_name>`).

**3. Nginx**
   - Displays all Nginx domains and their ports (`-n` or `--nginx`).
   - Provides detailed configuration information for a specific domain (`-n` `<domain>`).

**4. Users**
   - List all users and their last login times (`-u` or `--users`).
   - Provide detailed information about a specific user (`-u` `<username>`).

**5. Time Range**
   - Provides detailed information about activities within the specified time range (`-t` `<start_time>` `<end_time>`).
   
**6. Help**
   - Provides usage examples for each command-line flag (`-h` or `--help`).

## Implementing The `devopsfetch` Tool
The [devopsfetch](devopsfetch) tool was created using a Bash Script. It was designed to provide a convenient way of retrieving, displaying system information related to networking, Docker, Nginx, users and system activity logs. The script is divided divided 3 main functions which will be discussed below:

### 1. Essential Functions

**`log_output()`**: This function captures the script's output and appends it to a log file `/var/log/devopsfetch.log`. This is vital for troubleshooting and logging.

**`display_help()`**: This function displays the help message for the `devopsfetch` tool. It provides usage examples for each command-line flag.

**`format_table()`**: This function formats the output of the `devopsfetch` tool as a table. It takes the output of the `devopsfetch` tool as input and returns a user-friendly tabular format.

### 2. Core Functionality Functions

**`display_ports()`**: This displays information about all the active network ports using `ss`, `lsof` and `awk` commands.
- If no argument is passed alongside `-p`, it list all the active ports, their corresponding user and associated services.

- If an argument is passed alongside `-p`, it provides detailed information about a specific port.

**`display_docker()`**: This displays information about all the Docker images and containers using `docker images`, `docker ps` and `docker inspect`.
- If no argument is passed alongside `-d`, it lists all Docker images and containers.

- If a `container_name` is passed alongside `-d`, it provides detailed information (_i.e. Container Name, Image, Status, IP Address and Ports_) about the specified container.

**`display_nginx()`**: It retrieves information about Nginx domains and configuration in the `/etc/nginx/sites-enabled` and `/etc/nginx/conf.d` directories.
- If no argument is passed alongside `-n`, it lists all Nginx domains and their ports.

- If a `domain name` is passed alongside `-n`, it displays the Domain, Port and URL it is proxying to.

**`display_users()`**: It provides information about system users using `id`, `last` and `w` commands.
- If no argument is passed alongside `-u`, it list all users with their last login time.

- If a `user_name` is passed alongside `-u`, detailed information about the specified user is displayed (_i.e. current login status, last login time and resource usage details_).

**`display_time_range()`**: It displays system activities logged by `journalctl`.
- If a single date is provides `-t 2024-07-23`, it displays all the log entries on the specified date.

- If a range of dates is provided `-t 2024-07-23 2024-07-25`, it displays all the log entries within the specified date range.

### 3. Script Execution
The `case` statement is used to handle user input. It identifies the chosen option and calls the corresponding function to process the information and displays it.

#### Examples
- `devopsfetch -p`: Lists all active ports and their associated services.

- `devopsfetch -d`: List all Docker images and containers.

- `devopsfetch -d container_name`: Displays detailed information about the specified Docker container.
- `devopsfetch -u`: Lists all users and last login.

- `devopsfetch -u user_name`: Displays detailed information about the specified user.

- `devopsfetch -n`: Lists all Nginx domains and their ports.

- `devopsfetch -n domain_name`: Lists the domain name, port it is listening on and URL it is proxying to.

- `devopsfetch -t 2024-07-22`: Displays system activity logs for July 22nd, 2024.

- `devopsfetch -t 2024-07-18 2024-07-23`: Displays system activity logs for July 18th, 2024 to July 23rd, 2024.


## Installing and Configuring `devopsfetch` as an Executable and Monitoring Service

The [install_devopsfetch.sh](./install_devopsfetch.sh) script installs and configures [devopsfetch](./devopsfetch) as an Executable and Service. Here's a breakdown of the script:

### 1. Root Privileges
**`if [ "$EUID" -ne 0 ]`**: The script checks if the user has root privileges. If not, it exits with an error message.

### 2. Copy and Configure devopsfetch Script
**`cp devopsfetch /usr/local/bin/devopsfetch`**: Copies the main devopsfetch script from the current directory to `/usr/local/bin`. This makes it an **Executable**.

**`chmod +x /usr/local/bin/devopsfetch`**: Sets executable permission for devopsfetch.

### 3. Set Up Log File

**`touch /var/log/devopsfetch.log`**: Creates a log file for the service in /var/log.

**`chmod 666 /var/log/devopsfetch.log`**: Sets permissions to allow all users to read and write to the log file.

### 4. Create a Systemd Service Unit File
This file creates a service named `devopsfetch.service` in the systemd service directory.

```sh
[Unit]
Description=DevOpsFetch Monitoring Service
After=network.target

[Service]
ExecStart=/bin/bash -c '/usr/local/bin/devopsfetch -t "$(date -d \"5 minutes ago\" +\"%Y-%m-%d %H:%M:%S\")" "$(date +\"%Y-%m-%d %H:%M:%S\")"'
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```
_/etc/systemd/system/devopsfetch.service_

### 5. Create Systemd Timer File
This file defines the schedule for running the service. The timer runs 5 minutes after system boot and it runs every 5 minutes after it becomes active.

```sh
[Unit]
Description=Run DevOpsFetch every 5 minutes

[Timer]
OnBootSec=5min
OnUnitActiveSec=5min
Persistent=true

[Install]
WantedBy=timers.target
```
_/etc/systemd/system/devopsfetch.timer_

### 6. Enable and Start the Service and Timer
The commands below reload the **systemd daemon** so the new service and timer will reflect, enables the `devopsfetch.service` and `devopsfetch.timer` to start automatically on system boot.

```sh
systemctl daemon-reload
systemctl enable devopsfetch.service
systemctl enable devopsfetch.timer
systemctl start devopsfetch.timer
```

### 7. Configure Log Rotation
This create a file named `devopsfetch` in the `lograte` configuration directory. This file defines how the log file is rotate.

The content of the file:
- **`hourly`**: Specifies that the log file should be rotated hourly.

- **`rotate 288`**: Specifies that 288 rotated log files should be kept.

- **`compress`**: Specifies that the rotated logs should be compressed (zipped). 

- **`missingok`**: Specifies that if the log file is missing, it should not cause an error.

- **`notifempty`**: Specifies that an email notification should be sent if the log file is empty.

- **`create 666 root root`**: Specifies that a new log file should be created with read/write permissions for the root user, groups and others.

```sh
/var/log/devopsfetch.log {
    hourly
    rotate 288
    compress
    missingok
    notifempty
    create 666 root root
}
```

In summary. this script setup a monitoring service called `devopsfetch` that runs every 5 minutes, logs its out to `/var/log/devopsfetch.log`, starts automatically on system boot and rotates its log files hourly to conserve disk space.

## Validating The Scripts & Testing the Functionality `devopsfetch` Service

### Prerequisites
The full functionality of the `devopsfetch` service can only tested if the 2 packages are installed on your system. 
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Nginx](https://ubuntu.com/tutorials/install-and-configure-nginx#2-installing-nginx)

To validate the scripts, run the following commands:

```sh
sudo chmod +x devopsfetch install_devopsfetch.sh
sudo ./install_devopsfetch.sh
```

After running the commands above, run the commands below to test the functionality of the `devopsfetch.service`:

1. List Active Ports.

```sh
devopsfetch -p
```

![ports](./images/1%20ports.png)

2. List Docker images and containers.

```sh
devopsfetch -d 
```

![docker images and containers](./images/2%20image%20and%20container.png)

_**Note**: I created a container from the official **mariadb** image before running the command above._

3. List all users and last login.

```sh
devopsfetch -u
```

![users](./images/3%20user.png)

4. Display `ikennade.website` domain and the URL it proxies to.

```sh
devopsfetch -n ikennade.website
```

![domain_name](./images/4%20domain_name.png)

_**Note**: I configured Nginx to act a Reverse Proxy for a Golang Web App._

5. Display activities on a July 23rd, 2024.

```sh
devopsfetch -t 2024-07-23
```

![july 23](./images/5%20july23.png)