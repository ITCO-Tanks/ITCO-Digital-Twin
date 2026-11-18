# Changelog

## 1.7 (August 2026)

- `frame.ISO6346DimensionCode` renamed to `frame.ISO6346SizeTypeCode` (for example `22K2`).
- `frame.dimensions` (single object with `unit`, `length`, `height`, `width`) removed and replaced by separate `frame.length`, `frame.height` and `frame.width` objects, each with its own `unit` and `value`.
- `tankBasicInformation.lastTest` / `nextTest` objects added (`testType`, `testPerformed`, `testDate`).
- Measurement values are numbers, not strings.

## 1.6 (June–July 2026)

- `designSpecification.maximumCapacity` renamed to `designSpecification.capacity`.
- Field naming review with Bertschi, Stolt and EXSIF: `frame` agreed as the object name, with `frame/length` style sub-fields.

## 1.5 (January 2026)

- Changes from populating the message for a real Stolt tank.
- Added the tank type (`equipmentType`) and renamed the root object to `TankContainerDigitalTwin` so it covers swap-body tanks as well as ISO tanks.
- Typo, consistency and spelling corrections.

## 1.0 / 1.0.1 (June–August 2025)

- First finalised version of the message schema, agreed at the Work Group meeting of 12 June 2025.
- `digitalTwin` object added to hold `lastUpdatedDate` and `version`; `testHistory` object added.
- JSON Schema published with a semantic version (1.0.1).

## 0.1 – 0.2 (2024)

- Initial draft based on the ITCO vocabulary agreed with subject-matter experts.
- Standardised `fitmentLocation`, `liningMaterial`, `coatingMaterial`, and `location` instead of `position`.
