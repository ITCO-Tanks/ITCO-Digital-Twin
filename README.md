# ITCO Tank Container Digital Twin

The **ITCO Tank Container Digital Twin** is a standard JSON message describing the technical specification of a tank container (ISO tank or swap-body tank): identification, design specification, shell, frame, valves, fittings, test history and more. It is developed by the ITCO Digital Twin Work Group and lets operators share one consistent record of each tank with carriers, terminals and other partners.

**Current version: 1.7**

This repository is the home for the message specification, JSON schema, examples and supporting documentation. It continues the history of the original working repository at [StoltTankContainers/TankContainer-DigitalTwin](https://github.com/StoltTankContainers/TankContainer-DigitalTwin).

## Repository layout

| Path | Contents |
|---|---|
| `spec/DigitalTwinMessageFormat_v1.7.xlsx` | **Master specification**: every field, its level, data type, allowed values, ITCO mandatory/conditional status and BoxTech mapping. Includes a version history sheet. |
| `spec/ITCO_Digital_Twin_Vocabulary_20241025.xlsx` | Vocabulary agreed with subject-matter experts |
| `spec/archive/` | Earlier spec versions (v0.2, v1.00, v1.6) |
| `schema/v1.7/itco-tank-digital-twin.schema.json` | JSON Schema (draft-07) for v1.7, as enforced by the BoxTech `POST /tank` endpoint |
| `schema/v1.0/` | Original v1.0 schema and sample from the Stolt repository |
| `schema/archive/` | Interim May 2026 schema covering the BoxTech core fields |
| `examples/v1.7/` | Valid v1.7 messages: a minimum-fields example and a fully populated example |
| `examples/v1.5/` | Earlier examples, kept for reference (not valid against v1.7) |
| `examples/uploads/` | Real payloads sent by members during the first uploads |
| `docs/` | Guides for uploading Digital Twins to BIC BoxTech |
| `presentations/` | Work Group presentations (ITCO Antwerp, Oct 2026; COA Rotterdam, Nov 2026) |

## Minimum (mandatory) fields

A Digital Twin needs at least the following to be accepted by BoxTech. See `examples/v1.7/minimum-fields_SUTU0000059.json`.

- `tankBasicInformation.assetIdentification`: `BICCode`, `serialNumber`, `checkDigit`
- `tankBasicInformation.designSpecification`: `tareWeight`, `maximumGrossWeight`, `maximumPayload`, `capacity`, `RIDADRTankCode` (plus `Tcode`)
- `tankShell.baffles.bafflesFitted`
- `frame`: `ISO6346SizeTypeCode`, `length`, `width`, `height`, `stackingCapacity`, `rackingForce`

Measurements are objects with a `unit` (UN/ECE Rec 20 codes: `KGM`, `LTR`, `MMT`, `NEW`) and a numeric `value`. Dates use `YYYY-MM-DD`.

## Validating a message

```bash
pip install jsonschema
python -c "import json,jsonschema; jsonschema.validate(json.load(open('my-tank.json')), json.load(open('schema/v1.7/itco-tank-digital-twin.schema.json'))); print('valid')"
```

## Uploading to BIC BoxTech

Operators can upload a Digital Twin to the BIC BoxTech database with a single API call (`POST /api/v2.0/tank`). BoxTech derives its core container data from the message and stores the full Digital Twin alongside it. See [`docs/boxtech-upload-guide.md`](docs/boxtech-upload-guide.md) and the online guide at <https://docs.bic-boxtech.org/guides/itco-digital-twin>.

## Version history

See [CHANGELOG.md](CHANGELOG.md).

## Contributing

Proposed changes to the message format should be raised as an issue for discussion by the Work Group. Changes go into the specification spreadsheet first, then the schema and examples are updated to match.
