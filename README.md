# Building `devopsfetch` for Server Information Retrieval and Monitoring

## Objective
 Developing a tool for devops named `devopsfetch` that collects and displays System Information, Active Ports, User Logins, Nginx Configurations, Docker Images and Container Statuses. The tool is also implemented as a `systemd` service to Monitor and Log these activities continuously.

## Features
### Information Retrieval
1. Ports:
   - Displays all active ports and services (`-p`or `--port`).
   - Provides detailed information about a specific port (`-p` `<port_number>`).
2. Docker:
   - Lists all Docker images and containers (`-d` or `--docker`).
   - Provides detailed information about a specific container (`-d` `<container_name>`).
3. Nginx:
   - Displays all Nginx domains and their ports (`-n` or `--nginx`).
   - Provides detailed configuration information for a specific domain (`-n` `<domain>`).
4. Users:
   - List all users and their last login times (`-u` or `--users`).
   - Provide detailed information about a specific user (`-u` `<username>`).
5. Time Range:
   - Provides detailed information about activities within the specified time range (`-t` `<start_time>` `<end_time>`).
6. Help:
   - Provides usage examples for each command-line flag (`-h` or `--help`).

## Implementing The `devopsfetch` Tool
The approach to implementing the `devopsfetch` was divided 3 sections which will be discussed as follows:

### 1. Essential Functions

**`log_output()`**: This function captures the script's output and appends it to a log file `/var/log/devopsfetch.log`. This is vital for troubleshooting and logging.

**`display_help()`**: This function displays the help message for the `devopsfetch` tool. It provides usage examples for each command-line flag.

**`format_table()`**: This function formats the output of the `devopsfetch` tool as a table. It takes the output of the `devopsfetch` tool as input and returns a user-friendly tabular format.

### 2. Core Functionality Functions

**`display_ports()`**: This displays information about all the active network ports.
- If no argument is passed alongside `-p`, it list all the active ports, their corresponding user and associated services.

- If an argument is passed alongside `-p`, it provides detailed information about a specific port.

**`display_docker()`**: This displays information about all the Docker images and containers.
- If no argument is passed alongside `-d`, it lists all Docker images and containers.

- If a `container_name` is passed alongside `-d`, it provides detailed information (_i.e. Name, Image, State, IP, Ports, User and Services_) about the specified container.

**`display_nginx()`**: It retrieves information about Nginx domains and configuration.
- If no argument is passed alongside `-n`, it lists all Nginx domains and their ports.

- If a `domain name` is passed alongside `-n`, it displays the Domain, Port and URL it is proxying to.

**`display_users()`**: It provides information about system users.
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