---
name: tenable-one-api-claude-skill
description: >
  Tenable One exposure management platform expertise. Use when the user wants to
  interact with the Tenable API, manage vulnerabilities, export scan data, manage
  assets and tags, work with exposure metrics, build integrations, automate
  security workflows, or write scripts using pyTenable. Covers Vulnerability
  Management, Web App Scanning, Identity Exposure, Security Center, OT Security,
  Attack Surface Management, Exposure Management, and PCI ASV.
---

# Tenable One API Claude Skill

You are an expert in the Tenable One platform, its REST APIs, Python SDK (pyTenable), and security operations automation.

## Platform Overview

Tenable One is a unified exposure management platform that consolidates:

| Module | Purpose | API Base |
|--------|---------|----------|
| **Vulnerability Management** (VM) | Network & host vulnerability scanning | `cloud.tenable.com` |
| **Web App Scanning** (WAS) | DAST for web applications | `cloud.tenable.com` |
| **Identity Exposure** (IE) | Active Directory security monitoring (formerly Alsid) | `cloud.tenable.com` |
| **OT Security** | ICS/SCADA/OT network visibility & threat detection | On-prem appliance (GraphQL) |
| **Attack Surface Management** (ASM) | External attack surface discovery | `cloud.tenable.com` |
| **Exposure Management** | Unified exposure scoring, attack path analysis | `cloud.tenable.com` |
| **Security Center** (SC) | On-prem vulnerability management console | On-prem (`/rest/`) |
| **Platform & Settings** | Access control, agents, connectors, networks, tags | `cloud.tenable.com` |
| **PCI ASV** | PCI-DSS quarterly external scan compliance | `cloud.tenable.com` |
| **MSSP Portal** | Multi-tenant management for MSSPs | `cloud.tenable.com` |
| **Downloads** | Nessus scanner/agent/plugin downloads | `www.tenable.com/downloads/api/v2` |

---

## Authentication

All API calls require the `X-ApiKeys` header:

```
X-ApiKeys: accessKey={ACCESS_KEY};secretKey={SECRET_KEY}
```

### Key Rules
- API keys are per-user and inherit that user's permissions.
- Keys are generated in **Tenable Vulnerability Management > Settings > My Account > API Keys**.
- Or programmatically via `PUT /users/{user_id}/keys`.
- NEVER log or expose API keys in output. Always use environment variables:
  ```bash
  export TIO_ACCESS_KEY="your-access-key"
  export TIO_SECRET_KEY="your-secret-key"
  ```

### Request Headers
Every request MUST include:
```
Content-Type: application/json
Accept: application/json
X-ApiKeys: accessKey={ACCESS_KEY};secretKey={SECRET_KEY}
User-Agent: YourIntegration/1.0
```

---

## API Base URLs

| Environment | URL |
|-------------|-----|
| **US Cloud** | `https://cloud.tenable.com` |
| **US Cloud (FedRAMP)** | `https://fedcloud.tenable.com` |
| **EU Cloud** | `https://cloud.tenable.com` (same, region-aware) |

> **Tip:** Tenable also publishes an LLM-friendly index at `https://developer.tenable.com/llms.txt` and OpenAPI specs at `https://developer.tenable.com/reference/download-the-specs`.

---

## Rate & Concurrency Limits

| Limit Type | Value | Notes |
|------------|-------|-------|
| Rate limit | **Variable** — typically 200-300 req/min | Returns `429 Too Many Requests` |
| Retry header | `Retry-After` | Seconds to wait before retry |
| Export concurrency | **Up to 2 concurrent exports** per type (asset/vuln) | Additional requests queue |
| Chunk download | Chunks expire after **24 hours** | Re-request export if expired |

### Retry Strategy
```python
import time, requests

def api_call(method, url, headers, **kwargs):
    for attempt in range(5):
        resp = requests.request(method, url, headers=headers, **kwargs)
        if resp.status_code == 429:
            wait = int(resp.headers.get("Retry-After", 2 ** attempt))
            time.sleep(wait)
            continue
        resp.raise_for_status()
        return resp.json()
    raise Exception("Rate limit exceeded after 5 retries")
```

