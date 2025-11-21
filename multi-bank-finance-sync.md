# Multi-Bank Finance Management Solution

## Overview
Research findings for managing finances across:
- **Revolut** (banking)
- **Yuh** (Swiss banking)
- **Interactive Brokers** (investments)

## Platform Capabilities

### Revolut
**Export Options:**
- **CSV Export**: Available through app/web interface for personal accounts
- **Merchant API**: Full API access for business accounts with programmatic CSV report generation
- **Alternative Methods**: Third-party browser extensions and scripts for enhanced export

**Data Access:**
- Personal accounts: Manual CSV export or third-party tools
- Business accounts: Full REST API with custom report generation
- Export includes: transactions, categories, balances, currency data

### Yuh (Swiss Bank)
**Export Options:**
- **CSV Export**: Available via app (Account → Documents → Request → Account activities export)
- **API Access**: No public customer API currently available
- **Banking Provider**: Powered by Swissquote (FINMA authorized)

**Data Access:**
- Manual CSV export with customizable filters
- No programmatic API access for personal customers
- May need to contact Yuh support for API availability

### Interactive Brokers
**Export Options:**
- **Flex Web Service API**: Full programmatic access via HTTP API
- **Flex Queries**: Highly customizable report templates
- **Output Formats**: XML or TEXT/CSV

**Data Access:**
- Create custom Flex Query templates in Client Portal
- Two-step API process: trigger report generation, then download
- **Activity Flex**: Daily comprehensive data (end-of-day)
- **Trade Confirms Flex**: Real-time trade data

**Available Tools:**
- Python: `ibflex` library
- .NET: `ib-flex-reader`
- R: `IButils` package
- GitHub Actions: `ibkr-auto-exporter`

## Solution Options

### Option 1: Existing Platform - Kubera (Recommended for Quick Setup)

**Pros:**
- Direct integration with Interactive Brokers (via Yodlee)
- Multi-currency support
- All-in-one asset tracking (banking, investments, crypto)
- Professional interface with document storage
- Multiple aggregator partnerships for global connectivity

**Cons:**
- Paid service (subscription-based)
- Revolut integration may be limited (need to verify)
- Yuh likely requires manual CSV import
- Less control over data and automation

**Best For:** Users who want a ready-to-use solution and don't mind a subscription fee

### Option 2: Open Source Self-Hosted - Firefly III (Recommended for Privacy & Control)

**Pros:**
- Completely free and open source
- Self-hosted (full data control and privacy)
- Excellent CSV import capabilities
- Multi-currency support
- Active development and community
- REST API for custom integrations
- Docker deployment available

**Cons:**
- Requires technical setup (server/hosting)
- No direct bank API integrations (CSV-based)
- Manual import process (can be automated with scripts)
- Self-maintenance required

**Implementation:**
- Host Firefly III on a server (DigitalOcean, AWS, home server)
- Create automated scripts to:
  - Export Revolut transactions to CSV
  - Export Yuh transactions to CSV
  - Query Interactive Brokers Flex API
  - Import all data into Firefly III via its API

**Best For:** Privacy-conscious users with technical skills who want full control

### Option 3: Custom Solution (Recommended for Maximum Flexibility)

Build a custom sync system with:

**Architecture:**
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Revolut   │     │     Yuh     │     │ Interactive │
│   (CSV)     │     │   (CSV)     │     │   Brokers   │
│             │     │             │     │  (Flex API) │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                    │
       └───────────────────┴────────────────────┘
                           │
                    ┌──────▼──────┐
                    │   Sync      │
                    │   Engine    │
                    │  (Python)   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Database   │
                    │ (PostgreSQL)│
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Dashboard  │
                    │ (Web UI)    │
                    └─────────────┘
