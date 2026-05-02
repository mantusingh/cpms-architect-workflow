# Reference Documentation

Place OCPI and OCPP specification documents here. The architect agent reads these
at intake time and cites specific section numbers in all design output documents.

## Recommended Documents

| Document | Suggested Filename | Source |
|----------|-------------------|--------|
| OCPP 1.6J Specification | `OCPP_1.6J_specification.pdf` | openchargealliance.org |
| OCPP 1.6J JSON Schemas | `OCPP_1.6J_json_schemas.pdf` | openchargealliance.org |
| OCPP 2.0.1 Part 1 (Architecture) | `OCPP_2.0.1_part1_architecture.pdf` | openchargealliance.org |
| OCPP 2.0.1 Part 2 (Specification) | `OCPP_2.0.1_part2_specification.pdf` | openchargealliance.org |
| OCPP 2.0.1 Part 3 (JSON Schemas) | `OCPP_2.0.1_part3_json_schemas.pdf` | openchargealliance.org |
| OCPI 2.1.1 Specification | `OCPI_2.1.1_specification.pdf` | github.com/ocpi/ocpi |
| OCPI 2.2.1 Specification | `OCPI_2.2.1_specification.pdf` | github.com/ocpi/ocpi |
| ISO 15118-2 (Plug & Charge) | `ISO_15118-2.pdf` | iso.org (if PnC in scope) |

## Where to Get These

- **OCPP specifications**: https://www.openchargealliance.org/protocols/
  (requires free registration)
- **OCPI specifications**: https://github.com/ocpi/ocpi/releases
  (open source, free download)
- **ISO 15118**: Available through ISO or national standards bodies (paid)

## What Happens Without Spec Files

If this folder is empty, the agent will still function and design the architecture,
but it will rely on its training knowledge of the specifications rather than
validating against the actual document text.

**For production-grade architecture work, add the spec PDFs.** The agent will then:
- Quote exact section numbers and paragraph text
- Flag if a design choice conflicts with the spec
- Reference normative requirements ("the CSMS MUST...") directly from the text

## Spec Summaries (Quick Reference)

### OCPP 1.6J
JSON/WebSocket protocol. Charge Points connect to a Central System (CSMS).
Key messages: BootNotification, Authorize, StartTransaction, StopTransaction,
MeterValues, StatusNotification, RemoteStartTransaction, RemoteStopTransaction,
ChangeConfiguration, GetConfiguration, SetChargingProfile.

### OCPP 2.0.1
Major revision. Introduces: EVSE model (replaces connector model), Device Model
profile (component/variable configuration), Security profiles (TLS + client certs),
Smart Charging improvements, ISO 15118 profile (Plug & Charge), RequestStartTransaction
(CSMS-initiated, replaces RemoteStartTransaction).

Key structural change from 1.6J: configuration uses Component+Variable pairs instead
of simple key strings.

### OCPI 2.1.1
REST API for CPO/eMSP roaming. Core modules: Locations, Sessions, CDRs, Tariffs,
Tokens, Commands, Credentials. Token model: push from eMSP to CPO (PATCH /cpo/tokens).

### OCPI 2.2.1
Adds: ChargingProfiles module (§14) for real-time smart charging via roaming network,
enhanced CDR cost elements (parking, reservation), improved Hub topology support,
token pull model alternative to push.
