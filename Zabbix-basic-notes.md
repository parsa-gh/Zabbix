# Zabbix Notes

## MariaDB (Why is it used most of the time?)
MariaDB is used most of the time with Zabbix.

> Question: Why is MariaDB commonly used?

---

## Steps to Install Zabbix
1. Download the RPM package
2. Download MariaDB from the required resources

---

## Zabbix UTF-8 Encoding: utf8mb4 vs utf8
Zabbix uses **utf8mb4** encoding instead of **utf8**.

**Reason:**
- `utf8` uses a maximum of **3 bytes** to encode data. It can represent many Unicode characters, but some symbols (e.g., emojis) may not be supported.
- `utf8mb4` uses **4 bytes** to encode characters and covers the full set that `utf8` might exclude.
- Using `utf8mb4` is optional. An admin can use another encoding by setting:

--default-character-set=utf8mb4
**Reason:**
- `utf8` uses a maximum of **3 bytes** to encode data. It can represent many Unicode characters, but some symbols (e.g., emojis) may not be supported.
- `utf8mb4` uses **4 bytes** to encode characters and covers the full set that `utf8` might exclude.  
- Using `utf8mb4` is optional. An admin can use another encoding by setting:

--default-character-set=utf8mb4

## Default Zabbix Server Configuration Path
The default Zabbix server config is located at:
- `/etc/Zabbix/zabbix_server.conf`

---

## Production Security: SELinux or AppArmor
Using Zabbix in production requires configuring **SELinux** (Security-Enhanced Linux) or **AppArmor**.

These tools add security policies at the OS level to control what processes are allowed to do.

---

## Frontend Requirements: nginx/apache + PHP
Zabbix frontend needs an **nginx** or **apache** web server and **PHP version higher than 8**.

Based on documentation: **Apache is better**. It is known to be faster and often provides a slight advantage.

---

## Zabbix Components
- Zabbix server
- Zabbix DB
- Zabbix frontend
- Zabbix Agent(1/2)

---

## DB Credentials Storage in Frontend
In the frontend DB configuration, credentials can be stored in:

- **Plain text**
- **HashiCorp Vault**
  - Works with secret key exchange
  - Suitable for DevOps teams
- **CyberArk Vault**
  - Best option for enterprise companies
  - A PAM solution: you enter a portal and access only what you are allowed to
  - Passwords are stored in a Digital Vault

---

## Default User Credentials After Frontend Configuration
- User : **Admin**
- Pass : **zabbix**

---

## HA in Zabbix
HA (High Availability) came to Zabbix at version **6**.

At the end, everything Zabbix stores internally is saved in the database:

- host configs
- history data
- HA information
- and more

---

## VRRP (Virtual Router Redundancy Protocol)
VRRP is used for HA.

> More info will be added.

---

## NVPS (New Values Per Second)
NVPS is an indicator that helps you understand when you need to scale up the environment.

---

## Time in Zabbix
Time in Zabbix is based on the local Linux system time (and is changeable).

---

## LDAP / SAML Logins
Even if you use LDAP, SAML, or similar methods for people to log in to Zabbix, those users still need to be added in Zabbix.

---

## User Types in Zabbix (After v6)
- Users (never HA)
- Admin
- Super admin

---

## Zabbix Agent

### Zabbix Agent v2 (After v5)
After v5, Zabbix Agent v2 was published.

- v1 was written in C
- v2 is written in Golang + some C code reused from zabbix agent to :
  > Reduce the number of TCP connections;
  
  > Provide improved concurrency of checks;
  
  > Be easily extendable with plugins, which provide simple checks with minimal code and support complex checks consisting of long-running scripts and standalone data gathering with periodic reporting;

  > Function as a replacement for Zabbix agent, supporting all previous features.

---

## Zabbix Passive Agent
In passive mode:

- The client sends its data to the server
- Port: `10050/TCP`