```

**Components:**
1. **Data Collectors:**
   - Revolut: Script to automate CSV export (or manual download)
   - Yuh: Script to automate CSV export (or manual download)
   - Interactive Brokers: Python script using `ibflex` library

2. **Sync Engine:**
   - Python application to normalize and import data
   - Scheduled execution (cron/systemd timer)
   - Deduplication and reconciliation logic
   - Multi-currency handling

3. **Storage:**
   - PostgreSQL or SQLite database
   - Normalized schema for transactions, accounts, balances
   - Historical data retention

4. **Dashboard:**
   - Web interface (React, Vue, or simple HTML)
   - Charts and analytics (Chart.js, D3.js)
   - Budget tracking and reporting
   - Export capabilities

**Technology Stack:**
- **Backend**: Python (FastAPI or Flask)
- **Database**: PostgreSQL or SQLite
- **Frontend**: React/Vue or simple HTML+JavaScript
- **Scheduling**: Cron or Python schedule library
- **Deployment**: Docker Compose for easy setup

**Pros:**
- Complete control and customization
- Can add any features you need
- Privacy-focused (self-hosted)
- No subscription fees
- Can integrate additional data sources easily

**Cons:**
- Requires significant development effort
- Ongoing maintenance
- Need technical skills in Python, databases, web development

**Best For:** Developers who want a personalized solution with specific requirements

### Option 4: Hybrid Approach (Balanced Solution)

Combine Firefly III with custom automation:

**Setup:**
1. Install Firefly III for core finance management
2. Build custom scripts to automate data collection:
   - Revolut CSV export automation
   - Yuh CSV export automation
   - Interactive Brokers API integration
3. Use Firefly III Data Importer or API to automatically import collected data
4. Schedule regular sync (daily/weekly)

**Pros:**
- Leverage mature open-source platform (Firefly III)
- Automated data collection
- Less development than full custom solution
- Self-hosted and private
- Extensible when needed

**Cons:**
- Still requires some scripting/automation work
- Server hosting needed
- Manual setup of importers

**Best For:** Users with moderate technical skills who want automation with a proven foundation

## Recommended Approach

Based on your requirements (Revolut, Yuh, Interactive Brokers), I recommend the **Hybrid Approach** (Option 4):

### Why?
1. **Firefly III** provides a solid, proven foundation for personal finance management
2. **Custom automation scripts** handle the specifics of your three platforms
3. **Interactive Brokers API** can be fully automated with existing Python libraries
4. **CSV exports** from Revolut and Yuh work well with Firefly III's importer
5. **Self-hosted** means complete privacy and control
6. **Open source** means no vendor lock-in and community support

### Implementation Steps

#### Phase 1: Setup Firefly III (1-2 hours)
```bash
# Using Docker Compose
git clone https://github.com/firefly-iii/docker.git firefly-iii
cd firefly-iii
cp env.example .env
# Edit .env with your settings
docker-compose up -d
```

#### Phase 2: Configure Accounts (30 minutes)
- Create Revolut account in Firefly III
- Create Yuh account in Firefly III
- Create Interactive Brokers investment account in Firefly III
- Set up currencies (EUR, CHF, USD, etc.)

#### Phase 3: Build Automation Scripts (4-6 hours)

**Structure:**
```
finance-sync/
├── collectors/
│   ├── revolut_collector.py
│   ├── yuh_collector.py
│   └── ibkr_collector.py
├── importers/
│   ├── firefly_importer.py
│   └── csv_normalizer.py
├── config/
│   ├── config.yaml
│   └── credentials.yaml (gitignored)
├── requirements.txt
└── sync.py (main orchestrator)
```

**revolut_collector.py:**
```python
# Either automated export using browser automation (Selenium/Playwright)
# Or manual: instructions to download CSV from Revolut app
def collect_revolut_data(output_path):
    # Download CSV from Revolut
    # Or use third-party tools like RevolVer
    pass
```

**yuh_collector.py:**
```python
# Download CSV from Yuh app
def collect_yuh_data(output_path):
    # Instructions or automation to export from Yuh
    # Account → Documents → Request → Account activities export
    pass
```

**ibkr_collector.py:**
```python
from ibflex import client, parser

def collect_ibkr_data(token, query_id, output_path):
    # Download data using Flex Query API
    response = client.download(token, query_id)
    statement = parser.parse(response)
    # Convert to CSV format
    return statement
