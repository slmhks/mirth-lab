# Test Evidence

Raw payloads and screenshots are intentionally excluded from the
public portfolio. All identifiers below belong to synthetic laboratory
messages.

| Test | Recorded result | Public evidence | Result |
|---|---|---|---|
| ADT accepted | `MSA|AA|ADT000001` | CH01 export and sanitized test record | PASS |
| SIU routing | S12 and S15 reached their intended destinations | CH02 export and sanitized test record | PASS |
| CSV to ORM | Destination `SENT`; `MSA|AA|ORM-ORD10001` | CH03 export and sanitized test record | PASS |
| XML to ORU | ORU000001 produced 3 OBX; ORU000002 produced 4 OBX | CH04 export and sanitized test record | PASS |
| Receiver outage | ORU000003 changed from `QUEUED` to `SENT` after recovery | Recovery channel export and test record | PASS |
| Negative ACK | ORU000004 returned `AE` and destination became `ERROR` | Recovery and fake-receiver exports | PASS |
| Reprocessing | Original error preserved; new attempt became `SENT` with `AA` | Sanitized test record | PASS |
| ORU to FHIR | HTTP 200; transaction response contained 7 entries | CH05 export and sanitized test record | PASS |
| Missing PID-3.1 | Source transformer stopped with `Missing mandatory HL7 field: PID-3.1` | CH05 validation code | PASS |
| Alert action | Alert counter increased to 1 and Mailpit received the notification | Sanitized test record | PASS |
| Channel exports | Seven channel XML files available | `checksums/channel-exports.sha256` | PASS |
| Database backup | 1.5 MB PostgreSQL dump created and header validated | Private operational record | PASS |
| Database restore | Functional restoration | Not performed | NOT TESTED |

## Export security review

A case-insensitive credential search found only Mirth-generated
`anonymous` and empty password elements in inactive connector
properties. No active credentials, API keys, bearer tokens or secrets
were found. Server-level `.env`, credentials, database backups and
message files are excluded from version control.
