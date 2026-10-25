# Sentinel data-center monitoring foundation

Target: mixed on-premises sites. Sentinel combines operational inventory and
telemetry with its existing security incident workspace. This is an early
foundation; no real data-center hardware or vendor service was contacted during
development. It is not a complete DCIM, NMS, BMS, SIEM or production time-series
platform, and no availability/SLA or large-fleet scale claims are made.

## Architecture

Operators use the web dashboard or Windows console. Site collectors live inside
the management network and push authenticated reports to the control plane.
The API does not fetch device URLs, discover subnets or store BMC credentials.
Deploy the API with PostgreSQL behind HTTPS, and configure existing organization
service keys and dashboard roles. An administrator registers inventory; a service
key can report only to registered assets in its own organization.

The Windows console's owned API still binds to 127.0.0.1 and stores its workspace
locally. It is suitable for a local preview, not a reachable site gateway. Remote
collectors require a properly deployed/reachable control plane and the web UI.
Vercel can host the UI/API; polling physical equipment must run inside the site,
not inside a serverless request. Do not expose a BMC to the public internet.
No new cloud infrastructure, firewall rules, device accounts or credentials are
created automatically. Existing investigation/AI limits remain unchanged.

## Implemented and planned coverage

| Domain | Implemented foundation | Connection limits / next step |
|---|---|---|
| Inventory | Site, room, rack, asset kind, stable ID/key, expected interval | Operator-supplied locations; no discovery or floor-plan import |
| Servers / hypervisors | Host CPU, RAM, system-volume usage and availability | Host collector needs psutil; no VMware/Proxmox API connector yet |
| Hardware | Redfish system/chassis health, power state, targeted sensors | Exact configured HTTPS resource only; no links followed, OEM traversal, paging or discovery |
| Network | Availability, health and utilization telemetry schema | SNMPv3, interface counters/flows and native vendor collectors planned |
| Storage | Capacity percentage and hardware-health schema | OS collector covers the system volume only; array/IOPS/replication connectors planned |
| Power / UPS / PDU | Watts, load %, battery %, availability and health schema | Individual Redfish sensors/legacy PowerControl snapshots where exposed; SNMP/BMS support planned |
| Cooling / environment | Temperature, inlet temperature, humidity and fan RPM schema | Explicit sensor resources; no universal thermal limits, BACnet or Modbus connector |
| Operations | Confirmed threshold transitions, acknowledgement, stale/unknown coverage, charts | In-app alerts only; email, paging, maintenance windows and escalation policies planned |
| AI | Existing separate investigation/support module | No new local runtime, data-center AI diagnosis or autonomous remediation shipped |

Redfish maps Celsius only when ReadingUnits is Cel, power only for W, humidity
for %, and rotation for RPM. Legacy Thermal records expose the maximum reported
component and intake temperatures. A legacy PowerControl reading is accepted
only for one control, avoiding ambiguous aggregation. Register different sensors
as separate assets to preserve measurement boundaries. Unsupported units/status
remain missing; do not infer that all fans, PSUs, disks or ports are monitored.

## API setup

A hosted control plane is not included in this download. Contact the project
operator for API access and organization credentials before connecting a remote
collector. Private source access is required to deploy your own control plane.

For authorized deployments, apply migrations with the existing Alembic configuration (`alembic upgrade head`),
including revision d4e5f6a7b8c9. Configure production secrets and database as in
the existing deployment guide. Local tests never migrate a customer database.

All endpoints below are under `/api/v1/datacenter`:

| Endpoint | Auth | Purpose |
|---|---|---|
| GET /catalog | Dashboard bearer | Metric units, default rules, connector status |
| POST /assets | Admin bearer | Register inventory and optional explicit thresholds/expected metrics |
| GET /overview | Dashboard bearer | Site-filtered, paginated inventory and current-page status counts |
| POST /telemetry | X-Service-Key | Atomic batch of 1–50 samples |
| GET /alerts | Dashboard bearer | Paginated active operational alerts across sites |
| POST /alerts/{id}/acknowledge | Dashboard bearer | Record ownership; does not resolve or fix anything |
| GET /assets/{id}/history | Dashboard bearer | Latest bounded history query within 24 hours |

Example asset registration (send using an admin token, not a service key):

```json
{
  "asset_key": "dc1/hall-a/rack-01/inlet",
  "name": "Rack 01 intake sensor",
  "kind": "environment",
  "site": "DC1",
  "room": "Hall A",
  "rack": "Rack 01",
  "expected_interval_s": 60,
  "confirmations": 2,
  "expected_metrics": ["inlet_temperature_c"],
  "thresholds": {
    "inlet_temperature_c": {"direction": "high", "warning": 28, "critical": 32}
  }
}
```

