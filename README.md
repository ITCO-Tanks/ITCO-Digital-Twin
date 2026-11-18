# ITCO Tank Container Digital Twin

The **ITCO Tank Container Digital Twin** is a standard JSON message describing the technical specification of a tank container (ISO tank or swap-body tank): identification, design specification, shell, frame, valves, fittings, test history and more. It is developed by the ITCO Digital Twin Work Group and lets operators share one consistent record of each tank with carriers, terminals and other partners.

**Current version: 1.7**

This repository is the home for the message specification, JSON schema, examples and supporting documentation. It continues the history of the original working repository at [StoltTankContainers/TankContainer-DigitalTwin](https://github.com/StoltTankContainers/TankContainer-DigitalTwin).

## Getting started

The full Digital Twin has around **530 data attributes**. That is a huge amount to take on in one go, so ITCO is starting with the **14 core data points** and a single use case agreed in the [October 2026 ITCO Antwerp presentation](presentations/2026-10-29_ITCO-Antwerp_Digital-Twin.pdf).

**BIC BoxTech is ITCO's method of sharing the Digital Twin with the industry.** Rather than build a new platform, ITCO makes use of an established one. BoxTech is operated by the Bureau International des Containers (BIC), a non-profit organisation, and already serves the dry container industry with technical information about containers, specifically the data shown on the back of the container door. Tank operators upload their Digital Twins to BoxTech, and carriers, terminals and other parties look a tank up there by its container number.

**Use case 1: carriers onboarding shipper-owned tank containers.** Carriers need reliable data about a shipper-owned tank before they accept a booking. Today that data arrives by email, forms and scanned certificates, and takes around 20 minutes per tank on each side. With the 14 data points uploaded once to BoxTech, a carrier can look the tank up by container number instead.

| # | Data point | Digital Twin field | Mandatory for BoxTech |
|---|---|---|---|
| 1 | Container number | `tankBasicInformation.assetIdentification`: `BICCode`, `serialNumber`, `checkDigit` | Yes |
| 2 | ISO size type | `frame.ISO6346SizeTypeCode` | Yes |
| 3 | Tare weight | `tankBasicInformation.designSpecification.tareWeight` | Yes |
| 4 | Max payload | `tankBasicInformation.designSpecification.maximumPayload` | Yes |
| 5 | Max gross mass | `tankBasicInformation.designSpecification.maximumGrossWeight` | Yes |
| 6 | Inspection date | `tankBasicInformation.lastTest.testDate` | |
| 7 | Next inspection date | `tankBasicInformation.nextTest.testDate` | |
| 8 | Manufacture date | `tankBasicInformation.assetIdentification.manufactureDate` | |
| 9 | Capacity (litres) | `tankBasicInformation.designSpecification.capacity` | Yes |
| 10 | ADR/RID code | `tankBasicInformation.designSpecification.RIDADRTankCode` | Yes |
| 11 | IMDG code | `tankBasicInformation.designSpecification.Tcode` | Yes |
| 12 | Baffles | `tankShell.baffles.bafflesFitted` | Yes |
| 13 | Stacking | `frame.stackingCapacity` | |
| 14 | Racking | `frame.rackingForce` | |

BoxTech rejects an upload that is missing any field marked "Yes". The other five are what carriers ask for when onboarding a tank, so send them too. Start from [`schema/v1.7/itco-tank-digital-twin_boxtech-tanks.schema.json`](schema/v1.7/itco-tank-digital-twin_boxtech-tanks.schema.json), which describes each of these fields, and see `examples/v1.7/SUTU0000059-minimal.json` for a small message. Add the rest of the Digital Twin as further use cases need it.

Measurements are objects with a `unit` (UN/ECE Rec 20 codes: `KGM`, `LTR`, `MMT`, `NEW`) and a numeric `value`. Dates use `YYYY-MM-DD`.

## Repository layout

| Path | Contents |
|---|---|
| `spec/DigitalTwinMessageFormat_v1.7.xlsx` | **Master specification**: every field, its level, data type, allowed values, ITCO mandatory/conditional status and BoxTech mapping. Includes a version history sheet. |
| `spec/ITCO_Digital_Twin_Vocabulary_20241025.xlsx` | Vocabulary agreed with subject-matter experts |
| `spec/archive/` | Earlier spec versions (v0.2, v1.00, v1.6) |
| `schema/v1.7/itco-tank-digital-twin_boxtech-tanks.schema.json` | **Recommended starting point**: JSON Schema (draft-07) for the fields BIC BoxTech reads and holds, each described with its BoxTech mapping |
| `schema/v1.7/itco-tank-digital-twin.schema.json` | JSON Schema (draft-07) for v1.7, as enforced by the BoxTech `POST /tank` endpoint |
| `schema/v1.0/` | Original v1.0 schema and sample from the Stolt repository |
| `schema/archive/` | Interim May 2026 schema covering the BoxTech core fields |
| `examples/v1.7/` | Valid v1.7 messages: two minimal examples and a fully populated example |
| `examples/v1.5/` | Earlier examples, kept for reference (not valid against v1.7) |
| `docs/` | Guides for uploading Digital Twins to BIC BoxTech |
| `presentations/` | ITCO's original vision presentation "Mirroring Reality" (2025) and the Work Group presentation from ITCO Antwerp (Oct 2026) |

## Validating a message

```bash
pip install jsonschema
python -c "import json,jsonschema; jsonschema.validate(json.load(open('my-tank.json')), json.load(open('schema/v1.7/itco-tank-digital-twin_boxtech-tanks.schema.json'))); print('valid')"
```

## Uploading to BIC BoxTech

Operators can upload a Digital Twin to the BIC BoxTech database with a single API call (`POST /api/v2.0/tank`). BoxTech derives its core container data from the message and stores the full Digital Twin alongside it. See [`docs/boxtech-upload-guide.md`](docs/boxtech-upload-guide.md) and the online guide at <https://docs.bic-boxtech.org/guides/itco-digital-twin>.

## Version history

See [CHANGELOG.md](CHANGELOG.md).

## Contributing

Proposed changes to the message format should be raised as an issue for discussion by the Work Group. Changes go into the specification spreadsheet first, then the schema and examples are updated to match.