---

## Core API Patterns

### Platform & Settings

The Platform & Settings API is the cross-cutting management layer for access control, connectors, and infrastructure:

```
# Access Control (v3)
POST /api/v3/access-control/permissions       # Create permission
GET  /api/v3/access-control/permissions        # List permissions
DELETE /api/v3/access-control/permissions/{id} # Delete permission

# Agents
GET  /scanners/{scanner_id}/agents             # List agents
GET  /scanners/{scanner_id}/agent-groups        # List agent groups
POST /scanners/{scanner_id}/agent-groups        # Create agent group
PUT  /scanners/{scanner_id}/agent-groups/{id}/agents/{agent_id}  # Add agent to group

# Connectors (cloud import: AWS, Azure, GCP, etc.)
GET  /settings/connectors                       # List connectors
POST /settings/connectors                       # Create connector

# Exclusions
GET  /exclusions                                # List exclusions
POST /exclusions                                # Create exclusion

# Networks
GET  /networks                                  # List networks
POST /networks                                  # Create network
GET  /networks/{id}/scanners                    # List scanners in network
POST /networks/{id}/scanners/{scanner_uuid}     # Assign scanner to network

# Scanners
GET  /scanners                                  # List scanners
GET  /scanners/{id}                             # Get scanner details
```

### Downloads API

The Downloads API lets you programmatically download Nessus scanners, agents, and plugins:

```
# Base URL: https://www.tenable.com/downloads/api/v2

GET /pages                                      # List all downloadable products
GET /pages/{page_slug}                          # Get product download details
GET /pages/{page_slug}/files/{file_name}        # Download a specific file
```

Common page slugs: `nessus`, `nessus-agents`, `tenable-sc`, `tenable-sc-nessus-plugins`

---

### 1. Vulnerability Export (Chunked Async)

This is the **primary** way to extract vulnerability data at scale.

```
POST /vulns/export
→ Returns { "export_uuid": "..." }

GET /vulns/export/{export_uuid}/status
→ Returns { "status": "FINISHED", "chunks_available": [1, 2, 3] }

GET /vulns/export/{export_uuid}/chunks/{chunk_id}
→ Returns array of vulnerability objects
```

#### Export Request Body
```json
{
  "filters": {
    "severity": ["critical", "high"],
    "state": ["open", "reopened"],
    "since": 1700000000
  },
  "num_assets": 500
}
```

#### Filter Options
| Filter | Type | Values |
|--------|------|--------|
| `severity` | array | `info`, `low`, `medium`, `high`, `critical` |
| `state` | array | `open`, `reopened`, `fixed` |
| `since` | int | Unix timestamp — only vulns found/updated after |
| `cidr_range` | string | CIDR notation, e.g. `10.0.0.0/8` |
| `first_found` | int | Unix timestamp |
| `last_found` | int | Unix timestamp |
| `last_fixed` | int | Unix timestamp |
| `plugin_id` | array | Nessus plugin IDs |
| `vpr_score` | object | `{ "gte": 7.0 }` |
| `tags` | array | Tag category:value pairs |

### 2. Asset Export (Chunked Async)

Tenable provides two versions of the Asset Export API:
* **Asset Export v2 (Recommended):** `POST /api/v2/assets/export`. Supports both Host and Web App Scanning (WAS) assets, returns a cleaner data model, and uses the `ratings` object instead of deprecated legacy score fields.
* **Asset Export v1 (Legacy):** `POST /assets/export`.

#### V2 Asset Export API (Recommended)
```
POST /api/v2/assets/export
→ Returns { "export_uuid": "..." }

GET /api/v2/assets/export/{export_uuid}/status
GET /api/v2/assets/export/{export_uuid}/chunks/{chunk_id}
```