**Benefits:**
- Helps meet security requirements in some environments
  - Example: firewall blocks outgoing traffic
  - Example: you don’t want to involve a subnet you control differently

---

## Zabbix Active Agent
In active mode:

- The server asks for data frequently
- The hostname in the config file on the client OS must match the exact name in:
  - Data Collection → Hosts
- Port: `10051/TCP`

## Zabbix Simple Checks
Zabbix simple checks perform basic monitoring tasks that do not require scripts or complex flows (e.g., ping).

---

## Zabbix Trapper
Zabbix trapper allows external applications or scripts to send data to the Zabbix server.

- The Zabbix server stays passive and listens for incoming data.

---

## JMX (Java Management Extension)
JMX is a technology used to monitor and manage Java applications.

---

## Users, Media, Authentication, and Actions

### Users → Media
Notifications for external options such as:

- webhooks
- email
- etc.

### Users → Authentication
Ways to log in:

- Internal (default)
- HTTP Authentication
  - Web server authentication such as Kerberos/NTLM
- LDAP
  - Active Directory
- SAML
  - Security Assertion Markup Language
  - Uses SSO: the user authenticates with an identity server and Zabbix connects to the service through SSO
- MFA

### Alerts → Scripts
Scripts can be placed based on their target.

## Options

### Action → Operation
- Runs automatically based on conditions
- Example: if a CPU trigger fires → restart the service

### Manual Host Action
- Run scripts manually on monitoring hosts

### Manual Event Action
- Run scripts manually on monitoring problems/events

---

## Active vs Passive (Summary)
- **Active agent**: pushes data to the Zabbix server
- **Passive agent**: Zabbix server pulls data from the agent
- TCP connection is established from the server

---

## Event Correlation
Zabbix event correlation rules allow you to correlate events from different sources and generate new events based on correlation results.

This can be useful for:

- detecting complex problems spanning multiple systems
- reducing the number of alerts you need to respond to

---

## Difference Between Item and Trigger
- **Item**: collects data based on what you define
- **Trigger**: evaluates the data to detect problems

---

## Prototype (Discovery Macros)
Prototypes use discovery macros to automate some tasks.

**Example:** If you have 3 disks:

- free space on `/dev/sda`
- free space on `/dev/sdb`
- free space on `/dev/sdc`

Instead of writing everything manually, you can use:

- `free space on {#FSNAME}`

---

## Service Monitoring
Service monitoring monitors the whole infrastructure you mark as a service and correlates events.

## Dependent Item
- No external connection (depends on parent item data)
- Mainly used for:
  - creating multiple items from a single data source
  - reducing server load
  - preprocessing complex data

---

## Script Item Types

### Zabbix Agent (Active)
- Zabbix agent (active) collects data efficiently based on the pull/push method.

### Zabbix Agent (Passive)
- Zabbix agent (passive) is used for server-side polling behavior.

---

## Simple Check
Sends (For example) ICMP echo requests (ping) to verify if a host is reachable and responsive.

- Connection direction: Zabbix server → monitored host via ICMP
- Mainly used for: checking whether a device is up or down (host availability monitoring)

---

## SNMP Agent (Active-Pull)
To collect data from SNMP-monitored devices where it is impractical to use Zabbix agents (e.g., printers, network switches).

- SNMP checks are performed over **161/UDP only**
- The Zabbix server queries the monitored device
- The device responds with **OID (object IDs)** and values

---

## SNMP Trap (Passive-Push)
Devices send unsolicited alert notifications to Zabbix when specific events occur.

- Connection direction: monitored device → Zabbix trap receiver
- Port: **162/UDP**
- Mainly used for: real-time critical alerts and events without constant polling

---

## Zabbix Internal
Collects metadata about the Zabbix server and agent status itself.

- No external connection is needed
- Connection is internal within the Zabbix system
- Mainly used for: monitoring Zabbix server performance, queue status, and internal health checks

---

## Zabbix Trapper
Accepts data pushed directly by external sources or scripts to the Zabbix server.

