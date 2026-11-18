---
sidebar_position: 4
sidebar_label: "ITCO Digital Twin"
description: "Upload and retrieve ITCO Tank Container Digital Twins through the BoxTech API."
---

# ITCO Digital Twin

Tank container operators can submit a full **ITCO Tank Container Digital Twin** to BoxTech in a single API call. BoxTech extracts the technical characteristics it holds for every container — size type code, weights, capacity, dimensions and tank attributes — and stores the complete Digital Twin alongside them, so nothing you submit is discarded.

This gives you one upload that both maintains your BoxTech fleet record and preserves the full ITCO specification for the unit.

:::note
Uploading requires **uploader** rights, the same permission needed to manage a container fleet. See [API Documentation](/guides/api/api-documentation) for how to register and obtain a token.
:::

## Uploading a Digital Twin

Send the Digital Twin to the [Upload an ITCO Digital Twin for a tank container](/api/upload-an-itco-digital-twin-for-a-tank-container) endpoint.

You do not supply a container number. BoxTech constructs it from the `BICCode`, `serialNumber` and `checkDigit` in the payload, then validates it against ISO 6346 and confirms the BIC Code is registered to you.

```bash
curl -X POST https://app.bic-boxtech.org/api/v2.0/tank \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d @digital-twin.json
```

A successful upload returns the container number BoxTech recorded:

```json
{
  "status": "success",
  "message": "Tank Digital Twin uploaded successfully",
  "container_number": "STBU2636115"
}
```

## Mandatory fields

BoxTech accepts a Digital Twin only if it carries everything needed to record the container. A payload missing any of the following is rejected with `400` and is **not** partially ingested — you will never end up with a container in your fleet that is missing its size type code or weights.

| ITCO Path | BoxTech Field |
|-----------|---------------|
| `tankBasicInformation.assetIdentification.BICCode` | `container_number` (first 4 characters) |
| `tankBasicInformation.assetIdentification.serialNumber` | `container_number` (characters 5-10) |
| `tankBasicInformation.assetIdentification.checkDigit` | `container_number` (last character) |
| `tankBasicInformation.designSpecification.tareWeight.value` | `tare_kg` |
| `tankBasicInformation.designSpecification.maximumGrossWeight.value` | `max_gross_mass_kg` |
| `tankBasicInformation.designSpecification.maximumPayload.value` | `max_payload_kg` |
| `tankBasicInformation.designSpecification.capacity.value` | `capacity_litres` |
| `tankBasicInformation.designSpecification.RIDADRTankCode` | `adr_rid_code` |
| `tankBasicInformation.designSpecification.Tcode` (`TCode` also accepted) | `imdg_code` |
| `tankShell.baffles.bafflesFitted` | `baffles` |
| `frame.ISO6346SizeTypeCode` | `detail_st` |

BoxTech identifies a container by its 11-character `container_number` alone. The BIC Code, serial number and check digit are not held as separate fields — they are combined into the container number on upload, and split back out again when you read the container through [Get the technical details for a given container](/api/get-the-technical-details-for-a-given-container).

