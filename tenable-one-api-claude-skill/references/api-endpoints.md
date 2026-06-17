# Tenable One API Quick Reference

## Platform & Settings Endpoints

### Access Control (v3)
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v3/access-control/permissions` | Create permission |
| `GET` | `/api/v3/access-control/permissions` | List permissions |
| `GET` | `/api/v3/access-control/permissions/{id}` | Get permission details |
| `PUT` | `/api/v3/access-control/permissions/{id}` | Update permission |
| `DELETE` | `/api/v3/access-control/permissions/{id}` | Delete permission |

### Connectors
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/settings/connectors` | List connectors |
| `POST` | `/settings/connectors` | Create connector |
| `GET` | `/settings/connectors/{id}` | Get connector details |
| `PUT` | `/settings/connectors/{id}` | Update connector |
| `DELETE` | `/settings/connectors/{id}` | Delete connector |

### Exclusions
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/exclusions` | List scan exclusions |
| `POST` | `/exclusions` | Create exclusion |
| `PUT` | `/exclusions/{id}` | Update exclusion |
| `DELETE` | `/exclusions/{id}` | Delete exclusion |

### Networks
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/networks` | List networks |
| `POST` | `/networks` | Create network |
| `GET` | `/networks/{id}` | Get network details |
| `PUT` | `/networks/{id}` | Update network |
| `DELETE` | `/networks/{id}` | Delete network |
| `GET` | `/networks/{id}/scanners` | List scanners in network |
| `POST` | `/networks/{id}/scanners/{scanner_uuid}` | Assign scanner to network |

### Scanners
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/scanners` | List scanners |
| `GET` | `/scanners/{id}` | Get scanner details |
| `PUT` | `/scanners/{id}` | Update scanner |
| `DELETE` | `/scanners/{id}` | Delete scanner |

---

## Vulnerability Management Endpoints

### Vulnerability Exports
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/vulns/export` | Request vuln export (async) |
| `GET` | `/vulns/export/{uuid}/status` | Check export status |
| `GET` | `/vulns/export/{uuid}/chunks/{id}` | Download export chunk |
| `GET` | `/vulns/export/status` | List all export jobs |
| `POST` | `/vulns/export/{uuid}/cancel` | Cancel export job |

### Asset Exports
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v2/assets/export` | Request asset export v2 (async, recommended) |
| `GET` | `/api/v2/assets/export/{uuid}/status` | Check export v2 status |
| `GET` | `/api/v2/assets/export/{uuid}/chunks/{id}` | Download export v2 chunk |
| `POST` | `/assets/export` | Request asset export v1 (async, legacy) |
| `GET` | `/assets/export/{uuid}/status` | Check export v1 status |
| `GET` | `/assets/export/{uuid}/chunks/{id}` | Download export v1 chunk |

### Scans
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/scans` | List all scans |
| `POST` | `/scans` | Create a scan |
| `GET` | `/scans/{id}` | Get scan details |
| `PUT` | `/scans/{id}` | Update a scan |
| `DELETE` | `/scans/{id}` | Delete a scan |
| `POST` | `/scans/{id}/launch` | Launch a scan |
| `POST` | `/scans/{id}/pause` | Pause a running scan |
| `POST` | `/scans/{id}/resume` | Resume a paused scan |
| `POST` | `/scans/{id}/stop` | Stop a running scan |
| `POST` | `/scans/{id}/export` | Export scan results |
| `GET` | `/scans/{id}/export/{file_id}/status` | Check scan export status |
| `GET` | `/scans/{id}/export/{file_id}/download` | Download scan export |

### Assets
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/assets` | List assets |
| `GET` | `/assets/{uuid}` | Get asset details |
| `POST` | `/import/assets` | Import assets |
| `DELETE` | `/assets/{uuid}` | Delete asset |

### Tags
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/tags/values` | Create tag value |
| `GET` | `/tags/values` | List tag values |
| `GET` | `/tags/values/{uuid}` | Get tag value details |
| `PUT` | `/tags/values/{uuid}` | Update tag value |
| `DELETE` | `/tags/values/{uuid}` | Delete tag value |
| `POST` | `/tags/categories` | Create tag category |
| `GET` | `/tags/categories` | List tag categories |
| `POST` | `/tags/assets/assignments` | Assign/unassign tags |