- Connection direction: external application/script → Zabbix server
- Port: **10051/TCP**
- Mainly used for: custom applications to send their own metrics and data directly
- Key difference with External-Check is that agent runs script itself
- Server Load : Lower than external check

---

## External Check
Executes shell commands or scripts on the Zabbix server to retrieve monitoring data.

- Connection depends on the command executed (SSH, HTTP, etc.)
- Mainly used for: custom scripts or one-off commands to gather specific system information
- Key difference with Zabbix-trapper is that server runs script on agent
- Server load : High

---

## Database Monitor
Connects to databases (MySQL, PostgreSQL, Oracle, etc.) to execute queries and collect database metrics.

- Connection direction: Zabbix server → database server
- Ports:
  - **3306** for MySQL
  - **5432** for PostgreSQL
- Mainly used for: monitoring database performance, query results, and DB-specific metrics
### Main Approaches :
- 1. ODBC
  2. Agentless(External-Checks)
  3. Agent-Based : Agent does queries in Database
#### ODBC Data Flow:
- 4 Logical layer and 3 simplified layers are included.
  
- 4 Logical layer :
  
  1.ODBC API
  
  2.ODBC Driver Manager (unixODBC)
  
  3.DB-Specific driver --> `Translates ODBC to DB protocols`
  
  4.Actual DB
- 3 Simplified layers :

  1. ODBC API (Universal Interface)
  2. Translation layer (Middleware API)
  3. DB communication (Low-level queries)
 ##### How it works?
 ```
1. Application → ODBC API Call
                 (SQLConnect, SQLExecute, etc.)
                 
2. Driver Manager (unixODBC) → Reads .odbc.ini
                              → Locates driver
                              → Loads driver into memory
                              
3. Database Driver → Translates ODBC call to DB protocol
                  → Opens connection to database
                  → Sends native query/command
                  
4. Database → Executes query
           → Returns result set
           
5. Driver → Converts result to ODBC format
         → Returns to caller
```
## Zabbix Database History Tables

### History
Stores **numeric (float) values** from item collection
- Contains raw metric data: CPU usage, memory, network throughput, etc.
- **Huge table** — grows very quickly in large deployments
- Data structure: timestamp, itemid, value, clock

### History_uint
Stores **unsigned integer values** from items
- Examples: packet counts, error counts, connection counts
- More space-efficient than history for integer-only data
- Same purpose, different data type optimization

### History_str
Stores **string/text values** from items
- Examples: system uptime strings, status messages, file contents
- Grows slower than numeric tables but can still be large
- Used for non-numeric monitoring data

### History_log
Stores **log file entries** collected via log monitoring items
- Contains extracted log lines from monitored hosts
- Slower growth than others (depends on log volume)
- Useful for centralized log aggregation

## Zabbix Database Trends Tables

### Trends
Stores **aggregated numeric (float) values** from item collection
- Contains hourly aggregated data: average, min, max, count
- Reduces storage by summarizing `history` data after retention period
- Much smaller than `history` table
- Data structure: itemid, clock, num, value_min, value_avg, value_max

### Trends_uint
Stores **aggregated unsigned integer values** from items
- Examples: hourly summaries of packet counts, error counts, connection counts
- More space-efficient than `trends` for integer-only aggregated data
- Same purpose as `trends`, different data type optimization
- Reduces disk space while preserving historical trend analysis capability
---

# Zabbix Database Performance Optimization

## Overview

When running Zabbix at scale, the database is the first bottleneck. InnoDB I/O and internal caching layers become critical before CPU or memory issues arise. Database tuning directly determines overall system stability.

## 1. Zabbix Database Structure
```
 Note that Zabbix server doesn't write to DB directly, it writes in buffer(Cache) then cache will write it on DB, which may create some bottlenecks due to cache limit.
```
Zabbix data is divided into two main table groups:

