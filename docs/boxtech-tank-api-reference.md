# Tank Digital Twin API Guide

Base URL: `https://uat.bic-boxtech.org/api/v2.0`

## Authentication

All requests require a valid JWT token in the `Authorization` header. The user must have accepted the BoxTech terms and conditions.

**POST /tank** additionally requires the `UPLOADER_GROUP` group permission (user must have a valid BIC Code i.e. STBU)

---

## POST /tank

Upload an ITCO Digital Twin for a tank container. The container number is constructed automatically from the `BICCode`, `serialNumber`, and `checkDigit` fields in the payload.

```
POST https://uat.bic-boxtech.org/api/v2.0/tank
Content-Type: application/json
Authorization: Bearer <token>
```

### Mandatory Fields

A Digital Twin is only accepted if it carries everything BoxTech needs to record the
container. A payload missing any of the following is rejected with `400` — it is never
partially ingested.

| ITCO Path | BoxTech Field |
|-----------|---------------|
| `tankBasicInformation.assetIdentification.BICCode` | `bic_code` |
| `tankBasicInformation.assetIdentification.serialNumber` | `serial_number` |
| `tankBasicInformation.assetIdentification.checkDigit` | `check_digit` |
| `tankBasicInformation.designSpecification.tareWeight.value` | `tare_kg` |
| `tankBasicInformation.designSpecification.maximumGrossWeight.value` | `max_gross_mass_kg` |
| `tankBasicInformation.designSpecification.maximumPayload.value` | `max_payload_kg` |
| `tankBasicInformation.designSpecification.capacity.value` | `capacity_litres` |
| `tankBasicInformation.designSpecification.RIDADRTankCode` | `adr_rid_code` |
| `tankBasicInformation.designSpecification.Tcode` (`TCode` also accepted) | `imdg_code` |
| `tankShell.baffles.bafflesFitted` | `baffles` |
| `frame.ISO6346SizeTypeCode` | `detail_st` |

The size type code must also be a valid ISO 6346 code, and the three weights must
correlate (`tare_kg` + `max_payload_kg` = `max_gross_mass_kg`).

All other fields in the Digital Twin are optional to BoxTech. They are stored in full
against the container and returned unchanged by `GET /tank/{containerNumber}`.

### Required Units

| Measurement       | Required Unit |
|-------------------|---------------|
| Weights           | `KGM` (kilograms) |
| Capacity          | `LTR` (litres) |
| Dimensions        | `MMT` (millimetres) |
| Racking Force     | `NEW` (newtons) |

### ITCO to BoxTech Field Mapping

The API maps the ITCO Digital Twin structure to flat BoxTech fields as follows:

| BoxTech Field          | ITCO Source Path                                                  |
|------------------------|-------------------------------------------------------------------|
| `container_number`     | Constructed from `assetIdentification.BICCode` + `serialNumber` + `checkDigit` |
| `manufacturer`         | `tankBasicInformation.assetIdentification.manufacturerName`       |
| `manufacture_date`     | `tankBasicInformation.assetIdentification.manufactureDate`        |
| `manufacture_number`   | `tankBasicInformation.assetIdentification.manufacturerSerialNumber` |
| `tare_kg`              | `tankBasicInformation.designSpecification.tareWeight.value`       |
| `max_gross_mass_kg`    | `tankBasicInformation.designSpecification.maximumGrossWeight.value` |
| `max_payload_kg`       | `tankBasicInformation.designSpecification.maximumPayload.value`   |
| `capacity_litres`      | `tankBasicInformation.designSpecification.capacity.value`  |
| `cubic_capacity_cbm`   | Computed: `capacity_litres / 1000`                                |
| `adr_rid_code`         | `tankBasicInformation.designSpecification.RIDADRTankCode`         |
| `imdg_code`            | `tankBasicInformation.designSpecification.Tcode` (`TCode` also accepted)                  |
| `baffles`              | `tankShell.baffles.bafflesFitted` (`"yes"` -> `true`, else `false`) |
| `detail_st`            | `frame.ISO6346SizeTypeCode`                                       |
| `external_length_mm`   | `frame.length.value`                                              |
| `external_width_mm`    | `frame.width.value`                                               |
| `external_height_mm`   | `frame.height.value`                                              |
| `stacking_kg`          | `frame.stackingCapacity.value`                                    |
| `racking_n`            | `frame.rackingForce.value`                                        |