These temperature numbers are examples, not a thermal recommendation or equipment
specification. Configure limits from the facility/equipment operating requirements.
Providing thresholds replaces the defaults; an empty object disables threshold
rules for that asset. The first preview has registration, not asset editing.
Plan inventory/rules before connecting collectors; updates/import/delete and
maintenance workflows require additional implementation. The UI exposes basic
registration; custom expected metrics and rules currently use the API.

Use the returned UUID to report readings:

```json
{
  "samples": [{
    "asset_id": "REPLACE_WITH_REGISTERED_UUID",
    "observed_at": "2026-10-25T12:00:00Z",
    "collection_status": "ok",
    "metrics": {"inlet_temperature_c": 24.5}
  }]
}
```

Use a current timestamp. Readings must be finite numbers or null, known metric
keys only, percentages 0–100 and explicit 0/1 availability / 0/1/2 device health.
Timezone-aware observations must be within the last 24 hours and no more than
two minutes ahead. Body and authentication rate limits apply; large deployments
need measured sizing and deliberate rate/retention configuration.

## Collector setup

Extract Sentinel-Collector.zip. The included `collector.py` has one host dependency
is the pinned psutil in the adjacent requirements file. No agent was installed
as a service or scheduled. A single host report was verified against an isolated
localhost API on the development workstation; no physical data-center hardware
was connected.

Set SENTINEL_DC_SERVICE_KEY locally using the organization's provisioned service
key. Do not paste production secrets into chat, source, URLs or CLI flags.

```text
python collector.py --api https://sentinel-api.example --asset-id REGISTERED_UUID --mode host --interval 60
python collector.py --api https://sentinel-api.example --asset-id REGISTERED_UUID --mode redfish --endpoint https://bmc.example/redfish/v1/Chassis/1/Sensors/Intake --interval 60 --ca-bundle site-ca.pem
```

Redfish credentials use SENTINEL_REDFISH_USERNAME and SENTINEL_REDFISH_PASSWORD.
Provision a vendor-supported read-only account. The collector performs GETs only
and rejects HTTP BMC URLs, embedded credentials, oversized responses and redirects.
TLS certificate/hostname checks cannot be disabled; import/provide the site's
trusted CA instead. Its backend reports only normalized metrics, never controller
passwords or raw controller responses. Collection failure sends null/unknown,
not a fabricated down or healthy result. Delivery failures print a generic error
without secrets. There is no durable offline spool yet; missing reports become
stale in the dashboard. Use `--once` for a single configured read.

## Status and alert semantics

No readings means unknown. Reports older than max(3 × expected interval, 300 s)
are stale, not proof that equipment is down. Collection errors and missing expected
measurements prevent an overall normal condition. Default thresholds cover CPU,
RAM, storage/link use, UPS battery, availability and reported device health.
Temperature, watts, humidity and fan limits require explicit site configuration.
“No active alerts” is not full-health certification, especially where limits are
unconfigured or devices are unregistered. Capacity/power values are never summed
across nested server/PDU/UPS boundaries; no PUE or facility total is invented.

Two consecutive reports confirm a breach, severity change or recovery by default.
Duplicates do not advance confirmation; same timestamp with conflicting data
is rejected. Older reports can enter history but never replace latest state or
clear alerts. Missing measurements reset confirmation and cannot clear an active
alert. Acknowledgement is audited, not remediation. Audit transitions preserve
the current alert's opening, change, acknowledgement and recovery; this is not yet
a full alert-episode reporting system.

Overview summary counts refer to the CURRENT PAGE and say so in the API and UI.
The active-alert panel is separate, across sites, and reports its total/pagination.
History queries are capped at 500 rows within 24 hours. On ingestion, older rows
for that reporting asset are pruned; inactive assets are not globally vacuumed.
A production retention/roll-up job and time-series integration are still needed.
PostgreSQL row locks and optimistic asset revisions protect concurrent updates;
a conflict returns 409 and the collector/operator must retry the same batch.

## Preview and verification

The dashboard's sample environment is explicit, synthetic and local to the page.
It never creates inventory, acknowledges real alerts or silently replaces an API
failure. Live data starts empty until operators register and connect equipment.
API integration tests cover tenant/auth boundaries, confirmation, deduplication,
out-of-order telemetry, missing/stale data, recovery, acknowledgement and history.
Collector parsing/TLS rules are fixture-tested. These are not vendor certification
or physical data-center end-to-end tests.

Standards references: [DMTF Redfish schema](https://redfish.dmtf.org/schemas/DSP0268_2026.1.html),
[Prometheus exporters](https://prometheus.io/docs/instrumenting/exporters/).
