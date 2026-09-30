# Proportional delivery checks

Choose checks based on the agreed data contract and risk; do not import every item into a tiny local job.

| Area | Checks to consider |
| --- | --- |
| Source and permissions | Correct source/environment, extraction rights, rate limits, unchanged production source, no secrets or sensitive rows in test fixtures/logs |
| Contract and modeling | Grain, keys, nullable fields, units/time zone, source-of-record, expected output schema and agreed metric definitions |
| Loading | New rows, updates, deletes, duplicates, late/out-of-order events, partial batch, cursor boundary, pagination, replay/backfill, idempotent rerun |
| Transformation | Valid/invalid representative cases, business rules, reconciliation, schema drift, joins at intended grain, lineage |
| Quality | Uniqueness, non-null, referential checks, freshness, volume anomalies and business metrics according to impact; thresholds and owner |
| Operations | Schedules/dependencies, retries and alerts, incident contact, retention/deletion, backup/restore, cost envelope, rollback |
| Access | Intended consumers can read only what they are authorized to; contract and self-service definitions are discoverable |

Tests against real systems can modify state even if they are called “smoke tests.” Separate safe local checks from external tests and describe their write set. If a failure interrupts a load, check the persisted state before retrying; avoid duplicating records or replaying unbounded history. Report environment, sample type (synthetic versus real), exact checks, pass/fail and anything unverified.