All ITCO source paths are relative to `TankContainerDigitalTwin`.

BoxTech stores the 11-character `container_number` only - there are no separate
`bic_code`, `serial_number` or `check_digit` fields on a container. `GET /container`
derives those components back out of the container number when it builds a response.

### Automatic Conversions

The API also computes the following derived fields after mapping:

- **Weight**: `kg` -> `lbs`
- **Volume**: `cbm` -> `cuft`
- **Length**: `mm` -> `ft`
- **Racking**: `N` -> `kg` (if only newtons provided)
- **External dimensions**: Resolved from ISO standard when a known size type code is provided; input values used only for range/unassigned codes.

### Sample Payload

```json
{
  "TankContainerDigitalTwin": {
    "digitalTwin": {
      "lastUpdatedDate": "2025-09-02",
      "version": "1.10"
    },
    "tankBasicInformation": {
      "equipmentType": "ISOTank",
      "assetIdentification": {
        "manufacturerSerialNumber": "2308358",
        "manufacturerName": "Example Tank Manufacturer A",
        "manufactureDate": "2023-08-01",
        "initialCertificateIssuanceDate": "2023-08-01",
        "initialHydraulicTestDate": "2023-07-23",
        "BICCode": "STBU",
        "serialNumber": "263611",
        "checkDigit": "5",
        "owner": "Example Tank Leasing Ltd.",
        "operator": "Example Tank Operator B.V."
      },
      "testHistory": [],
      "designSpecification": {
        "RIDADRTankCode": "L4BN",
        "Tcode": "T11",
        "classificationSociety": "Example Classification Society",
        "tareWeight": {
          "unit": "KGM",
          "value": 4000
        },
        "capacity": {
          "unit": "LTR",
          "value": 25780
        },
        "maximumGrossWeight": {
          "unit": "KGM",
          "value": 36000
        },
        "maximumPayload": {
          "unit": "KGM",
          "value": 32000
        },
        "maximumExternalPressure": {
          "unit": "BAR",
          "value": 0.41
        },
        "maximumVesselTemperature": {
          "unit": "CEL",
          "value": 150
        },
        "minimumVesselTemperature": {
          "unit": "CEL",
          "value": -40
        },
        "testPressure": {
          "unit": "BAR",
          "value": 99
        },
        "typeApprovals": [
          {
            "category": "UN Portable Tank",
            "approvalBy": [
              {
                "approvalBody": "ADR/RID",
                "version": "6.8",
                "workingPressure": {
                  "unit": "BAR",
                  "value": 4
                }
              },
              {
                "approvalBody": "ADR/RID",
                "version": "6.7",
                "workingPressure": {
                  "unit": "BAR",
                  "value": 4
                }
              },
              {
                "approvalBody": "USDOT",
                "version": "",
                "workingPressure": {
                  "unit": "BAR",
                  "value": 4
                }
              },
              {
                "approvalBody": "ASME",
                "version": "",
                "workingPressure": {}
              }
            ]
          }
        ]
      }
    },
    "exteriorMarkings": {
      "isOperatorLogoDisplayed": true,
      "isAdditionalLabelBoardFitted": false,
      "additionalLabelBoards": []
    },
    "tankShell": {
      "tankVessel": {
        "designCodes": [
          "ASME VIII Div.1(NCS)",
          "EN14025"
        ],
        "numberOfCompartments": 1,
        "capacity": {
          "unit": "LTR",
          "nominalCapacity": 26000,
          "measuredCapacity": 25780
        },
        "newBuildInternalFinish": "On completion of fabrication, the vessel's internal surface is degreased, pickled, passivated and neutrilized. The opening points are sealed so that the tank is supplied clean and ready for use.",
        "newBuildSurfacePolish": "Dish Ends polished to Ra<=1.2 micrometres and Circumferential Welds with 300 mmgroun flush and polished to 180 grit on bottom centre line",
        "vacuumStiffeningRingsQuantity": 0
      },
      "barrel": {
        "materialType": "stainless steel",
        "isCoated": false,
        "coatingMaterial": "",
        "isLined": false,
        "liningMaterial": "",
        "minimumThickness": { "unit": "MMT", "value": 4.18 },
        "corrosionAllowance": { "unit": "MMT", "value": 0.42 },
        "manufacturedThickness": { "unit": "MMT", "value": 4.6 },
        "currentThickness": { "unit": "MMT", "value": 4.5 },
        "steelReferenceThickness": { "value": 6, "unit": "MMT" }
      },
      "dishEnds": {
        "materialType": "stainless steel",
        "minimumThickness": { "unit": "MMT", "value": 4.4 },
        "corrosionAllowance": { "unit": "MMT", "value": 0.4 },
        "manufacturedThickness": { "unit": "MMT", "value": 4.8 },
        "currentThickness": { "unit": "MMT", "value": 4.7 },
        "steelReferenceThickness": { "value": 6, "unit": "MMT" }
      },
      "baffles": {
        "bafflesFitted": "yes",
        "fitmentType": "welded",
        "materialType": "stainless steel",
        "manufacturedThickness": { "value": 4.6, "unit": "MMT" },
        "numberOfSpaceSections": 4,
        "numberOfBaffleSets": 3,
        "baffleDesign": "v-type curved",
        "baffleProfile": "curved",
        "areaCoveragePercent": 70
      }
    },
    "frame": {
      "frameType": "beam",
      "materialType": "carbon steel",
      "ISO668FreightContainerDesignation": "1EEE",
      "ISO6346SizeTypeCode": "22K2",
      "length": { "unit": "MMT", "value": 6058 },
      "height": { "unit": "MMT", "value": 2591 },
      "width": { "unit": "MMT", "value": 2438 },
      "paint": {
        "colour": "black",
        "type": "water based"
      },
      "stackingCapacity": {
        "unit": "KGM",
        "value": 216000
      },
      "rackingForce": {
        "unit": "NEW",
        "value": 150000
      },
      "castings": [
        {
          "location": "top",
          "standardDesignation": "ISO 1611",
          "numberOfPairs": 2,
          "materialType": "cast steel",
          "dimensionCodes": ["ISO 1161"]
        },
        {
          "location": "bottom",
          "standardDesignation": "ISO 1611",
          "numberOfPairs": 2,
          "materialType": "cast steel",
          "dimensionCodes": ["ISO 1161"]
        }
      ],
      "isBottomLongitudinalMemberFitted": true,
      "isTopLongitudinalMemberFitted": true,
      "isMisStackingProtectionFitted": true,
      "earthConnectionLocation": "rear",
      "isBullBarFitted": false,
      "bullBarMaterialType": "",
      "isTopFrameProtectionExtensionFitted": false,
      "isForkLiftCompatible": false,
      "isCollapsibleHandrailFitted": false,
      "collapsibleHandrails": {}
    },
    "walkwayAndLadders": {
      "walkway": {
        "isFitted": true,
        "coverageType": "full deck",
        "materialType": "aluminium",
        "standardDesignation": "ISO 1496-3",
        "dimensions": {
          "unit": "MMT",
          "longitudinalWidth": 475,
          "longitudinalDepth": 40,
          "overallLength": 6000
        }
      },
      "ladders": {
        "isFitted": true,
        "quantityOfLadders": 1,
        "ladderSpecification": [
          {
            "isHandgripFitted": true,
            "frontRearPosition": "rear",
            "sidePosition": "right",
            "constructionType": "fixed",
            "toeClearance": { "unit": "MMT", "value": 20 },
            "handgripPosition": "cut into corner protection plate",
            "materialType": "carbon steel"
          }
        ]
      }
    },
    "insulationAndCladding": {
      "isInsulationFitted": true,
      "insulationMaterial": {
        "mainBarrel": "polyurethane",
        "dishEnds": "rockwool",
        "detailAreas": "combination",
        "nominalThickness": {
          "unit": "MMT",
          "majorityOfBarrel": 45,
          "specificAreas": ""
        },
        "conductivity": { "unit": "D53", "value": 27.5 }
      },
      "isCladdingFitted": true,
      "claddingMaterial": {
        "mainBarrel": "GRP",
        "dishEnds": "GRP",
        "thickness": {
          "unit": "MMT",
          "mainBarrel": 1.6,
          "dishEnds": 2.0
        }
      },
      "isCladdingBandFitted": false,
      "numberOfCladdingBands": 0,
      "claddingSectionWidth": {}
    },
    "spillBoxesAndCompartments": {
      "isSpillBoxFitted": true,
      "numberOfSpillBoxes": 4,
      "spillBoxDetails": [
        {
          "fitmentLocation": "rear",
          "isInsulated": false,
          "isLidFitted": false,
          "lidLockingMechanism": "",
          "lidOpening": {},
          "isCoated": false,
          "coatingMaterial": "",
          "isCustomsSealsCompatible": false,
          "isHeatingSystemFitted": false,
          "drainageTubeMaterial": "plastic",
          "drainageTubeRouting": "outside cladding",
          "drainageTubeDimensions": { "unit": "MMT", "diameter": 40 }
        },
        {
          "fitmentLocation": "centre-rear",
          "isInsulated": false,
          "isLidFitted": false,
          "lidLockingMechanism": "",
          "lidOpening": {},
          "isCoated": false,
          "coatingMaterial": "",
          "isCustomsSealsCompatible": false,
          "isHeatingSystemFitted": false,
          "drainageTubeMaterial": "plastic",
          "drainageTubeRouting": "outside cladding",
          "drainageTubeDimensions": { "unit": "MMT", "diameter": 40 }
        },
        {
          "fitmentLocation": "centre-front",
          "isInsulated": false,
          "isLidFitted": false,
          "lidLockingMechanism": "",
          "lidOpening": {},
          "isCoated": false,
          "coatingMaterial": "",
          "isCustomsSealsCompatible": false,
          "isHeatingSystemFitted": false,
          "drainageTubeMaterial": "plastic",
          "drainageTubeRouting": "outside cladding",
          "drainageTubeDimensions": { "unit": "MMT", "diameter": 40 }
        },
        {
          "fitmentLocation": "front",
          "isInsulated": false,
          "isLidFitted": false,
          "lidLockingMechanism": "",
          "lidOpening": {},
          "isCoated": false,
          "coatingMaterial": "",
          "isCustomsSealsCompatible": false,
          "isHeatingSystemFitted": false,
          "drainageTubeMaterial": "plastic",
          "drainageTubeRouting": "outside cladding",
          "drainageTubeDimensions": { "unit": "MMT", "diameter": 40 }
        }
      ]
    },
    "manway": {
      "isDipstickBracketFitted": false,
      "isDipstickFitted": false,
      "isBaffleHatchFitted": true,
      "quantityOfBaffleHatches": 3,
      "manlid": {
        "closureType": "hinged",
        "handNutType": "wingnuts",
        "handNutMaterial": "brass",
        "numberOfClosures": 1,
        "dimensions": { "unit": "MMT", "diameter": 500 },
        "gasketMaterial": "PTFE",
        "isInsulationFitted": false,
        "insulationMaterial": "",
        "isCoated": false,
        "isLined": false,
        "coatingMaterial": "",
        "liningMaterial": ""
      },
      "baffleHatches": [
        {
          "location": "front",
          "closureType": "hinged",
          "handNutType": "wingnuts",
          "handNutMaterial": "brass",
          "quantityOfClosures": 4,
          "dimensions": { "unit": "MMT", "diameter": 300 },
          "gasketMaterial": "PTFE",
          "isInsulationFitted": false,
          "insulationMaterial": "",
          "isCoated": false,
          "isLined": false,
          "coatingMaterial": "",
          "liningMaterial": ""
        },
        {
          "location": "rear",
          "closureType": "hinged",
          "handNutType": "wingnuts",
          "handNutMaterial": "brass",
          "quantityOfClosures": 4,
          "dimensions": { "unit": "MMT", "diameter": 300 },
          "gasketMaterial": "PTFE",
          "isInsulationFitted": false,
          "insulationMaterial": "",
          "isCoated": false,
          "isLined": false,
          "coatingMaterial": "",
          "liningMaterial": ""
        },
        {
          "location": "centre-rear",
          "closureType": "hinged",
          "handNutType": "wingnuts",
          "handNutMaterial": "brass",
          "quantityOfClosures": 4,
          "dimensions": { "unit": "MMT", "diameter": 300 },
          "gasketMaterial": "PTFE",
          "isInsulationFitted": false,
          "insulationMaterial": "",
          "isCoated": false,
          "isLined": false,
          "coatingMaterial": "",
          "liningMaterial": ""
        }
      ]
    },
    "safetyReliefValves": {
      "reliefValveQuantity": 1,
      "reliefValves": [
        {
          "isRuptureDiscFitted": false,
          "location": "centre-front spillbox",
          "manufacturer": "Example Component Manufacturer A",
          "partCode": "010/144000",
          "serialNumber": "23073751",
          "connectionType": "threaded",
          "isVacuumCombinationType": false,
          "isCoated": false,
          "coatingMaterial": "",
          "isLined": false,
          "liningMaterial": "",
          "isFlameTrapFitted": false,
          "pressureGuageFitted": "provision",
          "boreDiameter": { "unit": "INH", "value": 2.5 },
          "pressureSetting": { "unit": "BAR", "value": 4.4 },
          "vacuumSetting": {},
          "pressureGuageRange": {},
          "ruptureDisc": {}
        }
      ]
    },
    "airlineVent": {
      "isHeated": false,
      "heatingType": "",
      "fitmentLocation": "tangentially off centre line",
      "manufacturer": "Example Component Manufacturer A",
      "partCode": "",
      "serialNumber": "",
      "valveType": "ball",
      "size": { "unit": "INH", "value": 1.5 },
      "terminalConnectionType": "threaded",
      "terminalConnectionLocation": "",
      "terminalOrientation": "vertical",
      "materialType": "stainless steel",
      "cap": {
        "threadType": "",
        "size": { "unit": "INH", "value": 1.5 }
      },
      "isGroundOperatedRemotSystemFitted": false,
      "groundOperatedRemoteSystemType": "",
      "pressureGuageFitted": "",
      "weldInFlange": {
        "orientation": "tangentially off centre line",
        "fitmentLocation": "rear spill box",
        "centeringPosition": "off centre-line",
        "isRecessed": false,
        "materialType": "",
        "standardDesignation": "DN 40",
        "boreDiameter": { "unit": "INH", "value": 1.5 },
        "pitchCircleDiameter": { "unit": "MMT", "value": 103.4 },
        "flangeHoles": { "unit": "M Size", "value": 10, "quantity": 4 }
      },
      "weldInFlangeGasket": {},
      "weldInFlangeFasteners": {},
      "syphonTube": {}
    },
    "topDischargeValve": {
      "isFitted": true,
      "isHeated": false,
      "heatingType": "",
      "fitmentLocation": "centre-rear spill box",
      "manufacturer": "Example Component Manufacturer A",
      "partCode": "",
      "serialNumber": "",
      "type": "butterfly",
      "size": { "unit": "MMT", "value": 80 },
      "terminalConnectionType": "flanged",
      "terminalOrientation": "vertical",
      "materialType": "stainless steel",
      "centeringPosition": "",
      "weldInFlange": {
        "orientation": "tangential",
        "fitmentLocation": "rear spill box",
        "centeringPosition": "centre-line",
        "isRecessed": false,
        "materialType": "1.4404",
        "boreDiameter": { "unit": "MMT", "value": 100 },
        "boltingPattern": {
          "type": "current",
          "standardDesignation": "DN 80",
          "pitchCircleDiameter": { "unit": "MMT", "value": 168 },
          "flangeHoles": { "unit": "M Size", "value": 3, "quantity": 4 }
        }
      },
      "weldInFlangeGasket": {
        "isIntegral": true,
        "materialType": "PTFE envelope",
        "manufacturer": "Example Component Manufacturer B",
        "partCode": "",
        "serialNumber": "",
        "clearanceHeight": { "unit": "MMT", "value": 2.3 }
      },
      "weldInFlangeFasteners": {
        "fastenerType": "bolt",
        "materialType": "stainless steel",
        "size": { "unit": "M Number", "value": 10 }
      },
      "isSyphonTubeRecessProvision": true,
      "syphonTubeRecessProvision": {},
      "syphonTube": {
        "isFitted": true,
        "materialType": "stainless steel",
        "isGuideFittedAtTankFloor": true,
        "isSumpFittedAtTankFloor": false,
        "isCoated": false,
        "coatingMaterial": "",
        "isLined": false,
        "liningMaterial": "",
        "outerDiameter": { "unit": "MMT", "value": 86 },
        "tankFloorTerminationGap": { "unit": "MMT", "value": 3 }
      }
    },
    "topSpareProvision": {
      "isFitted": true,
      "fitmentLocation": "centre-rear spill box",
      "materialType": "stainless steel",
      "weldInFlange": {
        "orientation": "tangential",
        "fitmentLocation": "rear spill box",
        "centeringPosition": "tangential",
        "isRecessed": true,
        "materialType": "1.4404",
        "boreDiameter": { "unit": "MMT", "value": 80 },
        "boltingPattern": {
          "type": "current",
          "standardDesignation": "DN 80",
          "pitchCircleDiameter": { "unit": "MMT", "value": 168 },
          "flangeHoles": { "unit": "M Size", "value": 3, "quantity": 4 }
        }
      },
      "weldInFlangeGasket": {
        "isIntegral": false,
        "materialType": "PTFE envelope",
        "manufacturer": "",
        "partCode": "",
        "serialNumber": "",
        "clearanceHeight": { "unit": "MMT", "value": 3 }
      },
      "weldInFlangeFasteners": {
        "fastenerType": "bolt",
        "materialType": "stainless steel",
        "size": { "unit": "M Size", "value": 10 }
      }
    },
    "bottomDischargeAssembly": {
      "weldInFlange": {},
      "weldInFlangeGasket": {},
      "weldInFlangeFasteners": {},
      "footvalve": {
        "gasketMaterialBetweenValves": "PTFE",
        "isFitted": true,
        "isHeated": false,
        "heatingType": "",
        "fitmentLocation": "bottom valve cabinet",
        "manufacturer": "Example Component Manufacturer A",
        "partCode": "",
        "serialNumber": "",
        "valveType": "poppet",
        "isLined": false,
        "liningMaterial": "",
        "isCoated": false,
        "coatingMaterial": "",
        "size": { "unit": "MMT", "value": 80 },
        "design": "Hi-Lift",
        "angleInDegrees": 45
      },
      "secondaryValve": {
        "isFitted": true,
        "isHeated": false,
        "heatingType": "",
        "fitmentLocation": "",
        "manufacturer": "Example Component Manufacturer A",
        "partCode": "",
        "serialNumber": "",
        "valveType": "butterfly",
        "isLined": false,
        "liningMaterial": "",
        "isCoated": false,
        "coatingMaterial": "",
        "size": { "unit": "MMT", "value": 80 },
        "terminalConnectionType": "flanged",
        "terminalOrientation": "",
        "materialType": "stainless steel"
      },
      "thirdClosure": {
        "terminationType": "Spigot flange & cap",
        "materialType": "stainless steel",
        "manufacturer": "Example Component Manufacturer A",
        "partCode": "",
        "cap": {
          "threadType": "DN 80",
          "size": { "unit": "MMT", "value": 80 }
        },
        "flangeType": "",
        "flangeDesignation": "",
        "otherClosureDetail": "BSP closure for the dust cap"
      },
      "fusibleLink": {
        "isFitted": "provision",
        "lastCheckDate": ""
      },
      "emergencyClosureCable": {
        "isFitted": true,
        "fitmentLocation": "right hand side"
      }
    },
    "valveCabinets": [
      {
        "fitmentLocation": "centre-rear bottom",
        "isFitted": true,
        "isInsulated": false,
        "isDoorLidFitted": false,
        "materialType": "stainless steel",
        "customsSealCompatible": true,
        "locationsOnContainer": []
      }
    ],
    "heatingAndCooling": {
      "isSteamHeatingFitted": true,
      "isElectricBlanketHeatingFitted": false,
      "isElectricGlycolHeatingFitted": false,
      "isElectricGlycolHeatingAndCoolingFitted": false,
      "steamHeatingSystem": {
        "numberOfSteamRuns": 10,
        "steamRunMaterialType": "304 Stainless Steel",
        "isDrainValveFitted": true,
        "effectiveHeatingArea": { "unit": "MTK", "value": 12 },
        "isAdditionalCircuitFitted": false,
        "additionalCircuits": [],
        "maximumWorkingPressure": { "unit": "BAR", "value": 4 },
        "maximumTestPressure": { "unit": "BAR", "value": 6 },
        "isSafetyReliefValveFitted": true,
        "safetyReliefValvePressureSetting": { "unit": "BAR", "value": 4.4 },
        "steamTube": {
          "inletConnection": {
            "type": "threaded",
            "size": { "unit": "INH", "value": 1 },
            "location": "left rear"
          },
          "outletConnection": {
            "type": "threaded",
            "size": { "unit": "INH", "value": 0.75 },
            "location": "right rear"
          }
        },
        "drainValve": {
          "type": "ball",
          "size": { "unit": "MMT", "value": 25 },
          "isSteamTrapFitted": false
        }
      },
      "electricBlanketHeatingSystem": {},
      "electricGlycolHeatingSystem": {},
      "electricGlycolHeatingAndCoolingSystem": {}
    },
    "accessoriesAndNonValveComponents": {
      "specialistTankAccessories": {
        "isBreatherFitted": false,
        "isTrackingDeviceProvision": true,
        "isPoweredTrackingDeviceFitted": false,
        "isUnpoweredTransponderFitted": false
      },
      "temperatureMeasurement": {
        "shellSurfaceTemperatureGuageType": "analogue",
        "liquidCoreTemperatureSensorFitted": false
      },
      "isPressureSensorFitted": true,
      "isLiquidSensorFitted": false,
      "isLeakageDetectionSensorFitted": false
    }
  }
}
```

