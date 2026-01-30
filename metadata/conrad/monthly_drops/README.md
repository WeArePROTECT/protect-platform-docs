# Conrad Monthly Metadata Drops

This directory documents the Conrad team's recurring metadata export process.

## Source Location

Monthly drop files are placed on THAR at:
```
/usr2/people/protect/Conrad_Lab/metadata
```

## Update Cadence

Conrad provides recurring CSV "drops" when new samples/data are added. Drops occur **often monthly, but not guaranteed every month**. The cadence is event-driven (when new data are available) rather than strictly calendar-based.

## File Naming

Monthly drop files are named like:
- `protect_metadata_8_27_2025.csv`
- `protect_metadata_9_24_2025.csv`
- `protect_metadata_12_29_2025.csv`

## Export Structure Evolution

- **August 2025 export** had more fields
- **Later exports (September/December 2025)** represent the stable structure currently expected
- The structure evolution reflects refinement of the metadata schema over time

## Sensitive Content

**All actual drop CSV files contain sensitive data and must NOT be committed to this repository.**

## Schema Documentation

Non-sensitive column snapshots (headers only) are maintained in `../schema_snapshots/` for documentation purposes. These files document the structure of exports without exposing sensitive data.

---

**End of README**
