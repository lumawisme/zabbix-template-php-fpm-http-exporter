# Zabbix Template for PHP-FPM HTTP Exporter

A production-ready, highly optimized Zabbix template designed to monitor PHP-FPM performance metrics seamlessly. 

> ⚠️ **Credits & Acknowledgments**
> This repository contains **only the Zabbix template configuration**. All credit for the underlying metrics collection engine goes to the creator of the original Prometheus exporter: **[bakins/php-fpm-exporter](https://github.com/bakins/php-fpm-exporter)**. If you use this monitoring integration, please consider starring their original repository!

---

## 🚀 Template Design Features

- **Master/Dependent Item Architecture:** Performs exactly *one* network request per polling cycle to extract all data, completely eliminating unnecessary scraping overhead on your production application server.
- **Zabbix 7.2+ Native Formatting:** Built and structured to support the latest JSON import schemas.
- **Port Flexibility:** Built using user macros, allowing you to quickly change target firewall ports without altering the core items.
- **Human-Readable States:** Integrated value mapping translates raw binary `0` and `1` values into transparent `Up` / `Down` service indicators.

---

## 📊 Monitored Metrics & Triggers

The template dynamically parses the exporter data using Prometheus pattern preprocessing rules to track:

| Metric Name | Zabbix Key | Type | Trigger Threshold / Alert |
| :--- | :--- | :--- | :--- |
| **PHP-FPM Status** | `phpfpm.up` | Gauge | **DISASTER:** Fires instantly if the service crashes. |
| **Exporter Reachability** | `phpfpm.up` | No Data | **DISASTER:** Fires if Zabbix gets 3 minutes of total network silence. |
| **Active Processes** | `phpfpm.processes.active` | Gauge | Performance tracking curve. |
| **Idle Processes** | `phpfpm.processes.idle` | Gauge | Pool scaling curve. |
| **Listen Queue Length** | `phpfpm.queue.current` | Gauge | **AVERAGE:** Fires if > 5 requests are stuck waiting in line. |
| **Max Children Reached** | `phpfpm.max_children` | Counter Delta | **HIGH:** Warning if the pool maxes out process allocations. |
| **Slow Requests Tally** | `phpfpm.slow_requests` | Counter Delta | Increments if your PHP code hits a performance bottleneck. |
| **Total Accepted Connections** | `phpfpm.connections.total` | Counter | **INFO:** Flipped counter triggers an auto-closing event log on service restarts. |
| **Traffic Throughput** | `phpfpm.connections.rate` | Deriv (Float) | Tracks active processed requests per second. |

---

## 🛠️ Prerequisites & Setup

### 1. Enable the PHP-FPM Status Page
Your target server pool configuration (usually `/etc/php-fpm.d/www.conf`) must have its status endpoint uncommented and enabled:
```ini
pm.status_path = /status
```

### 2. Deploy the Original Exporter
Download and execute the binary from bakins/php-fpm-exporter targeting the industry-standard port 9253:
```bash
# Example for a RHEL System running a native Unix Socket:
./php-fpm-exporter --fastcgi unix:///run/php-fpm/www.sock --addr 0.0.0.0:9253
```

## 📥 Installation
Download the **template_php_fpm_http_exporter.json** file from this repository.

In your Zabbix Web Console, navigate to **Data collection > Templates**.

Click **Import** in the top-right corner, upload the JSON file, and confirm.

Link the template to your designated host (e.g., your web server).

(Optional) If your exporter runs on a port other than **9253**, navigate to your host's Macros configuration tab and update the **{$PHPFPM.PORT}** variable.