### Agents
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/scanners/{scanner_id}/agents` | List agents |
| `GET` | `/scanners/{scanner_id}/agents/{agent_id}` | Get agent details |
| `DELETE` | `/scanners/{scanner_id}/agents/{agent_id}` | Unlink agent |
| `GET` | `/scanners/{scanner_id}/agent-groups` | List agent groups |
| `POST` | `/scanners/{scanner_id}/agent-groups` | Create agent group |
| `PUT` | `/scanners/{scanner_id}/agent-groups/{id}/agents/{agent_id}` | Add agent to group |

### Users & Access Control
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/users` | List users |
| `POST` | `/users` | Create user |
| `GET` | `/users/{id}` | Get user details |
| `PUT` | `/users/{id}` | Update user |
| `DELETE` | `/users/{id}` | Delete user |
| `PUT` | `/users/{id}/keys` | Generate API keys for user |
| `GET` | `/groups` | List groups |
| `POST` | `/groups` | Create group |
| `POST` | `/v3/access-control/permissions` | Create permission (v3) |

---

## Exposure Management Endpoints

### Exposure View
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v3/exposure-management/exposure-view/cards/search` | Search exposure cards |
| `GET` | `/api/v3/exposure-management/exposure-view/cards/{id}` | Get card details + CES |

### Inventory
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v3/assets/search` | Search assets (inventory) |
| `POST` | `/api/v3/findings/search` | Search findings |
| `POST` | `/api/v3/software/search` | Search installed software |
| `POST` | `/api/v3/tags/search` | Search tags |
| `GET` | `/api/v3/assets/properties` | List filterable asset properties |
| `GET` | `/api/v3/findings/properties` | List filterable finding properties |

### Inventory Exports
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v3/assets/export` | Export assets (async, beta) |
| `POST` | `/api/v3/findings/export` | Export findings (async, beta) |
| `GET` | `/api/v3/exports/{id}/status` | Check export status |
| `GET` | `/api/v3/exports/{id}/chunks/{chunk_id}` | Download chunk |

### Attack Path Analysis
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v3/attack-path/paths/search` | Search attack paths |
| `POST` | `/api/v3/attack-path/techniques/search` | Search attack techniques |
| `POST` | `/api/v3/attack-path/export/attack-paths` | Export attack paths |
| `POST` | `/api/v3/attack-path/export/attack-techniques` | Export techniques |
| `POST` | `/api/v3/attack-path/export/mitre-heatmap` | Export MITRE heatmap |
| `GET` | `/api/v3/attack-path/export/{id}/status` | Export status |
| `GET` | `/api/v3/attack-path/export/{id}/download` | Download export |

---

## Web App Scanning (WAS) Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/was/v2/configs` | List scan configurations |
| `POST` | `/was/v2/configs` | Create scan configuration |
| `GET` | `/was/v2/configs/{id}` | Get config details |
| `PUT` | `/was/v2/configs/{id}` | Update config |
| `DELETE` | `/was/v2/configs/{id}` | Delete config |
| `GET` | `/was/v2/scans` | List WAS scans |
| `POST` | `/was/v2/scans` | Launch WAS scan |
| `GET` | `/was/v2/scans/{id}` | Get scan details |
| `GET` | `/was/v2/scans/{id}/vulnerabilities` | Get scan vulns |
| `GET` | `/was/v2/scans/{id}/report` | Download scan report |
| `GET` | `/was/v2/templates` | List WAS templates |

---

## Identity Exposure Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/infrastructures` | List AD infrastructures |
| `GET` | `/api/directories` | List AD directories |
| `GET` | `/api/profiles` | List IoE/IoA profiles |
| `GET` | `/api/checkers` | List security checkers |
| `GET` | `/api/categories` | List checker categories |
| `POST` | `/api/events/search` | Search AD events |
| `POST` | `/api/profiles/{id}/checkers/{id}/deviances` | Get deviances by checker |
| `GET` | `/api/profiles/{id}/scores` | Get directory scores |
| `GET` | `/api/profiles/{id}/topology` | Get AD topology |

---

## Attack Surface Management (ASM) Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/asm/assets` | List external assets |
| `GET` | `/asm/assets/{id}` | Get asset details |
| `GET` | `/asm/assets/{id}/history` | Get asset history |
| `POST` | `/asm/assets/export/json` | Export assets (JSON) |
| `POST` | `/asm/assets/export/csv` | Export assets (CSV) |
| `GET` | `/asm/sources` | List discovery sources |
| `POST` | `/asm/sources/domain` | Add domain source |
| `POST` | `/asm/sources/ip` | Add IP source |
| `POST` | `/asm/sources/asn` | Add ASN source |

---

## Security Center Endpoints

Security Center is on-premises. Base URL: `https://{SC_HOST}/rest/`

Auth header: `x-apikey: accesskey={ACCESS_KEY}; secretkey={SECRET_KEY};`