### curl Example

```bash
curl -X POST https://uat.bic-boxtech.org/api/v2.0/tank \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d @itco_payload.json
```

Or inline (minimal example with required fields only):

```bash
curl -X POST https://uat.bic-boxtech.org/api/v2.0/tank \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>
rAYoSwfQG5A" \
  -d '{
    "TankContainerDigitalTwin": {
      "tankBasicInformation": {
        "assetIdentification": {
          "BICCode": "STBU",
          "serialNumber": "263611",
          "checkDigit": "5",
          "manufacturerName": "Example Tank Manufacturer A",
          "manufactureDate": "2023-08-01",
          "manufacturerSerialNumber": "2308358",
          "operator": "Example Tank Operator B.V."
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
  }'
```

### Success Response

```
200 OK
```

```json
{
  "status": "success",
  "message": "Tank Digital Twin uploaded successfully",
  "container_number": "STBU2636115"
}
```

### Error Responses

Errors return a single `message` field, for example:

```json
{
  "message": "Missing required field 'ISO6346SizeTypeCode' in frame"
}
```

| Code | Reason |
|------|--------|
| `400` | Invalid or missing payload, a missing mandatory field (see [Mandatory Fields](#mandatory-fields)), invalid container number, invalid ISO size type, incorrect units, weight validation failure, container marked as sold/scrapped |
| `401` | Terms not accepted or missing `UPLOADER_GROUP` permission |
| `429` | Quota exceeded |

A rejected upload is not partially ingested - no container is created and no twin is stored.

---

## GET /tank/{containerNumber}

Retrieve the full ITCO Digital Twin for a previously uploaded tank container.

### curl Example

```bash
curl -X GET https://uat.bic-boxtech.org/api/v2.0/tank/STBU2636115 \
  -H "Authorization: Bearer <token>"
```

### Success Response

```
200 OK
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
| `container_number` | Container number constructed at upload |
| `digital_twin_version` | ITCO version declared in `digitalTwin.version`, or `null` if not supplied |
| `uploaded_at` | When BoxTech received the twin (UTC) |
| `itco_digital_twin` | The full Digital Twin, unchanged from submission |

`itco_digital_twin` is abbreviated above - every section submitted is returned.

### Error Responses

| Code | Reason |
|------|--------|
| `400` | Invalid container number format |
| `401` | Terms not accepted |
| `404` | No Digital Twin found for this container |