```

**firefly_importer.py:**
```python
import requests

def import_to_firefly(csv_path, account_id, firefly_url, api_token):
    # Use Firefly III API to import transactions
    # POST /api/v1/transactions
    pass
```

**sync.py:**
```python
#!/usr/bin/env python3
import schedule
import time
from collectors import revolut_collector, yuh_collector, ibkr_collector
from importers import firefly_importer

def sync_all():
    print("Starting sync...")

    # Collect data from all sources
    revolut_collector.collect_revolut_data("data/revolut.csv")
    yuh_collector.collect_yuh_data("data/yuh.csv")
    ibkr_collector.collect_ibkr_data("data/ibkr.csv")

    # Import to Firefly III
    firefly_importer.import_all("data/")

    print("Sync completed!")

if __name__ == "__main__":
    # Run immediately
    sync_all()

    # Schedule daily sync
    schedule.every().day.at("06:00").do(sync_all)

    while True:
        schedule.run_pending()
        time.sleep(60)
```

#### Phase 4: Schedule Automation (30 minutes)
```bash
# Option 1: Cron job
0 6 * * * /usr/bin/python3 /path/to/sync.py

# Option 2: Systemd timer
# Create service and timer files

# Option 3: Run sync.py as daemon with schedule library
```

#### Phase 5: Monitor and Refine (ongoing)
- Check sync logs
- Verify transaction imports
- Add error handling
- Implement notifications (email/Telegram)

## Alternative Tools to Consider

### Other Open Source Options:
- **Paisa**: Plaintext accounting with CSV import
- **Denaro**: Modern UI, excellent CSV support
- **MoneyManager Ex**: Desktop application, multi-platform
- **GnuCash**: Full-featured accounting software

### Commercial Options:
- **Quicken**: Desktop software with some bank integrations
- **YNAB (You Need A Budget)**: Popular budgeting tool (limited bank connections)
- **Monarch Money**: Modern Mint alternative (US-focused)

## Security Considerations

1. **API Credentials**: Store securely (use environment variables or encrypted vaults)
2. **CSV Files**: Delete after import or encrypt storage
3. **Firefly III Access**: Use strong passwords, enable 2FA if available
4. **HTTPS**: Always use SSL/TLS for web access
5. **Backups**: Regular encrypted backups of database
6. **Network**: Host on private network or use VPN

## Cost Analysis

### Firefly III + Automation (Hybrid):
- Hosting: $5-10/month (VPS) or $0 (self-hosted on existing hardware)
- Development time: ~10 hours initial setup
- Maintenance: ~1 hour/month
- **Total**: $5-10/month + time investment

### Kubera:
- Subscription: ~$100-150/year
- Setup time: ~1 hour
- Maintenance: Minimal
- **Total**: ~$10-12/month + minimal time

### Custom Solution:
- Hosting: $5-10/month (VPS)
- Development time: ~40-60 hours initial build
- Maintenance: ~2-3 hours/month
- **Total**: $5-10/month + significant time investment

## Next Steps

1. **Decide on approach** based on technical skills, time, and privacy preferences
2. **Test CSV exports** from Revolut and Yuh to understand data formats
3. **Set up Interactive Brokers Flex Query** for automated data export
4. **Choose hosting** (cloud VPS, home server, or local Docker)
5. **Start with Firefly III** as foundation
6. **Build collectors** one platform at a time
7. **Automate imports** and schedule regular syncs
8. **Monitor and improve** over time

## Resources

### Documentation:
- [Firefly III Documentation](https://docs.firefly-iii.org/)
- [Interactive Brokers Flex Web Service](https://www.interactivebrokers.com/campus/ibkr-api-page/flex-web-service/)
- [ibflex Python Library](https://github.com/csingley/ibflex)

### Tools:
- [Firefly III Data Importer](https://github.com/firefly-iii/data-importer)
- [RevolVer (Revolut exporter)](https://github.com/Tomasinjo/RevolVer)
- [IBKR Auto Exporter](https://github.com/jefrnc/ibkr-auto-exporter)

Would you like me to help you implement any of these solutions?
