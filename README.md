# Linux-Server-Monitoring-Tool

A comprehensive command-line server monitoring tool built with shell scripting that tracks system resources (CPU, memory, disk usage) and sends automated alerts when thresholds are breached.

## Features

- **Real-time Monitoring**: Monitor CPU, memory, and disk usage
- **Automated Alerts**: Email and log-based alerts when thresholds are exceeded
- **Configurable Thresholds**: Customizable warning and critical thresholds
- **Multiple Operating Modes**: One-time checks, continuous monitoring, or scheduled runs
- **Detailed Reporting**: Generate comprehensive system reports
- **Process Analysis**: View top CPU and memory consuming processes
- **Systemd Integration**: Run as a system service
- **Log Management**: Structured logging with automatic rotation
- **Color-coded Output**: Easy-to-read terminal output with status colors

## Requirements

- Linux operating system (Ubuntu, CentOS, RHEL, Debian, etc.)
- Bash shell (version 4.0 or higher)
- `bc` calculator package
- `mail` command (optional, for email alerts)

## Installation

### Quick Installation

1. **Download the files**:
   ```bash
   # Create project directory
   mkdir server-monitor && cd server-monitor
   
   # Copy the server_monitor.sh script and install.sh to this directory
   ```

2. **Run the installation script**:
   ```bash
   chmod +x install.sh
   ./install.sh
   ```

### Manual Installation

1. **Install dependencies**:
   ```bash
   # Ubuntu/Debian
   sudo apt-get install bc mailutils
   
   # CentOS/RHEL
   sudo yum install bc mailx
   ```

2. **Make the script executable**:
   ```bash
   chmod +x server_monitor.sh
   ```

3. **Create configuration file**:
   ```bash
   ./server_monitor.sh --config
   ```

## Configuration

The tool uses a configuration file located at `~/.server_monitor.conf`:

```bash
# Threshold values (in percentage)
CPU_THRESHOLD=80
MEMORY_THRESHOLD=85
DISK_THRESHOLD=90

# Check interval in seconds (for continuous monitoring)
CHECK_INTERVAL=60

# Email settings
ALERT_EMAIL="admin@example.com"
ENABLE_EMAIL_ALERTS=true

# Logging settings
ENABLE_LOG_ALERTS=true
```

### Configurable Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `CPU_THRESHOLD` | CPU usage alert threshold (%) | 80 |
| `MEMORY_THRESHOLD` | Memory usage alert threshold (%) | 85 |
| `DISK_THRESHOLD` | Disk usage alert threshold (%) | 90 |
| `CHECK_INTERVAL` | Monitoring interval in seconds | 60 |
| `ALERT_EMAIL` | Email address for alerts | "" |
| `ENABLE_EMAIL_ALERTS` | Enable/disable email notifications | false |
| `ENABLE_LOG_ALERTS` | Enable/disable log file alerts | true |

## Usage

### Command Line Options

```bash
./server_monitor.sh [OPTIONS]

Options:
  -h, --help              Show help message
  -s, --status            Show current system status
  -m, --monitor           Start continuous monitoring
  -c, --config            Create/recreate configuration file
  -t, --top               Show top processes
  -r, --report            Generate detailed report
  -l, --logs              Show recent log entries
  --test-alert            Send test alert
```

### Usage Examples

#### Check Current Status
```bash
./server_monitor.sh --status
```
Output:
```
=== System Information ===
Hostname: webserver01
Uptime: up 5 days, 14 hours, 32 minutes
Load Average:  0.45, 0.52, 0.48
Kernel: 5.4.0-74-generic
Date: Mon Sep 16 10:30:45 2025

=== Current Resource Usage ===
CPU Usage: 45.2% (OK)
Memory Usage: 67.8% (OK)
Disk Usage (/): 78.5% (OK)
```

#### Start Continuous Monitoring
```bash
./server_monitor.sh --monitor
```

#### Generate Detailed Report
```bash
./server_monitor.sh --report
```

#### View Top Processes
```bash
./server_monitor.sh --top
```

