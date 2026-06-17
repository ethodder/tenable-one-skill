# Tenable Vulnerability Export Response Schema

## Top-Level Fields

```json
{
  "asset": { ... },
  "output": "string — plugin output text",
  "plugin": { ... },
  "port": {
    "port": 443,
    "protocol": "TCP",
    "service": "www"
  },
  "scan": {
    "completed_at": "2024-06-15T12:00:00Z",
    "schedule_uuid": "abc-123-...",
    "started_at": "2024-06-15T11:30:00Z",
    "uuid": "scan-uuid-..."
  },
  "severity": "critical",
  "severity_id": 4,
  "severity_default_id": 4,
  "severity_modification_type": "NONE",
  "first_found": "2024-01-01T00:00:00Z",
  "last_found": "2024-06-15T12:00:00Z",
  "last_fixed": null,
  "indexed": "2024-06-15T12:05:00Z",
  "state": "OPEN",
  "source": "NESSUS_SCAN"
}
```

## `asset` Object

```json
{
  "uuid": "asset-uuid-...",
  "hostname": "web-server-01",
  "fqdn": "web-server-01.example.com",
  "ipv4": "192.168.1.10",
  "ipv6": "::1",
  "operating_system": ["Ubuntu 22.04 LTS"],
  "network_id": "00000000-0000-0000-0000-000000000000",
  "tracked": true,
  "agent_uuid": "agent-uuid-...",
  "mac_address": "AA:BB:CC:DD:EE:FF",
  "netbios_name": "WEBSERVER01",
  "device_type": "general-purpose"
}
```

## `plugin` Object

```json
{
  "id": 12345,
  "name": "Apache HTTP Server < 2.4.52 Multiple Vulnerabilities",
  "family": "Web Servers",
  "family_id": 36,
  "description": "The remote web server is affected by...",
  "solution": "Upgrade to Apache 2.4.52 or later.",
  "synopsis": "The remote web server is affected by multiple vulnerabilities.",
  "type": "remote",
  "has_patch": true,
  "risk_factor": "Critical",
  "see_also": ["https://httpd.apache.org/security/vulnerabilities_24.html"],
  "cve": ["CVE-2021-44790", "CVE-2021-44224"],
  "bid": [123456],
  "xref": ["IAVA:2021-A-1234"],
  "cpe": ["cpe:/a:apache:http_server"],
  "cvss_base_score": 9.8,
  "cvss_temporal_score": 8.5,
  "cvss_vector": {
    "raw": "AV:N/AC:L/Au:N/C:C/I:C/A:C"
  },
  "cvss3_base_score": 9.8,
  "cvss3_temporal_score": 8.5,
  "cvss3_vector": {
    "raw": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H"
  },
  "vpr": {
    "score": 8.4,
    "drivers": {
      "age_of_vuln": { "lower_bound": 366, "upper_bound": 730 },
      "exploit_code_maturity": "FUNCTIONAL",
      "cvss3_impact_score": 5.9,
      "threat_intensity_last28": "VERY_HIGH",
      "threat_recency": { "lower_bound": 0, "upper_bound": 7 },
      "threat_sources_last28": ["Exploit DB", "Metasploit"]
    },
    "updated": "2024-06-10T00:00:00Z"
  },
  "modification_date": "2024-01-15T00:00:00Z",
  "publication_date": "2021-12-20T00:00:00Z",
  "plugin_modification_date": "2024-01-15T00:00:00Z",
  "plugin_publication_date": "2021-12-21T00:00:00Z"
}
```

## Severity Mapping

| `severity_id` | `severity` | Description |
|---------------|-----------|-------------|
| 0 | `info` | Informational finding |
| 1 | `low` | Low risk |
| 2 | `medium` | Medium risk |
| 3 | `high` | High risk |
| 4 | `critical` | Critical risk |

## VPR Driver Values

