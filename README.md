I am building a Django-based Network Monitoring System for an isolated intranet environment (no internet access).

PROJECT PURPOSE:
The system scans an internal network periodically using Nmap and tracks all connected devices. It detects changes between scans and generates alerts for security and network monitoring.

--------------------------------------------
CURRENT TECHNOLOGY STACK:
- Backend: Django (Python)
- Database: PostgreSQL
- Scanner: Nmap (via Python subprocess or python-nmap)
- Architecture: Service-layer based (not logic inside views/models)
- Deployment: Fully offline / air-gapped system

--------------------------------------------
CURRENT PROJECT STRUCTURE:

Django project contains these apps:

1. accounts → authentication (future login system)
2. devices → stores current known state of all devices
3. scans → stores scan session metadata
4. scanner → core scanning + parsing + processing logic
   - services/
     - nmap_runner.py
     - xml_parser.py
     - scan_processor.py
5. alerts → alert generation and management
   - services/
     - alert_service.py
6. dashboard → future UI/API layer

--------------------------------------------
DATABASE DESIGN (CURRENT INTENT):

We plan to have 4 core tables:

1. Device Table (CURRENT STATE)
- mac_address (primary identity)
- ip_address
- hostname
- vendor
- operating_system
- is_authorized
- is_online
- last_seen

2. Scan Table (SNAPSHOT)
- scan_id
- timestamp
- subnet_scanned
- total_devices_found

3. DeviceHistory Table (CHANGE LOG)
- device reference
- old state
- new state
- change type (IP change, OS change, etc.)
- timestamp

4. Alert Table (EVENTS)
- alert_type (new device, missing device, spoofing, OS change, etc.)
- device reference
- severity
- timestamp
- description

--------------------------------------------
SCAN DATA FORMAT (INPUT FROM NMAP):

Each scan produces a list like:

{
  ip,
  mac,
  hostname,
  vendor,
  os,
  open_ports
}

--------------------------------------------
CURRENT SYSTEM FLOW:

1. Nmap scan runs on subnet (/24 blocks of a /16 network)
2. Output is parsed into structured Python objects (list of dicts)
3. Scan results are passed to scan_processor service
4. scan_processor compares scan data with Device table
5. Changes are detected
6. Device table is updated (latest truth)
7. DeviceHistory is written
8. Alerts are generated and stored
9. Dashboard reads from DB

--------------------------------------------
CHANGE DETECTION RULES (IMPORTANT):

We detect 6 types of changes:

1. New MAC address → new device detected
2. IP change for same MAC → device moved
3. Missing device → not seen in latest scan
4. MAC/IP spoofing → same IP used by multiple MACs
5. Unauthorized device reappears online
6. OS change detected (Windows ↔ Linux, etc.)

--------------------------------------------
DESIGN PRINCIPLES:

- MAC address is the unique identifier for devices
- Device table always represents latest known state
- Scan table is temporary snapshot
- DeviceHistory stores only changes (audit log)
- Alerts are derived from comparison logic
- First scan acts as baseline

--------------------------------------------
CURRENT DEVELOPMENT STATUS:

- Django project is already created
- Apps are created and registered
- Service layer structure is created
- PostgreSQL is configured and connected
- Initial migrations are done
- GitHub repository already initialized and pushed
- No models are fully implemented yet (next step)

--------------------------------------------
WHAT I NEED NOW:

I want a FULL END-TO-END ARCHITECTURE EXPLANATION AND NEXT DEVELOPMENT PLAN INCLUDING:

1. Final architecture diagram (text-based is fine)
2. Explanation of how each app communicates
3. Exact flow of scan → parse → compare → DB update → alerts
4. Which module handles what responsibility
5. Clear explanation of data flow between tables
6. How to implement change detection properly in scan_processor
7. What should be implemented next in order (step-by-step roadmap)
8. How to structure Django models properly before coding begins

IMPORTANT:
- Must be production-grade design thinking
- Must be implementable in Django
- Must consider scalability for large networks (~1000+ devices)
- Must consider performance and efficiency of comparisons