##### V2 Request Body & Filters
```json
{
  "filters": {
    "since": 1700000000,
    "types": ["host", "webapp"],
    "sources": ["NESSUS_SCAN", "WAS"],
    "tags": ["Location:NYC"],
    "is_licensed": true,
    "is_terminated": false,
    "is_deleted": false
  },
  "chunk_size": 1000,
  "include_resource_tags": true
}
```

#### V1 Asset Export API (Legacy)
```
POST /assets/export
→ Returns { "export_uuid": "..." }

GET /assets/export/{export_uuid}/status
GET /assets/export/{export_uuid}/chunks/{chunk_id}
```

### 3. Scan Management

```
# List scans
GET /scans

# Create a scan
POST /scans
{
  "uuid": "{template_uuid}",
  "settings": {
    "name": "Weekly Network Scan",
    "text_targets": "10.0.0.0/24",
    "enabled": true,
    "scanner_id": 1,
    "launch": "ON_DEMAND",
    "rrules": "FREQ=WEEKLY;INTERVAL=1;BYDAY=MO"
  }
}

# Launch a scan
POST /scans/{scan_id}/launch

# Check scan status
GET /scans/{scan_id}

# Export scan results
POST /scans/{scan_id}/export
{ "format": "nessus" }  # or "csv", "pdf", "html", "db"
```

### 4. Asset & Tag Management

```
# List assets
GET /assets

# Get asset details
GET /assets/{asset_uuid}

# Create a tag category + value
POST /tags/values
{
  "category_name": "Environment",
  "value": "Production",
  "description": "Production environment assets"
}

# Assign tags to assets (bulk)
POST /tags/assets/assignments
{
  "action": "add",
  "assets": ["{asset_uuid_1}", "{asset_uuid_2}"],
  "tags": ["{tag_value_uuid}"]
}
```

### 5. Exposure Management (Exposure View)

```
# Search Exposure View cards
POST /api/v3/exposure-management/exposure-view/cards/search
{
  "limit": 50,
  "sort": [{"property": "name", "order": "asc"}]
}

# Get card details (includes CES / trend data)
GET /api/v3/exposure-management/exposure-view/cards/{card_id}
```

### 6. Inventory Search (Exposure Management)

```
# Search assets in inventory
POST /api/v3/assets/search
{
  "filter": {
    "and": [
      {"property": "types", "operator": "eq", "value": "host"}
    ]
  },
  "limit": 200,
  "sort": [{"property": "name", "order": "asc"}]
}

# Search findings
POST /api/v3/findings/search

# Search software
POST /api/v3/software/search
```

### 7. Attack Path Analysis

```
# Search top attack paths
POST /api/v3/attack-path/paths/search

# Search attack techniques
POST /api/v3/attack-path/techniques/search

# Export MITRE ATT&CK heatmap
POST /api/v3/attack-path/export/mitre-heatmap
```

### 8. Web App Scanning (WAS)

```
# List WAS scans
GET /was/v2/scans

# Create WAS scan configuration
POST /was/v2/configs
{
  "name": "My Web App Scan",
  "target": "https://example.com",
  "template_id": "{was_template_uuid}"
}

# Launch WAS scan
POST /was/v2/scans
{ "config_id": "{config_id}" }

# Get WAS scan status
GET /was/v2/scans/{scan_id}

# Get WAS vulnerabilities
GET /was/v2/scans/{scan_id}/vulnerabilities
```

---

## Key Tenable Terminology