### `exploit_code_maturity`
| Value | Description |
|-------|-------------|
| `UNPROVEN` | No known exploit |
| `POC` | Proof of concept exists |
| `FUNCTIONAL` | Working exploit available |
| `HIGH` | Weaponized exploit in widespread use |

### `threat_intensity_last28`
| Value | Description |
|-------|-------------|
| `VERY_LOW` | Minimal threat activity |
| `LOW` | Some threat activity |
| `MEDIUM` | Moderate threat activity |
| `HIGH` | Significant threat activity |
| `VERY_HIGH` | Intense threat activity |

## Asset Export V2 Response Schema (Recommended)

```json
{
  "id": "asset-uuid-...",
  "has_agent": true,
  "has_plugin_results": true,
  "created_at": "2024-01-01T00:00:00Z",
  "terminated_at": null,
  "terminated_by": null,
  "updated_at": "2024-06-15T12:00:00Z",
  "deleted_at": null,
  "deleted_by": null,
  "first_seen": "2024-01-01T00:00:00Z",
  "last_seen": "2024-06-15T12:00:00Z",
  "first_scan_time": "2024-01-01T00:00:00Z",
  "last_scan_time": "2024-06-15T12:00:00Z",
  "last_authenticated_scan_date": "2024-06-15T12:00:00Z",
  "last_licensed_scan_date": "2024-06-15T12:00:00Z",
  "last_schedule_id": "schedule-uuid-...",
  "sources": [
    {
      "name": "NESSUS_SCAN",
      "first_seen": "2024-01-01T00:00:00Z",
      "last_seen": "2024-06-15T12:00:00Z"
    }
  ],
  "tags": [
    {
      "tag_uuid": "tag-uuid-...",
      "tag_key": "Environment",
      "tag_value": "Production",
      "added_by": "user-uuid-...",
      "added_at": "2024-01-15T00:00:00Z"
    }
  ],
  "ratings": {
    "acr": {
      "score": 7,
      "drivers": [
        {
          "driver": "business_criticality",
          "value": "high"
        }
      ]
    },
    "aes": {
      "score": 720,
      "drivers": []
    }
  },
  "network": {
    "fqdns": ["web-server-01.example.com"],
    "ipv4s": ["192.168.1.10"],
    "ipv6s": ["::1"],
    "hostnames": ["web-server-01"],
    "mac_addresses": ["AA:BB:CC:DD:EE:FF"],
    "netbios_names": ["WEBSERVER01"],
    "network_interfaces": [
      {
        "name": "eth0",
        "fqdns": ["web-server-01.example.com"],
        "mac_addresses": ["AA:BB:CC:DD:EE:FF"],
        "ipv4s": ["192.168.1.10"],
        "ipv6s": ["::1"]
      }
    ]
  },
  "operating_systems": ["Ubuntu 22.04 LTS"],
  "system_types": ["general-purpose"],
  "installed_software": ["cpe:/a:apache:http_server:2.4.49"],
  "servicenow_sysid": null,
  "qualys_asset_ids": [],
  "qualys_host_ids": [],
  "manufacturer_tpm_ids": [],
  "symantec_ep_hardware_keys": [],
  "ssh_fingerprints": [],
  "mcafee_epo_guid": null,
  "mcafee_epo_agent_guid": null,
  "bigfix_asset_id": null
}
```

## Asset Export V1 Response Schema (Legacy)

In V1 asset export, fields like `fqdns`, `ipv4s`, `acr_score`, and `exposure_score` are root-level fields rather than nested inside `network` and `ratings` sub-objects.

```json
{
  "id": "asset-uuid-...",
  "has_agent": true,
  "has_plugin_results": true,
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-06-15T12:00:00Z",
  "acr_score": 7,
  "exposure_score": 720,
  "fqdns": ["web-server-01.example.com"],
  "ipv4s": ["192.168.1.10"],
  "operating_systems": ["Ubuntu 22.04 LTS"],
  "sources": [
    {
      "name": "NESSUS_SCAN"
    }
  ]
}
```