| Type | Tables | Characteristics |
|------|--------|-----------------|
| History | history, history_uint, history_str, history_log | Stores raw metrics per second, heavy random I/O load |
| Trends | trends, trends_uint | Aggregated hourly averages/max/min values, lighter I/O |

Almost all database load originates from the history tables. The key is how efficiently you manage and periodically clean up these tables.

## 2. Separating History and Trends Storage

Split history and trends data across different disks:
- Use **SSDs for history tables** — handles constant random writes
- Use **traditional storage for trends** — lighter I/O requirements
- This reduces disk contention dramatically

## 3. Partitioned Tables

Partitioned table design prevents the database from stalling when the housekeeper deletes millions of rows.

## 4. Housekeeper Tuning

The Housekeeper deletes old data. In large setups, disable automatic cleanup for most data types and use partitioned tables instead.

**Configuration (Administration → Housekeeping):**
- Trends → ☐ Disable
- Events and alerts → ☐ Disable
- Services → ☐ Disable
- User sessions → ☑ Keep enabled
- History → ☐ Disable
  
## HTTP Agent
Sends HTTP/HTTPS requests to web servers and collects response data or metrics.

- Connection direction: Zabbix server → web server
- Ports:
  - **80/TCP** (HTTP)
  - **443/TCP** (HTTPS)
- Mainly used for: monitoring web applications, APIs, and checking HTTP response times/status codes

---

## IPMI(Intelligent Platform Management Interface) Agent
Communicates with IPMI (Intelligent Platform Management Interface) to monitor hardware sensors and server health.

- Connection direction: Zabbix server → IPMI interface
- Port: **623/UDP**
- Mainly used for: physical server monitoring like temperature, voltage, fan status, and power supply health

### How IPMI works?
- IPMI is a set of computer interface specifications for an autonomous computer subsystem that provides management and monitoring capabilities independently of the operating system
- The monitored system may be powered off, but must be connected to a power source
- It has its own IP, but the con is, some devices may not support IPMI
  <img width="500" height="355" alt="image" src="https://github.com/user-attachments/assets/5a128ecc-6d29-4f0a-8645-28b723a797c1" />

---

## SSH Agent
Executes commands remotely on monitored hosts via SSH.

- Connection direction: Zabbix server → SSH daemon on monitored host → Executes command
- Port: **22/TCP**
- Mainly used for: remote Linux/Unix command execution to gather system metrics when the Zabbix agent is not available

---

## Telnet Agent
Connects to devices via Telnet to execute commands and retrieve output data.

- Connection direction: Zabbix server → Telnet service on monitored device → execute commands
- Port: **23/TCP**
- Mainly used for: legacy devices and network equipment that only support Telnet for remote management

---

## JMX Agent
Collects metrics from Java applications via Java Management Extensions (JMX).

- Connection direction: Zabbix server → JMX port on Java application
- Default port: **9999/TCP**
- Mainly used for: monitoring Java application performance, memory, threads, and application-specific metrics

---

## Calculated
Performs mathematical calculations on values from other items to derive new metrics.

- No external connection (uses data from other Zabbix items)
- Mainly used for:
  - combining multiple metrics
  - computing ratios
  - creating custom KPIs from existing item values

---

## Script Item
Executes JavaScript or custom scripts on the Zabbix server to collect or process data.

- Connection depends on what the script does (it may connect to APIs, databases, or external services)
- Mainly used for:
  - complex data collection logic
  - custom transformations
  - integration with external systems

---

## Browser
Uses browser automation to interact with web applications and extract data from rendered content.

collect data by executing a user-defined JavaScript code and retrieving data over HTTP/HTTPS. This item can simulate such browser-related actions as clicking, entering text, navigating through web pages, and other user interactions with websites or web applications.

- Connection direction: Zabbix server browser engine → web server
- Ports: **80/443/TCP** via HTTP/HTTPS
- Mainly used for:
  - monitoring dynamic web applications
  - JavaScript-rendered pages
  - user interaction-based scenarios