| Term | Definition |
|------|-----------|
| **VPR** | Vulnerability Priority Rating (0–10). Tenable's proprietary risk score that factors in threat intelligence, exploit maturity, and age. More actionable than CVSS alone. |
| **ACR** | Asset Criticality Rating (1–10). Measures business criticality of an asset. |
| **AES** | Asset Exposure Score. Combines VPR of vulns on an asset with the asset's ACR. |
| **CES** | Cyber Exposure Score (0–1000). Organization-wide exposure metric in Exposure View. |
| **Lumin** | Legacy add-on product (end-of-sale) that provides exposure analytics. Functionalities now part of Tenable One Exposure Management. |
| **Plugin ID** | Unique identifier for a Nessus check/test (e.g., Plugin 19506 = "Nessus Scan Information"). |
| **Severity** | `info` (0), `low` (1), `medium` (2), `high` (3), `critical` (4) |
| **State** | Vulnerability state: `open`, `reopened`, `fixed` |
| **Agent** | Nessus Agent — lightweight scanner installed on endpoints. Reports to cloud. |
| **Scanner** | Nessus scanner appliance (cloud-linked or on-prem). |
| **Network** | Logical grouping for scanners and scan zones in VM. |
| **Tag** | Category:Value pair for organizing assets (e.g., `Environment:Production`). |
| **Connector** | Integration that imports assets from AWS, Azure, GCP, etc. |

---

## Product Branding (Current → Legacy)

Tenable rebranded its product line. Always use **current names** in documentation and output. The legacy names still appear in pyTenable class names, API paths, and older docs.

| Current Name | Legacy Name | pyTenable Class | Notes |
|-------------|-------------|-----------------|-------|
| **Tenable Vulnerability Management** | Tenable.io | `TenableIO` | `from tenable.io import TenableIO` |
| **Tenable Security Center** | Tenable.sc, SecurityCenter | `TenableSC` | `from tenable.sc import TenableSC` |
| **Tenable OT Security** | Tenable.ot, Indegy | `TenableOT` | `from tenable.ot import TenableOT` |
| **Tenable Identity Exposure** | Tenable.ad, Alsid | `TenableIE` | `from tenable.ie import TenableIE` |
| **Tenable Cloud Security** | Tenable.cs | `CloudSecurity` | CNAPP — API docs in-product only |
| **Tenable Web App Scanning** | Tenable.io WAS | (part of `TenableIO`) | Endpoints under `/was/v2/` |
| **Tenable Attack Surface Management** | Tenable.asm | (part of `TenableIO`) | Endpoints under `/asm/` |
| **Tenable Lumin** | Lumin | (part of `TenableIO`) | EoS add-on; features rolled into Tenable One Exposure Management. |
| **Tenable One** | — | — | The Exposure Management platform. |

> **Important:** pyTenable class names (`TenableIO`, `TenableSC`, etc.) still use the legacy naming. This is correct — don't rename them in code.

---

## pyTenable (Python SDK)

### Installation
```bash
pip install pytenable
```

### Initialization
```python
from tenable.io import TenableIO

tio = TenableIO(
    access_key='YOUR_ACCESS_KEY',
    secret_key='YOUR_SECRET_KEY',
    # vendor='YourCompany',        # optional User-Agent prefix
    # product='YourIntegration',   # optional
)
```

> **Best practice:** Use environment variables:
> ```python
> import os
> tio = TenableIO(
>     access_key=os.environ['TIO_ACCESS_KEY'],
>     secret_key=os.environ['TIO_SECRET_KEY'],
> )
> ```

### Common Operations

#### Export Vulnerabilities
```python
# Export all critical/high open vulns
vulns = tio.exports.vulns(
    severity=['critical', 'high'],
    state=['open', 'reopened'],
    num_assets=500,
)
for vuln in vulns:
    print(f"{vuln['plugin']['id']}: {vuln['plugin']['name']} "
          f"(VPR: {vuln.get('plugin', {}).get('vpr', {}).get('score', 'N/A')})")
```

#### Export Assets (V2 - Recommended)
```python
# V2 returns nested 'ratings' and 'network' objects, and supports WAS assets
assets = tio.exports.assets_v2(
    chunk_size=1000,
    include_resource_tags=True,
)
for asset in assets:
    network = asset.get('network', {})
    fqdns = network.get('fqdns', [])
    ipv4s = network.get('ipv4s', [])
    
    ratings = asset.get('ratings', {})
    acr = ratings.get('acr', {}).get('score', 'N/A')
    aes = ratings.get('aes', {}).get('score', 'N/A')
    print(f"{asset['id']}: {fqdns} / {ipv4s} (ACR: {acr}, AES: {aes})")
```

