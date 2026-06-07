# Shared Schemas

JSON Schema definitions shared across the MSFT360Tools apps. These are the
single source of truth for cross-app data contracts.

## Files

| File | Purpose | Producer | Consumer |
|------|---------|----------|----------|
| `util360-export.v1.schema.json` | Util360 -> Manager360 export envelope | Util360App | Manager360App |

## Versioning

- Schema version is encoded in the filename (`.v1.`) and asserted by the schema's
  `schema_version` `pattern` (`^1\.[0-9]+$`).
- Adding an optional field is a minor bump (still v1).
- Removing/renaming/retyping a field is a major bump -> ship `.v2.schema.json`
  and keep v1 for backward read.

## Validation

Both apps validate against this schema using `ajv` + `ajv-formats`:

- **Util360App** (producer): validate the envelope before writing the export file
  so a corrupt export never leaves the source app.
- **Manager360App** (consumer): validate before any database write so malformed
  or tampered payloads are rejected with a clear error and zero side effects.

## Defensive limits

The schema enforces hard guards against the failure modes that historically
crashed v1:

- Dates pinned to `2000-01-01 .. 2100-12-31` (no year 9999).
- Hour fields bounded by realistic ceilings (daily `<= 24`, weekly `<= 168`,
  monthly `<= 744`, annual `<= 10000`).
- `additionalProperties: false` on every object -> typos and injected fields
  are rejected.
- String `maxLength` on every text field -> bounded memory use.
- `payload_hash` must be a 64-char lowercase hex SHA-256.