Give a structured, clear, engineering-level explanation.



Change detection prompt-

I am building a Django-based network monitoring system (air-gapped / intranet environment).

I already have a design where:

1. Device table = current known state of each device (latest snapshot)
2. Scan table = raw scan results for each scan cycle
3. DeviceHistory table = logs of changes between states
4. Alert table = stores detected security/monitoring alerts

Each scan produces a list of devices like:
{
  ip,
  mac,
  hostname,
  vendor,
  os,
  open_ports
}

RULES:
- First scan acts as baseline (initial population of Device table)
- Every next scan must compare against existing Device table
- MAC address is the PRIMARY identity of a device
- IP can change but MAC identifies device uniquely
- Devices not seen in latest scan are considered "missing/offline"
- New MAC = new device
- OS changes must be detected (Windows ↔ Linux, etc.)
- IP change for same MAC must be detected
- Duplicate IP across MACs may indicate spoofing
- If an unauthorized device comes back online, it should trigger alert

I want a COMPLETE pseudocode for change detection logic that includes:

1. Input: previous device state from DB + current scan result
2. Step-by-step comparison logic
3. Handling each case:
   - new device detection
   - existing device update
   - IP change detection
   - OS change detection
   - missing device detection
   - device reappearing detection
   - possible MAC/IP spoofing detection
   - unauthorized device logic
4. How Device table should be updated after comparison
5. How DeviceHistory should be written
6. How Alerts should be generated and what fields they should contain
7. Clear looping structure (how we iterate scan list vs DB records)
8. Edge cases (duplicate MACs, duplicate IPs, partial scan data)
9. Must be structured like production-grade backend logic (not just theory)

Also explain the data flow:
Scan → Parse → Compare → Update DB → Create Alerts

Give output in clean pseudocode format suitable for implementing in Python/Django service layer.

MVP ----------------------/////////////

PROJECT SUMMARY

We are building a Django-based Network Monitoring System for a completely offline/air-gapped intranet environment. The system periodically scans internal networks using Nmap, detects connected devices, tracks changes between scans, and generates security/network alerts. The main goal is to monitor devices inside a private organization network without requiring internet access.

What we have completed till now:

* Django project is created
* PostgreSQL is connected and working
* GitHub repository is initialized
* Apps created:

  * accounts
  * devices
  * scans
  * scanner
  * alerts
  * dashboard

Project architecture:

* Using service-layer design
* Business logic is kept inside services/
* Not placing logic inside views/models

Current service structure:

* scanner/services/

  * nmap_runner.py
  * xml_parser.py
  * scan_processor.py
* alerts/services/

  * alert_service.py

System purpose:

* Run Nmap scans on internal subnets
* Detect devices on the network
* Compare scans with existing device records
* Detect changes
* Generate alerts

Planned core database tables:

1. Device
2. Scan
3. DeviceHistory
4. Alert

Important design decisions:

* MAC address is the primary identity of a device
* Device table stores latest/current state only
* DeviceHistory stores change logs
* Alerts are generated from comparison logic
* First scan acts as baseline

Current scan flow:

1. Nmap scan runs
2. Output parsed into structured Python objects
3. scan_processor compares results with DB
4. Device changes detected
5. Alerts generated
6. Dashboard will read from database

Planned change detection:

* New device detected
* IP change for same MAC
* Missing device
* MAC/IP spoofing
* Unauthorized device reappears
* OS change detected

Current status:

* No final models implemented yet
* Ready to start MVP implementation now
* Goal is to keep MVP extremely simple first
_____________________________________________________

commands for testing:
* python manage.py runscan
* python manage.py shell

to view:
from devices.models import Device
Device.objects.all()

from alerts.models import Alert
Alert.objects.all()

from scans.models import Scan
Scan.objects.all()

to delete :
from devices.models import Device
from alerts.models import Alert

Device.objects.all().delete()
Alert.objects.all().delete()

mvp update prompt:-
We are building a Django-based Network Monitoring System MVP for an offline/air-gapped intranet environment.

