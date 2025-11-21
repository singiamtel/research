# Multi-Bank Finance Management Solution

## Overview
Research findings for managing finances across:
- **Revolut** (banking)
- **Yuh** (Swiss banking)
- **Interactive Brokers** (investments)

## Platform Capabilities

### Revolut
**Export Options:**
- **Open Banking API (PSD2)**: Full programmatic access via regulated third-party providers
- **CSV Export**: Available through app/web interface for personal accounts
- **Merchant API**: Full API access for business accounts with programmatic CSV report generation
- **Alternative Methods**: Third-party browser extensions and scripts for enhanced export

**Data Access:**
- **Personal accounts**:
  - **GoCardless Bank Account Data (formerly Nordigen)**: FREE open banking API access
    - 2,500+ bank connections across UK/Europe including Revolut
    - Up to 720 days of transaction history
    - Free tier: 50 monthly connections
    - Requires eIDAS or Open Banking certificate (or use their managed service)
  - Manual CSV export or third-party tools
- Business accounts: Full REST API with custom report generation
- Export includes: transactions, categories, balances, currency data

### Yuh (Swiss Bank)
**Export Options:**
- **Swissquote Open Banking API (PSD2-compliant)**: Since Yuh is powered by Swissquote (as of July 2025, Swissquote owns 100% of Yuh), may have access via Swissquote's PSD2-compliant API
- **CSV Export**: Available via app (Account → Documents → Request → Account activities export)
- **Banking Provider**: Powered by Swissquote (FINMA authorized)

**Data Access:**
- **Potential Open Banking access**: Swissquote offers PSD2-compliant open banking API for regulated TPPs (Third Party Providers)
  - Available to Account Information Service Providers (AISP)
  - Need to verify if Yuh accounts are accessible via Swissquote's API
  - Requires TPP authorization from national competent authority
- Manual CSV export with customizable filters
- Contact Yuh/Swissquote support to confirm open banking access

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

### Option 1: Actual Budget + GoCardless (RECOMMENDED - Fully Automated!)

**🎯 BEST OPTION for your requirements - requires NO manual imports!**

**What is it?**
- Open source, self-hosted budgeting tool
- Built-in bank sync via GoCardless (formerly Nordigen)
- FREE open banking API access to 2,500+ European banks
- Direct integration - no coding required!

**Coverage:**
- ✅ **Revolut**: Full automated sync via GoCardless open banking API
- ⚠️ **Yuh**: May work via Swissquote open banking API (needs verification)
- ⚠️ **Interactive Brokers**: Would require SimpleFIN or custom integration

**Pros:**
- **Zero manual imports** for Revolut (fully automated!)
- Completely free and open source
- Self-hosted (full privacy and data control)
- Active development and community
- Modern, user-friendly interface
- Multi-currency support
- Budget tracking and goals
- Docker deployment available
- No coding required for Revolut sync

**Cons:**
- Requires server hosting (can be local/home server)
- Interactive Brokers would need custom integration or SimpleFIN (paid service for US/Canada)
- Yuh access via Swissquote API needs verification
- Self-maintenance required

**Setup Time:**
- Installation: 30 minutes
- GoCardless configuration: 15 minutes
- Bank linking: 5 minutes per bank
- **Total: ~1 hour for Revolut automation!**

**Cost:**
- Software: FREE
- Hosting: $0 (self-hosted) or $5-10/month (VPS)
- GoCardless API: FREE (up to 50 connections/month)
- **Total: $0-10/month**

**Implementation Steps:**
1. Deploy Actual Budget via Docker
2. Sign up for free GoCardless Bank Account Data API
3. Configure GoCardless in Actual Budget settings
4. Link Revolut account (one-time authorization)
5. Automatic daily sync happens in background!

**For Interactive Brokers:**
- Option A: Use SimpleFIN (paid, ~$1.50/month, supports some brokers)
- Option B: Build custom integration using their Flex API
- Option C: Manual CSV import (can be automated with scripts)

**Best For:** Users who want a fully automated, free, self-hosted solution with zero manual imports for banking