#### Export Assets (V1 - Legacy)
```python
assets = tio.exports.assets(
    is_licensed=True,
    chunk_size=1000,
)
for asset in assets:
    # V1 returns acr_score and exposure_score as root fields (deprecated)
    print(f"{asset['id']}: {asset.get('fqdns', [])} (ACR: {asset.get('acr_score')})")
```

#### Manage Scans
```python
# List scans
for scan in tio.scans.list():
    print(f"{scan['id']}: {scan['name']} ({scan['status']})")

# Create and launch a scan
scan = tio.scans.create(
    name='API-Created Scan',
    targets=['10.0.0.0/24'],
    template='basic',
)
tio.scans.launch(scan['id'])
```

#### Manage Tags
```python
# Create a tag
tag = tio.tags.create('Environment', 'Production')

# Assign to assets
tio.tags.assign(
    assets=[asset_uuid_1, asset_uuid_2],
    tags=[tag['uuid']],
)

# List all tag values
for tv in tio.tags.list():
    print(f"{tv['category_name']}:{tv['value']}")
```

#### Agent Management
```python
# List agents
for agent in tio.agents.list():
    print(f"{agent['name']} - {agent['status']} - {agent['ip']}")
```

---

## navi — Community CLI Tool

`navi` is a popular community CLI tool for querying Tenable data without writing code:

```bash
pip install navi-pro

# Configure
navi keys --a $TIO_ACCESS_KEY --s $TIO_SECRET_KEY

# Download data to local SQLite DB
navi update full          # All data
navi update assets        # Assets only
navi update vulns         # Vulnerabilities only

# Query with SQL
navi explore data query "SELECT * FROM assets WHERE operating_system LIKE '%Linux%';"
navi explore data query "SELECT plugin_id, plugin_name, COUNT(*) as count FROM vulns WHERE severity = 'Critical' GROUP BY plugin_id ORDER BY count DESC LIMIT 20;"
```

> **Note:** `navi` is community-maintained, not officially supported by Tenable. Also see **navi-mcp** below for MCP server integration.

---

## MCP Servers (Model Context Protocol)

MCP servers allow AI clients (Claude Desktop, Claude Code, Cursor, VS Code Copilot) to interact with Tenable directly through structured tools.

### Tenable Hexa AI MCP Server (Official)

The official Tenable-hosted MCP server, exposing **90 structured tools** from the Exposure Data Fabric. No local installation required.

- **URL:** `https://cloud.tenable.com/mcp/`
- **Auth:** `X-ApiKeys: accessKey={ACCESS_KEY};secretKey={SECRET_KEY}`
- **License Required:** Tenable One Foundation or Tenable One Advanced
- **Docs:** https://docs.tenable.com/exposure-management/Content/getting-started/hexa-AI-MCP.htm

#### Claude Desktop Config
```json
{
  "mcpServers": {
    "tenable": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://cloud.tenable.com/mcp/",
        "--header",
        "X-ApiKeys: accessKey=<YOUR_ACCESS_KEY>;secretKey=<YOUR_SECRET_KEY>"
      ]
    }
  }
}
```

#### Claude Code
```bash
claude mcp add \
    --transport http tenable https://cloud.tenable.com/mcp \
    --header "X-ApiKeys: accessKey=<YOUR_ACCESS_KEY>;secretKey=<YOUR_SECRET_KEY>"
```

#### Cursor / VS Code
- **Transport:** HTTP
- **URL:** `https://cloud.tenable.com/mcp/`
- **Headers:** `X-ApiKeys: accessKey=<YOUR_ACCESS_KEY>;secretKey=<YOUR_SECRET_KEY>`

