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
| Valid JWT | HTTP 200; source `TRANSFORMED`; destination `SENT`; `authenticated_and_authorized` | CH06 export and sanitized test record | PASS |
| Malformed JWT | Request rejected with HTTP 401 | CH06 validation code | PASS |
| Invalid JWT signature | Structurally valid modified token rejected with HTTP 401 and `invalid_jwt_signature` | CH06 validation code and sanitized test record | PASS |
| Alert action | Alert counter increased to 1 and Mailpit received the notification | Sanitized test record | PASS |
| Channel exports | Eight channel XML files available | `checksums/channel-exports.sha256` | PASS |
| Database backup | 1.5 MB PostgreSQL dump created and header validated | Private operational record | PASS |
| Database restore | Functional restoration | Not performed | NOT TESTED |
| Expired JWT | Expired genuine token rejected with HTTP 401 and `token_expired` | Sanitized test record | PASS |
| Wrong audience | Genuine token for `billing-api` rejected with HTTP 401 and `invalid_audience` | Sanitized test record | PASS |
| Wrong authorized party | Genuine token from another client rejected with HTTP 401 and `invalid_authorized_party` | Sanitized test record | PASS |
| Missing role | Authenticated token without `radiology-order.submit` rejected with HTTP 403 and `missing_required_role` | Sanitized test record | PASS |
| Restored configuration | Fresh token after Keycloak restoration returned HTTP 200 and `authenticated_and_authorized` | Sanitized test record | PASS |

## Export security review

A case-insensitive credential search found only Mirth-generated
`anonymous` and empty password elements in inactive connector
properties. No active credentials, API keys, bearer tokens or secrets
were found. Server-level `.env`, credentials, database backups and
message files are excluded from version control.

## CH06 security and runtime notes

The CH06 laboratory demonstrated several implementation details that are important for production interoperability work:

- Mirth and Keycloak communicate through the Docker user-defined network.
- Mirth reaches Keycloak by container DNS name (`mirth-lab-keycloak:8080`) rather than the Windows host address.
- Mirth Connect 4.5.2 running Java 17 encountered a Rhino reflective-access limitation when using `java.net.URL` with the JDK internal HTTP implementation.
- The laboratory reused Mirth's existing Apache HttpClient 4.5.13 and HttpCore 4.4.13 libraries through `/opt/connect/custom-lib`.
- `server.includecustomlib` was enabled to expose those libraries to channel JavaScript.
- The JWT signing key is selected by matching the token header `kid` to the Keycloak JWKS.
- Successful RSA signature verification establishes token integrity and cryptographic authenticity but does not replace claim validation.
- `iss`, `exp`, `aud`, `azp`, and the required application role are validated independently.
- Existing JWTs are immutable; Keycloak role or mapper changes require a newly issued token before those changes appear in claims.
- Development message storage may retain the HTTP `Authorization` header in source metadata. Credential persistence must be reviewed before production deployment.
- The laboratory currently emits `WWW-Authenticate: Bearer` statically; production hardening should return it only when appropriate.
- Interactive changes made inside the running container are suitable for laboratory troubleshooting but should be converted into reproducible image or mounted-configuration changes for production.