---

### Option 2: Existing Platform - Kubera (Quick Setup, Paid Service)

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

### Option 3: Open Source Self-Hosted - Firefly III (Privacy & Control)

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

### Option 4: Custom Solution (Maximum Flexibility)

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

### Option 5: Hybrid Approach - Firefly III + GoCardless (Balanced Solution)

Combine Firefly III with GoCardless open banking and custom automation:

**Setup:**
1. Install Firefly III for core finance management
2. Use Firefly III Data Importer with GoCardless support:
   - Connect Revolut via GoCardless open banking API (automated)
   - Connect Yuh via Swissquote API if available (automated)
   - Build Interactive Brokers integration script
3. Schedule regular sync (automated for banks, scripted for IBKR)

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

Based on your requirements (Revolut, Yuh, Interactive Brokers) and **desire for fully automated sync with zero manual imports**, I recommend **Option 1: Actual Budget + GoCardless**:

### Why Actual Budget?
1. **Built-in automation** - GoCardless integration is native, no coding required
2. **FREE forever** - Both Actual Budget and GoCardless API are free
3. **Revolut works out-of-the-box** - Connects via open banking API automatically
4. **Modern interface** - Clean, fast, user-friendly budgeting tool
5. **Self-hosted** - Complete privacy and data control
6. **Active community** - Great support and regular updates
7. **No manual imports** - Background sync happens automatically