## Alert System

### Alert Levels

1. **OK**: Usage below 80% of threshold
2. **WARNING**: Usage between 80% and threshold
3. **CRITICAL**: Usage above threshold

### Alert Methods

1. **Console Output**: Color-coded status messages
2. **Log File**: Structured logging to `/var/log/server_monitor.log`
3. **Email Alerts**: Automated email notifications (requires mail setup)

### Sample Alert Messages

```
[2025-09-16 10:35:22] [ERROR] ALERT: High CPU usage detected on webserver01 - Current: 85.3%, Threshold: 80%
[2025-09-16 10:35:22] [ERROR] ALERT: High Memory usage detected on webserver01 - Current: 92.1%, Threshold: 85%
```

## Automation Options

### 1. Cron Job (Periodic Checks)

Add to crontab for regular monitoring:
```bash
# Check every 5 minutes
*/5 * * * * /path/to/server_monitor.sh --status >/dev/null 2>&1

# Generate daily report
0 6 * * * /path/to/server_monitor.sh --report
```

### 2. Systemd Service (Continuous Monitoring)

The installation script creates a systemd service:

```bash
# Enable and start the service
sudo systemctl enable server-monitor
sudo systemctl start server-monitor

# Check service status
sudo systemctl status server-monitor

# View service logs
sudo journalctl -u server-monitor -f
```

## File Structure

```
server-monitor/
├── server_monitor.sh      # Main monitoring script
├── install.sh            # Installation and setup script
├── README.md             # This documentation
└── examples/
    ├── crontab.example   # Sample cron configuration
    └── alerts.example    # Sample alert configurations
```

## Log Files

- **Main Log**: `/var/log/server_monitor.log` (or `~/server_monitor.log` if no write access)
- **Service Log**: `journalctl -u server-monitor`
- **Log Rotation**: Automatically configured for daily rotation, keeping 7 days

## Troubleshooting

### Common Issues

1. **Permission Denied for Log File**:
   ```bash
   sudo chmod 666 /var/log/server_monitor.log
   ```

2. **Mail Command Not Found**:
   ```bash
   # Ubuntu/Debian
   sudo apt-get install mailutils
   
   # CentOS/RHEL
   sudo yum install mailx
   ```

3. **BC Calculator Not Found**:
   ```bash
   # Ubuntu/Debian
   sudo apt-get install bc
   
   # CentOS/RHEL
   sudo yum install bc
   ```

### Debug Mode

Run with verbose output:
```bash
bash -x ./server_monitor.sh --status
```

## Customization

### Adding Custom Metrics

You can extend the script to monitor additional metrics:

1. **Network Usage**:
   ```bash
   check_network() {
       local rx_bytes=$(cat /sys/class/net/eth0/statistics/rx_bytes)
       local tx_bytes=$(cat /sys/class/net/eth0/statistics/tx_bytes)
       # Add your logic here
   }
   ```

2. **Service Status**:
   ```bash
   check_service() {
       local service_name=$1
       systemctl is-active --quiet "$service_name"
       return $?
   }
   ```

### Custom Alert Handlers

Add custom alert methods by modifying the `send_email_alert` function:

```bash
send_custom_alert() {
    local message=$1
    
    # Slack webhook
    curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"$message\"}" \
        YOUR_SLACK_WEBHOOK_URL
    
    # SMS via API
    # Add your SMS API integration here
}
```

## Performance Impact

The monitoring tool is designed to have minimal system impact:

- **CPU Usage**: < 1% during checks
- **Memory Usage**: < 10MB
- **Disk I/O**: Minimal, only during logging
- **Network**: Only for email alerts (if enabled)

## Security Considerations

1. **File Permissions**: Ensure proper permissions on config and log files
2. **Email Security**: Use secure SMTP settings for email alerts
3. **Service Account**: Run the service with minimal required privileges
4. **Log Rotation**: Prevent log files from consuming excessive disk space

## Contributing

To contribute to this project:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly on different Linux distributions
5. Submit a pull request