#### Example Workflows
- "Tag all Windows devices with `OS:Windows`"
- "Build a risk dashboard showing critical vulns by business unit"
- "Investigate critical vulnerabilities on internet-facing assets"
- "Launch a targeted scan against 10.0.1.0/24"

### navi-mcp (Community)

MCP server for controlling and automating navi. Wraps navi's local SQLite data for AI access.

- **GitHub:** https://github.com/packetchaos/navi-mcp
- **Install:** Clone repo, configure with navi's local database
- **Use case:** AI-driven querying of locally cached Tenable data via navi

### tenable-mcp-mssp (Community)

MCP server for orchestrating Tenable MSSP child container workflows. Enables bulk queries and actions across multiple tenants.

- **GitHub:** https://github.com/andrewspearson/tenable-mcp-mssp
- **Use case:** MSSPs managing multiple Tenable customer environments — bulk tag, bulk export, cross-tenant reporting

---

## Tenable OT Security

OT Security monitors ICS/SCADA/OT environments. It uses a **GraphQL API** (not REST) on an on-prem appliance:

```
POST https://{OT_APPLIANCE_IP}/graphql
```

- Includes a **GraphiQL playground** in the UI at `https://{OT_APPLIANCE_IP}/graphiql`
- API key generated in **OT Security UI → Users → Local Users → Generate API Key**
- pyTenable provides a `TenableOT` class:

```python
from tenable.ot import TenableOT

tot = TenableOT(
    api_key='YOUR_OT_API_KEY',
    url='https://OT_APPLIANCE_IP',
)

# List OT/ICS assets
for asset in tot.assets.list():
    print(vars(asset))

# Query with GraphQL directly
tot.graphql.query('''
  query {
    assets(filter: { type: PLC }) {
      nodes {
        id
        name
        type
        vendor
        firmwareVersion
        risk
      }
    }
  }
''')
```

> **Note:** Tenable Cloud Security (CNAPP) API documentation is only available in-product and requires an active license. It is not covered in this skill.

---

## Tenable Identity Exposure

Identity Exposure (formerly Alsid) uses a dedicated API with its own API key:

```python
from tenable.ie import TenableIE

tie = TenableIE(url='https://your-instance.tenable.com', api_key='YOUR_API_KEY')

# IoE — Indicators of Exposure (misconfigurations)
deviances = tie.deviances.list(...)

# IoA — Indicators of Attack (real-time detection)
attacks = tie.attacks.list(...)
```

Key concepts: **Indicators of Exposure (IoE)**, **Indicators of Attack (IoA)**, **Deviances** (objects deviating from best practices), **MITRE ATT&CK mapping**.

---

## Tenable Security Center

Security Center is an **on-premises** vulnerability management console with its own REST API. It does **not** use `cloud.tenable.com`.

### Authentication
```
x-apikey: accesskey={ACCESS_KEY}; secretkey={SECRET_KEY};
```

Base URL: `https://{SC_HOST}/rest/`

### Key Difference from Tenable Vulnerability Management
Security Center uses a **synchronous, paginated Analysis API** instead of async chunked exports. Be careful with page sizes — SC builds pages in memory before sending.

### Analysis API (Data Exports)
The Analysis API (`POST /rest/analysis`) is the primary way to extract data:

```python
from tenable.sc import TenableSC

tsc = TenableSC(
    url='https://SC_HOST',
    access_key=os.environ['TSC_ACCESS_KEY'],
    secret_key=os.environ['TSC_SECRET_KEY'],
)

# Export asset summary
assets = tsc.analysis.vulns(tool='sumip')

# Export open vuln findings (exclude info severity)
findings = tsc.analysis.vulns(
    ('severity', '=', '1,2,3,4'),   # low, medium, high, critical
)

# Temporal delta — last 1 day of open findings
recent = tsc.analysis.vulns(
    ('severity', '=', '1,2,3,4'),
    ('lastSeen', '=', '0:1'),        # 0:1 = last 1 calendar day
)

# Fixed/patched findings — last 1 day
closed = tsc.analysis.vulns(
    ('severity', '=', '1,2,3,4'),
    ('lastMitigated', '=', '0:1'),
    source='patched',
)

# WAS findings (separate from network vulns)
was_findings = tsc.analysis.vulns(
    ('severity', '=', '1,2,3,4'),
    ('wasVuln', '=', 'onlyWas'),
    tool='wasvulndetail',
)
```