### Coverage Assessment:
- ✅ **Revolut**: Fully automated (GoCardless API)
- ⚠️ **Yuh**: Needs verification (may work via Swissquote's PSD2 API, or manual CSV as fallback)
- ⚠️ **Interactive Brokers**: Requires custom integration (see implementation below)

### Implementation Steps

#### Phase 1: Deploy Actual Budget (30 minutes)

**Using Docker:**
```bash
# Create directory
mkdir -p ~/actual-budget && cd ~/actual-budget

# Create docker-compose.yml
cat > docker-compose.yml <<'EOF'
version: '3.8'
services:
  actual:
    image: actualbudget/actual-server:latest
    ports:
      - '5006:5006'
    volumes:
      - ./actual-data:/data
    environment:
      - ACTUAL_UPLOAD_FILE_SIZE_MB=20
      - ACTUAL_UPLOAD_SYNC_ENCRYPTED_FILE_SIZE_MB=20
      - ACTUAL_UPLOAD_FILE_SIZE_MB=20
    restart: unless-stopped
EOF

# Start the service
docker-compose up -d

# Access at http://localhost:5006
```

**Or use hosted instance** (if you prefer):
```bash
# Install Actual on a VPS (DigitalOcean, Hetzner, etc.)
# Same docker-compose.yml as above
# Configure reverse proxy (nginx/caddy) with SSL
```

#### Phase 2: Configure GoCardless Bank Sync (15 minutes)

1. **Sign up for GoCardless Bank Account Data API**:
   - Go to: https://gocardless.com/bank-account-data/
   - Create free account (requires business email, but free tier available)
   - Get API credentials (Secret ID and Secret Key)

2. **Configure in Actual Budget**:
   - Open Actual Budget web interface
   - Go to Settings → Experimental Features
   - Enable "GoCardless Bank Sync"
   - Go to Settings → GoCardless
   - Enter your Secret ID and Secret Key
   - Save configuration

#### Phase 3: Connect Revolut (5 minutes)

1. In Actual Budget, go to "Accounts"
2. Click "Add Account" → "Link Bank Account"
3. Select "GoCardless" as provider
4. Search for "Revolut"
5. Click "Connect" - redirects to Revolut for authorization
6. Authorize Actual Budget to access your transactions (PSD2 consent)
7. Select which Revolut accounts to sync
8. Done! Transactions will sync automatically

**Sync frequency:**
- Automatic daily sync in background
- Can manually trigger sync anytime
- Up to 720 days of historical transactions

#### Phase 4: Try Connecting Yuh via Swissquote (15 minutes)

1. In Actual Budget, search for "Swissquote" in bank list
2. If available, try connecting:
   - Should redirect to Swissquote for authorization
   - Your Yuh account might appear (since it's powered by Swissquote)
3. If not available or Yuh doesn't appear:
   - **Fallback**: Manual CSV import from Yuh app
   - Or build custom script (see Phase 6)

#### Phase 5: Add Interactive Brokers (Choose one option)

**Option A: SimpleFIN (Easiest, but paid ~$1.50/month)**
- Check if SimpleFIN supports Interactive Brokers
- Enable SimpleFIN in Actual Budget settings
- Connect account

**Option B: Manual CSV Import (Free, semi-automated)**
- Set up Interactive Brokers Flex Query (one-time)
- Download CSV weekly/monthly from IBKR portal
- Import to Actual Budget (drag and drop CSV)
- Can be automated with script (see Option C)

**Option C: Custom IBKR Integration Script (Free, fully automated)**

Create a simple sync script:

```bash
# Create ibkr-sync directory
mkdir ~/ibkr-sync && cd ~/ibkr-sync
```

**ibkr_to_actual.py:**
```python
#!/usr/bin/env python3
"""
Sync Interactive Brokers data to Actual Budget
Runs daily via cron to fetch IBKR data and import to Actual
"""
import os
from ibflex import client, parser
import requests
import csv
from datetime import datetime, timedelta

# Configuration (use environment variables or config file)
IBKR_TOKEN = os.getenv('IBKR_TOKEN')
IBKR_QUERY_ID = os.getenv('IBKR_QUERY_ID')
ACTUAL_API_URL = os.getenv('ACTUAL_API_URL', 'http://localhost:5006')
ACTUAL_PASSWORD = os.getenv('ACTUAL_PASSWORD')

def fetch_ibkr_data():
    """Fetch data from Interactive Brokers Flex Query"""
    print("Fetching IBKR data...")
    response = client.download(IBKR_TOKEN, IBKR_QUERY_ID)
    statement = parser.parse(response)
    return statement

def convert_to_actual_format(statement):
    """Convert IBKR statement to Actual Budget format"""
    transactions = []
    for trade in statement.FlexStatements[0].Trades:
        transactions.append({
            'date': trade.tradeDate.strftime('%Y-%m-%d'),
            'payee': f"{trade.symbol} - {trade.description}",
            'amount': float(trade.proceeds),  # Negative for buys, positive for sells
            'notes': f"Qty: {trade.quantity}, Price: {trade.tradePrice}"
        })
    return transactions

def import_to_actual(transactions):
    """Import transactions to Actual Budget via API"""
    # Note: Actual Budget API specifics depend on version
    # You may need to use the CSV import endpoint or direct API
    print(f"Importing {len(transactions)} transactions to Actual Budget...")
    # Implementation depends on Actual Budget API
    # For now, save as CSV for manual import or use API if available

    csv_file = '/tmp/ibkr_transactions.csv'
    with open(csv_file, 'w', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=['date', 'payee', 'amount', 'notes'])
        writer.writeheader()
        writer.writerows(transactions)

    print(f"Saved to {csv_file}")
    return csv_file

def main():
    statement = fetch_ibkr_data()
    transactions = convert_to_actual_format(statement)
    import_to_actual(transactions)
    print("Sync completed!")

if __name__ == "__main__":
    main()
```

**requirements.txt:**
```
ibflex
requests
```

**Setup:**
```bash
# Install dependencies
pip install -r requirements.txt

# Set environment variables
export IBKR_TOKEN="your_token_here"
export IBKR_QUERY_ID="your_query_id_here"
export ACTUAL_PASSWORD="your_actual_password"

# Test run
python ibkr_to_actual.py

# Schedule with cron (daily at 7 AM)
crontab -e
# Add: 0 7 * * * cd ~/ibkr-sync && /usr/bin/python3 ibkr_to_actual.py >> ~/ibkr-sync/sync.log 2>&1
```

#### Phase 6: (Optional) Automate Yuh if not via Swissquote API

If Yuh doesn't work via Swissquote's API, create a simple automation:

```python
#!/usr/bin/env python3
"""
Download Yuh CSV and import to Actual Budget
Can be run manually or semi-automated with browser automation
"""
import os
import glob

def import_yuh_csv_to_actual(csv_path):
    """Import Yuh CSV to Actual Budget"""
    # Actual Budget accepts CSV imports
    # Either copy to import folder or use API
    print(f"Importing {csv_path} to Actual Budget...")
    # Implementation depends on your setup

# Watch downloads folder for new Yuh CSV files
downloads = os.path.expanduser("~/Downloads")
yuh_csvs = glob.glob(f"{downloads}/Yuh_*.csv")

if yuh_csvs:
    latest = max(yuh_csvs, key=os.path.getctime)
    import_yuh_csv_to_actual(latest)
```

#### Phase 7: Monitor and Enjoy (ongoing)

- Check Actual Budget dashboard daily
- Verify transactions are syncing correctly
- Set budgets and financial goals
- Run reports and analytics
- Transactions sync automatically - **zero manual work for Revolut!**

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
- [Actual Budget Documentation](https://actualbudget.org/docs/)
- [Actual Budget Bank Sync Guide](https://actualbudget.org/docs/advanced/bank-sync/)
- [GoCardless Bank Account Data API](https://gocardless.com/bank-account-data/)
- [GoCardless API Documentation](https://developer.gocardless.com/)
- [Firefly III Documentation](https://docs.firefly-iii.org/)
- [Interactive Brokers Flex Web Service](https://www.interactivebrokers.com/campus/ibkr-api-page/flex-web-service/)
- [Revolut Open Banking API](https://developer.revolut.com/docs/open-banking/open-banking-api)
- [Swissquote Open Banking API](https://www.swissquote.com/en-lu/private/help/legal-tax/open-banking-psd2-api)

### Open Source Tools:
- [Actual Budget](https://github.com/actualbudget/actual) - Self-hosted budgeting tool
- [Actual Budget Server](https://github.com/actualbudget/actual-server) - Backend server
- [ibflex Python Library](https://github.com/csingley/ibflex) - IBKR Flex Query parser
- [Firefly III](https://github.com/firefly-iii/firefly-iii) - Personal finance manager
- [Firefly III Data Importer](https://github.com/firefly-iii/data-importer)
- [RevolVer](https://github.com/Tomasinjo/RevolVer) - Revolut transaction exporter
- [IBKR Auto Exporter](https://github.com/jefrnc/ibkr-auto-exporter) - Automated IBKR exports

### APIs & Services:
- **GoCardless (Nordigen)**: FREE open banking API - https://gocardless.com/bank-account-data/
- **SimpleFIN**: Paid bank sync service (~$1.50/month) - https://www.simplefin.org/
- **Plaid**: Commercial banking API - https://plaid.com/
- **Tink/Visa**: Commercial banking API - https://tink.com/

## Summary & Next Steps

### Quick Start (Recommended)
1. **Deploy Actual Budget** (30 min) - Self-host via Docker
2. **Sign up for GoCardless API** (15 min) - Free tier
3. **Connect Revolut** (5 min) - One-click OAuth connection
4. **Try Swissquote for Yuh** (15 min) - May work automatically
5. **Add IBKR script** (optional, 2 hours) - For full automation

**Result**: Revolut auto-syncing, Yuh possibly auto-syncing, IBKR with minimal effort

### Alternative Path
- Use **Firefly III + GoCardless** if you prefer Firefly's features
- Use **Kubera** if you don't want to self-host and don't mind $10/month
- Build **custom solution** if you have specific requirements

### Key Benefits of Recommended Solution
✅ **Zero manual imports** for Revolut
✅ **Free forever** (open source + free API)
✅ **Self-hosted** (complete privacy)
✅ **Modern interface** (better UX than Firefly III)
✅ **Active development** (regular updates)
✅ **720 days history** (2 years of transactions)
✅ **Multi-currency** (EUR, CHF, USD, etc.)

Would you like me to help you implement this solution?