The size type code must be a valid [ISO 6346 size type code](https://www.bic-code.org/size-and-type-code/), and the three weights must correlate — `tare_kg` + `max_payload_kg` must equal `max_gross_mass_kg`.

Every other field in the Digital Twin is optional as far as BoxTech is concerned. All of them are stored against the container and returned unchanged when you read the twin back.

## Required units

Measurements are submitted as a `unit` and `value` pair. BoxTech rejects a payload that declares a unit it does not expect, rather than silently misinterpreting the value.

| Measurement | Required Unit |
|-------------|---------------|
| Weights | `KGM` (kilograms) |
| Capacity | `LTR` (litres) |
| Dimensions | `MMT` (millimetres) |
| Racking force | `NEW` (newtons) |

Conversions to imperial units are performed by BoxTech — submit metric only.

## What BoxTech extracts

These are the fields lifted out of the Digital Twin and held as the container's technical record, alongside the ISO paths they come from:

| BoxTech Field | ITCO Source Path |
|---------------|------------------|
| `container_number` | `assetIdentification.BICCode` + `serialNumber` + `checkDigit` |
| `manufacturer` | `tankBasicInformation.assetIdentification.manufacturerName` |
| `manufacture_date` | `tankBasicInformation.assetIdentification.manufactureDate` |
| `manufacture_number` | `tankBasicInformation.assetIdentification.manufacturerSerialNumber` |
| `current_operator` (returned as `uploaderaccountname`) | `tankBasicInformation.assetIdentification.operator` |
| `tare_kg` | `tankBasicInformation.designSpecification.tareWeight.value` |
| `max_gross_mass_kg` | `tankBasicInformation.designSpecification.maximumGrossWeight.value` |
| `max_payload_kg` | `tankBasicInformation.designSpecification.maximumPayload.value` |
| `capacity_litres` | `tankBasicInformation.designSpecification.capacity.value` |
| `cubic_capacity_cbm` | Computed from `capacity_litres` |
| `adr_rid_code` | `tankBasicInformation.designSpecification.RIDADRTankCode` |
| `imdg_code` | `tankBasicInformation.designSpecification.Tcode` (`TCode` also accepted) |
| `baffles` | `tankShell.baffles.bafflesFitted` |
| `detail_st` | `frame.ISO6346SizeTypeCode` |
| `external_length_mm` | `frame.length.value` |
| `external_width_mm` | `frame.width.value` |
| `external_height_mm` | `frame.height.value` |
| `stacking_kg` | `frame.stackingCapacity.value` |
| `racking_n` | `frame.rackingForce.value` |

All source paths are relative to `TankContainerDigitalTwin`.

BoxTech also derives the imperial equivalents of the weights, volumes and dimensions, the ISO group type code, and the racking force in kilograms where only newtons were supplied. External dimensions are taken from the ISO standard for the size type code you provide; the dimensions in your payload are used only where the code is a range or unassigned code with no standard dimensions.

## Minimal payload

A Digital Twin is normally far larger than this — the example below shows only the sections BoxTech requires, to make the mandatory shape clear.

```json
{
  "TankContainerDigitalTwin": {
    "tankBasicInformation": {
      "assetIdentification": {
        "BICCode": "STBU",
        "serialNumber": "263611",
        "checkDigit": "5",
        "manufacturerName": "Tank Manufacturer Co.",
        "manufactureDate": "2023-08-01",
        "manufacturerSerialNumber": "2308358",
        "operator": "Tank Operator B.V"
      },
      "designSpecification": {
        "RIDADRTankCode": "L4BN",
        "Tcode": "T11",
        "tareWeight": { "unit": "KGM", "value": 4000 },
        "capacity": { "unit": "LTR", "value": 25780 },
        "maximumGrossWeight": { "unit": "KGM", "value": 36000 },
        "maximumPayload": { "unit": "KGM", "value": 32000 }
      }
    },
    "tankShell": {
      "baffles": { "bafflesFitted": "yes" }
    },
    "frame": {
      "ISO6346SizeTypeCode": "22K2",
      "length": { "unit": "MMT", "value": 6058 },
      "height": { "unit": "MMT", "value": 2591 },
      "width": { "unit": "MMT", "value": 2438 },
      "stackingCapacity": { "unit": "KGM", "value": 216000 },
      "rackingForce": { "unit": "NEW", "value": 150000 }
    }
  }
}
```

## Retrieving a Digital Twin

The [Get the ITCO Digital Twin for a tank container](/api/get-the-itco-digital-twin-for-a-tank-container) endpoint returns the complete Digital Twin exactly as it was submitted, wrapped with the date BoxTech received it.

```bash
curl -X GET https://app.bic-boxtech.org/api/v2.0/tank/STBU2636115 \
  -H "Authorization: Bearer <token>"
```

```json
{
  "container_number": "STBU2636115",
  "digital_twin_version": "1.10",
  "uploaded_at": "2025-09-02T10:30:00",
  "itco_digital_twin": {
    "TankContainerDigitalTwin": {
      "digitalTwin": {
        "lastUpdatedDate": "2025-09-02",
        "version": "1.10"
      },
      "tankBasicInformation": { "...": "as submitted" },
      "tankShell": { "...": "as submitted" },
      "frame": {
        "frameType": "beam",
        "materialType": "carbon steel",
        "ISO6346SizeTypeCode": "22K2",
        "length": { "unit": "MMT", "value": 6058 },
        "height": { "unit": "MMT", "value": 2591 },
        "width": { "unit": "MMT", "value": 2438 },
        "stackingCapacity": { "unit": "KGM", "value": 216000 },
        "rackingForce": { "unit": "NEW", "value": 150000 }
      }
    }
  }
}
```

| Field | Meaning |
|-------|---------|
| `container_number` | The container number BoxTech constructed at upload |
| `digital_twin_version` | The ITCO specification version declared in `digitalTwin.version`, or `null` if the twin did not carry a `digitalTwin` block |
| `uploaded_at` | When BoxTech received the twin (UTC) |
| `itco_digital_twin` | The Digital Twin, byte-for-byte as submitted |

`itco_digital_twin` is abbreviated above. It contains every section you uploaded — a complete twin typically runs to several thousand lines covering the shell, manway, valves, walkways, cladding and fittings — returned unchanged.

If no Digital Twin has been uploaded for the container, the endpoint returns `404`. The container's technical details may still be available through [Get the technical details for a given container](/api/get-the-technical-details-for-a-given-container) if it was added to BoxTech by another route.

## Error responses

Every error returns a `message` explaining what was wrong:

```json
{
  "message": "Missing required field 'ISO6346SizeTypeCode' in frame"
}
```

| Code | Reason |
|------|--------|
| `400` | A mandatory field is missing, a unit is not the one BoxTech expects, the container number or size type code is invalid, the weights do not correlate, or the container is flagged as sold or scrapped |
| `401` | Terms and conditions not accepted, or the account lacks uploader rights |
| `404` | No Digital Twin held for this container (`GET` only) |
| `429` | Monthly, daily or per-second quota exceeded |

A rejected upload changes nothing — the container is not created and no partial record is stored, so it is safe to correct the payload and send it again.

## Keeping the twin current

A Digital Twin describes the container as built and as currently configured. Re-upload the twin whenever the specification changes — for example after a major repair, a valve or fitting replacement, or a change of operator. Each upload replaces the stored twin for that container and refreshes the technical record in your fleet.

For general guidance on automating fleet updates, see [Automating the Updates](/guides/integrations/automating-the-updates).

## Attaching the tank certificate

You can also attach the tank's periodic inspection certificate to the container. See [Inspection Certificates](/guides/integrations/inspection-certificates).