### Analysis API Tools
| Tool | Purpose |
|------|---------|
| `vulndetails` | Raw vulnerability findings (recommended for exports) |
| `sumip` | Asset summary (one row per host) |
| `sumseverity` | Summary by severity |
| `sumfamily` | Summary by plugin family |
| `wasvulndetail` | Web App Scanning findings |

### Source Types
| Source | Description |
|--------|---------|
| `cumulative` | Current active vulns (stateful — default) |
| `patched` | Fixed/remediated vulns (stateful) |
| `individual` | Single scan results (not stateful) |

### Temporal Filter Formats
- **Calendar days:** `0:30` = last 30 days to today
- **Unix timestamps:** `1668961007-1671552959` = specific range (oldest-newest, note the dash)

### SC-Specific API Resources
Analysis, Scans, Scan Policies, Scan Zones, Repositories, Assets, Credentials, Alerts, Reports, Queries, Tickets, Plugins, Users, Groups, Roles, Organizations, Dashboards, Accept/Recast Risk Rules.

> **Tip:** Recommended page size for SC exports: `2000–10000`. Too large = memory issues on the console. Too small = excessive churn.

### References
- **SC API Docs:** https://docs.tenable.com/security-center/api/index.htm
- **pyTenable SC:** https://pytenable.readthedocs.io/en/stable/api/sc/index.html

---

## Common Automation Recipes

### Recipe 1: Export Critical Vulns to CSV
```python
import csv
from tenable.io import TenableIO

tio = TenableIO()
vulns = tio.exports.vulns(severity=['critical'], state=['open', 'reopened'])

with open('critical_vulns.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['Plugin ID', 'Name', 'Severity', 'VPR', 'CVE', 
                     'Asset IP', 'Asset FQDN', 'First Found', 'Last Found'])
    for v in vulns:
        writer.writerow([
            v['plugin']['id'],
            v['plugin']['name'],
            v['severity'],
            v.get('plugin', {}).get('vpr', {}).get('score', ''),
            ', '.join(v.get('plugin', {}).get('cve', [])),
            v.get('asset', {}).get('ipv4', ''),
            v.get('asset', {}).get('fqdn', ''),
            v.get('first_found', ''),
            v.get('last_found', ''),
        ])
```

### Recipe 2: SLA Tracking — Vulns Open Beyond Policy
```python
import time
from tenable.io import TenableIO

SLA_DAYS = {'critical': 7, 'high': 30, 'medium': 90, 'low': 180}
now = int(time.time())

tio = TenableIO()
vulns = tio.exports.vulns(state=['open', 'reopened'])

breached = []
for v in vulns:
    sev = v.get('severity_id', 0)
    sev_name = ['info', 'low', 'medium', 'high', 'critical'][sev]
    if sev_name not in SLA_DAYS:
        continue
    first_found = v.get('first_found', now)
    age_days = (now - first_found) / 86400
    if age_days > SLA_DAYS[sev_name]:
        breached.append({
            'plugin': v['plugin']['name'],
            'severity': sev_name,
            'age_days': round(age_days),
            'sla_days': SLA_DAYS[sev_name],
            'asset': v.get('asset', {}).get('fqdn', v.get('asset', {}).get('ipv4', '')),
        })

print(f"SLA Breached: {len(breached)} vulnerabilities")
```