Current tech stack:

* Django
* PostgreSQL
* Service-layer architecture
* Fake scan data for testing (real Nmap integration postponed for now)

Goal of MVP:

* Simulate network scans
* Store devices
* Detect changes
* Generate alerts
* Validate monitoring logic before integrating real Nmap

What we have completed till now:

1. Django project setup completed

* PostgreSQL connected
* Apps created and registered:

  * devices
  * scans
  * scanner
  * alerts
  * dashboard
  * accounts

2. Core models created

* Device model
* Scan model
* Alert model

3. Device model stores:

* MAC address
* IP address
* hostname
* vendor
* operating system
* online status
* authorization status
* last seen timestamp

4. Alert model stores:

* alert type
* linked device
* description
* timestamp

5. Scan model stores:

* subnet scanned
* total devices found
* scan timestamp

6. Created fake scanner service

* Returns static Python list of fake devices
* Used instead of real Nmap for MVP testing

7. Created scan_processor service
   Current logic:

* Detect new devices
* Detect IP changes
* Detect OS changes
* Detect missing/offline devices
* Update current device state
* Generate alerts

8. Implemented baseline logic

* First scan acts as baseline
* No alerts generated on first scan
* Alerts only generated from future changes

9. Created Django management command:

* python manage.py runscan

Flow:

* fake scan runs
* scan metadata stored
* scan processed
* devices updated
* alerts generated

10. Django admin configured

* Device model visible
* Alert model visible
* Scan model visible

11. Testing completed successfully
    Verified:

* baseline scans work
* devices are stored
* alerts are generated correctly
* duplicate devices are not recreated
* online/offline logic works

Current project status:

* Core monitoring logic MVP is working successfully using fake scan data
* Real Nmap integration intentionally postponed until logic validation is fully complete

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

22/5/26

You are analyzing a Django-based network monitoring system project.

Project name: network-monitoring-system
Language: Python
Framework: Django 5.2.14
Database: PostgreSQL configured in `config/settings.py` (`network_monitor` on localhost:5432)
Dependencies in `requirements.txt`:
  - asgiref==3.11.1
  - Django==5.2.14
  - psycopg2-binary==2.9.12
  - python-nmap==0.7.1
  - sqlparse==0.5.5
  - typing_extensions==4.15.0
  - tzdata==2026.2

Root files:
  - `manage.py` : Django CLI entrypoint
  - `README.md` : project notes
  - `requirements.txt` : dependency list
  - `db.sqlite3` : local SQLite file present (project uses PostgreSQL settings)
  - `config/` : Django project config and routing

Key config:
  - `config/settings.py`
    - INSTALLED_APPS: accounts, devices, alerts, scans, scanner, dashboard
    - DEBUG = True
    - TIME_ZONE = Asia/Kolkata
    - DATABASES uses PostgreSQL
  - `config/urls.py`
  - `config/wsgi.py`
  - `config/asgi.py`

Django apps and responsibilities:
  - `accounts/` : user/account models, admin, views
  - `devices/` : device inventory models, admin
  - `alerts/` : alert models, views, alerting logic
  - `scans/` : scan records models and views
  - `scanner/` : scanning engine
      - `management/commands/runscan.py` : execute scans
      - `services/nmap_runner.py` : run nmap scans
      - `services/xml_parser.py` : parse nmap XML output
      - `services/normalizer.py` : normalize scan data
      - `services/scan_processor.py` : process parsed scans into database records
  - `dashboard/` : user-facing UI and templates
      - `dashboard/templates/dashboard/base.html`
      - `dashboard/templates/dashboard/index.html`
      - `dashboard/templates/dashboard/devices.html`
      - `dashboard/templates/dashboard/scans.html`
      - `dashboard/templates/dashboard/alerts.html`

Project purpose:
  - Perform network scanning with nmap
  - Parse and normalize scan results
  - Store devices, scan results, and alerts in a Django database
  - Provide a dashboard UI for monitoring network state