### Analysis (Data Export)
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/rest/analysis` | Query vuln/asset data (primary export) |

### Scans & Policies
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/scan` | List scans |
| `POST` | `/rest/scan` | Create scan |
| `GET` | `/rest/scan/{id}` | Get scan details |
| `PATCH` | `/rest/scan/{id}/launch` | Launch scan |
| `GET` | `/rest/scanResult` | List scan results |
| `GET` | `/rest/scanResult/{id}` | Get scan result details |
| `POST` | `/rest/scanResult/{id}/download` | Download scan result |
| `GET` | `/rest/policy` | List scan policies |
| `POST` | `/rest/policy` | Create scan policy |
| `GET` | `/rest/policy/template` | List policy templates |
| `GET` | `/rest/zone` | List scan zones |

### Assets & Repositories
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/asset` | List asset lists |
| `POST` | `/rest/asset` | Create asset list |
| `GET` | `/rest/repository` | List repositories |
| `GET` | `/rest/repository/{id}` | Get repository details |

### Credentials & Config
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/credential` | List credentials |
| `POST` | `/rest/credential` | Create credential |
| `GET` | `/rest/scanner` | List scanners |
| `GET` | `/rest/status` | Get system status |
| `GET` | `/rest/feed` | Get feed/plugin update status |

### Reports & Alerts
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/report` | List reports |
| `POST` | `/rest/report/{id}/launch` | Launch report |
| `GET` | `/rest/reportDefinition` | List report definitions |
| `GET` | `/rest/alert` | List alerts |
| `POST` | `/rest/alert` | Create alert |

### Users & Access
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/user` | List users |
| `GET` | `/rest/currentUser` | Get current user |
| `GET` | `/rest/role` | List roles |
| `GET` | `/rest/group` | List groups |
| `GET` | `/rest/organization` | List organizations |

### Risk Rules & Queries
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/acceptRiskRule` | List accept risk rules |
| `POST` | `/rest/acceptRiskRule` | Create accept risk rule |
| `GET` | `/rest/recastRiskRule` | List recast risk rules |
| `POST` | `/rest/recastRiskRule` | Create recast risk rule |
| `GET` | `/rest/query` | List saved queries |
| `POST` | `/rest/query` | Create query |
| `GET` | `/rest/ticket` | List tickets |

---

## MSSP Portal Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/mssp/accounts` | List child accounts |
| `GET` | `/mssp/accounts/{uuid}` | Get child account details |
| `POST` | `/mssp/accounts/eval/v2` | Create eval account |
| `GET` | `/mssp/accounts/{uuid}/domains` | List domains |
| `POST` | `/mssp/child-containers/{uuid}/keys` | Generate child auth keys |
| `GET` | `/mssp/account-groups` | List account groups |
| `POST` | `/mssp/account-groups` | Create account group |

---

## OT Security Endpoints (GraphQL)

OT Security uses a **GraphQL API** hosted on the on-prem appliance, not REST on `cloud.tenable.com`.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `https://{OT_IP}/graphql` | All queries and mutations |
| `GET` | `https://{OT_IP}/graphiql` | Interactive GraphQL playground (browser) |

### Common GraphQL Queries
```graphql
# List assets
query { assets { nodes { id name type vendor risk ipAddress firmwareVersion } } }

# List vulnerabilities
query { vulnerabilities { nodes { id cveId severity riskScore asset { name ipAddress } } } }

# List network segments
query { networkSegments { nodes { id name type assets { nodes { name } } } } }

# List events/alerts
query { events(filter: { severity: HIGH }) { nodes { id title srcIP dstIP protocol } } }
```

### Authentication
```
Authorization: Key {YOUR_OT_API_KEY}
```

> **Note:** Tenable Cloud Security (CNAPP) API documentation is only available in-product and requires an active license. It is not covered in this reference.

---

## Scan Templates (Common UUIDs)

Use `GET /editor/scan/templates` to list available templates. Common ones:

| Template | Description |
|----------|-------------|
| `basic` | Basic Network Scan |
| `discovery` | Host Discovery |
| `advanced` | Advanced Scan |
| `agent_basic` | Basic Agent Scan |
| `agent_advanced` | Advanced Agent Scan |
| `webapp` | Web Application Tests |
| `compliance` | Policy Compliance Auditing |
| `pci` | PCI Quarterly External Scan |
| `malware` | Malware Scan |
| `scap` | SCAP and OVAL Auditing |

---

## Downloads API Endpoints

Base URL: `https://www.tenable.com/downloads/api/v2`

No authentication required for listing; download links may require a valid Tenable account.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/pages` | List all downloadable products |
| `GET` | `/pages/{page_slug}` | Get product details & file list |
| `GET` | `/pages/{page_slug}/files/{file_name}` | Download a file |

### Common Page Slugs
| Slug | Product |
|------|---------|
| `nessus` | Nessus Scanner |
| `nessus-agents` | Nessus Agent |
| `tenable-sc` | Tenable Security Center |
| `tenable-sc-nessus-plugins` | SC Plugin Feed |
| `nessus-manager` | Nessus Manager |
| `tenable-core-nessus` | Tenable Core + Nessus |