### Recipe 3: Auto-Tag Assets by Subnet
```python
from tenable.io import TenableIO
import ipaddress

SUBNET_TAGS = {
    '10.1.0.0/16': ('Location', 'HQ'),
    '10.2.0.0/16': ('Location', 'Branch-East'),
    '10.3.0.0/16': ('Location', 'Branch-West'),
    '172.16.0.0/12': ('Environment', 'Lab'),
}

tio = TenableIO()

# Ensure tags exist
tag_uuids = {}
for subnet, (cat, val) in SUBNET_TAGS.items():
    key = f"{cat}:{val}"
    if key not in tag_uuids:
        try:
            tag = tio.tags.create(cat, val)
        except Exception:
            # Tag may already exist; find it
            for t in tio.tags.list():
                if t['category_name'] == cat and t['value'] == val:
                    tag = t
                    break
        tag_uuids[key] = tag['uuid']

# Export assets (v2) and tag by subnet
assets = tio.exports.assets_v2(is_licensed=True)
for asset in assets:
    network = asset.get('network', {})
    ipv4s = network.get('ipv4s', [])
    for ip in ipv4s:
        addr = ipaddress.ip_address(ip)
        for subnet, (cat, val) in SUBNET_TAGS.items():
            if addr in ipaddress.ip_network(subnet):
                tio.tags.assign(
                    assets=[asset['id']],
                    tags=[tag_uuids[f"{cat}:{val}"]],
                )
                break
```

---

## Integration Patterns

### Jira Ticket Creation
```python
# Pattern: Export vulns → create Jira tickets for critical findings
# Use pyTenable for vuln export + jira-python for ticket creation
# Group vulns by asset to avoid ticket flooding
```

### ServiceNow CMDB Sync
```python
# Pattern: Export assets → update/create CIs in ServiceNow CMDB
# Match on servicenow_sysid if available, else FQDN/IP
```

### SIEM Integration
```python
# Pattern: Export vulns → forward to Splunk/Sentinel/QRadar via syslog or REST
# Use the 'since' filter for incremental exports
```

### Slack/Teams Alerting
```python
# Pattern: Scheduled script checks for new critical vulns
# Sends webhook notification with summary
```

---

## Constraints & Rules

1. **NEVER hardcode API keys.** Always use environment variables or a secrets manager.
2. **Always use chunked exports** for bulk data (vulns, assets). Don't use workbench endpoints for large datasets — they are paginated and slower.
3. **Respect rate limits.** Implement exponential backoff with `Retry-After` header.
4. **Export concurrency:** Max 2 concurrent exports per type. Queue additional requests.
5. **Chunk expiration:** Download chunks within 24 hours of export completion.
6. **Prefer pyTenable** over raw HTTP when writing Python. It handles pagination, chunking, and retries automatically.
7. **Use `since` parameter** for incremental/delta exports to avoid re-downloading everything.
8. **VPR > CVSS** for prioritization. VPR incorporates real-world threat intelligence; CVSS is static.
9. **Tags over Target Groups.** Target Groups are deprecated; always use Tags for asset organization.
10. **Don't use `/workbenches/` for large exports.** These are paginated UI-facing endpoints. Use `/vulns/export` and `/assets/export` instead.

---

## Common API Errors

| Code | Meaning | Resolution |
|------|---------|------------|
| `401` | Invalid or missing API keys | Check `X-ApiKeys` header |
| `403` | Insufficient permissions | User needs higher role or specific privilege |
| `404` | Resource not found | Check UUID/ID; resource may be deleted |
| `409` | Conflict (e.g., scan already running) | Wait for current operation to complete |
| `429` | Rate limited | Read `Retry-After` header, backoff |
| `500` | Server error | Retry with backoff; contact Tenable support if persistent |

---

## References

- **API Explorer:** https://developer.tenable.com/reference/navigate
- **Documentation:** https://developer.tenable.com/docs
- **pyTenable Docs:** https://pytenable.readthedocs.io
- **OpenAPI Specs:** https://developer.tenable.com/reference/download-the-specs
- **LLM-Friendly Index:** https://developer.tenable.com/llms.txt
- **Tenable User Guides:** https://docs.tenable.com
