# Mirth HL7 v2 and FHIR Integration Lab

A production-oriented healthcare interoperability laboratory built with Mirth Connect, Docker, and PostgreSQL.

The project implements inbound and outbound HL7 v2 interfaces, TCP/MLLP communication, acknowledgment validation, file-to-HL7 transformations, failure recovery, controlled message reprocessing, PHI-safe operational alerting, and an HL7 ORU-to-FHIR R4 transformation.

It demonstrates how I design, configure, test, troubleshoot, and document healthcare interfaces using production-oriented practices. It is not a production deployment or a distribution of Mirth Connect, even though it replicates most of the settings of real environment I faced during my experience using Mirth.

## Project objectives

* Build inbound ADT and SIU interfaces over MLLP.
* Generate outbound ORM and ORU messages from file-based inputs.
* Correlate outbound messages and acknowledgments using `MSH-10` and `MSA-2`.
* Distinguish transport failures from application-level rejections.
* Demonstrate destination queuing, retry, recovery, and controlled reprocessing.
* Transform an HL7 ORU result into a FHIR R4 transaction Bundle.
* Apply PHI-safe logging, alerting, evidence handling, and publication practices.
* Produce reproducible configuration exports, checksums, test records, and an operational runbook.

## Skills demonstrated

* Mirth Connect channel development and administration
* HL7 v2.3 ADT, SIU, ORM, and ORU workflows
* TCP/MLLP communication
* HL7 acknowledgment construction, correlation, and validation
* CSV-to-HL7 and XML-to-HL7 transformation
* JavaScript filters, transformers, validation, and logging
* FHIR R4 resources and transaction Bundles
* HTTP/REST destination processing
* Connection-failure analysis and destination queue recovery
* Negative-ACK handling and message reprocessing
* Configuration externalization
* PHI-safe operational logging and alerting
* Docker Compose and PostgreSQL administration
* Channel export integrity verification
* Backup preparation and operational documentation

## Technologies

|Technology|Use in the laboratory|
|-|-|
|Mirth Connect 4.5.2|Integration engine and channel runtime|
|PostgreSQL 17|Mirth configuration and message database|
|Docker Desktop and Docker Compose|Reproducible local infrastructure|
|HL7 v2.3|ADT, SIU, ORM, and ORU messaging|
|FHIR R4|Transaction Bundle and clinical resources|
|TCP/MLLP|HL7 transport and acknowledgment exchange|
|HTTP/REST|FHIR transaction submission|
|JavaScript|Filters, transformations, validation, and logging|
|Mailpit|Local operational-alert testing|
|HAPI FHIR test server|FHIR R4 destination used for laboratory validation|
|SmartHL7 tools|Synthetic HL7 sending and receiving during development|

## Implemented interfaces

|Channel|Source|Transformation|Destination|Protocol|Purpose|
|-|-|-|-|-|-|
|`CH01\\\_IN\\\_ADT\\\_MLLP`|TCP Listener `:6661`|Validate and map ADT|File Writer / source response|MLLP|Process ADT A01/A08 events|
|`CH02\\\_IN\\\_SIU\\\_MLLP`|TCP Listener `:6662`|Filter SIU events|File Writer / source response|MLLP|Route SIU S12/S15 events|
|`CH03\\\_OUT\\\_ORM\\\_MLLP`|File Reader|CSV to ORM^O01|SmartHL7 `:7771`|MLLP|Send radiology orders|
|`CH04\\\_OUT\\\_ORU\\\_MLLP`|File Reader|XML to ORU^R01|SmartHL7 `:7772`|MLLP|Send results with repeating OBX segments|
|`CH04\\\_OUT\\\_ORU\\\_MLLP\\\_RECOVERY`|File Reader|XML to ORU^R01|Test receiver `:7774`|MLLP|Test queuing, rejection, and recovery|
|`CH\\\_TEST\\\_FAKE\\\_RIS\\\_AE`|TCP Listener `:7774`|Build controlled AE ACK|Source response / File Writer|MLLP|Simulate application rejection|
|`CH05\\\_ORU\\\_FILE\\\_TO\\\_FHIR\\\_R4`|File Reader|ORU to FHIR transaction Bundle|HAPI FHIR R4|HTTPS|Submit FHIR resources|

See [the complete interface inventory](docs/interface-inventory.md).

## Repository structure

```text
.
├── README.md
├── compose.yaml
├── .env.example
├── .gitignore
├── checksums/
│   └── channel-exports.sha256
├── docs/
│   ├── interface-inventory.md
│   ├── operations-runbook.md
│   └── test-evidence.md
├── evidence/
│   ├── command-output/
│   └── screenshots/
│       └── README.md
└── exports/
    └── channels/
        ├── CH01\\\_IN\\\_ADT\\\_MLLP.xml
        ├── CH02\\\_IN\\\_SIU\\\_MLLP.xml
        ├── CH03\\\_OUT\\\_ORM\\\_MLLP.xml
        ├── CH04\\\_OUT\\\_ORU\\\_MLLP.xml
        ├── CH04\\\_OUT\\\_ORU\\\_MLLP\\\_RECOVERY.xml
        ├── CH05\\\_ORU\\\_FILE\\\_TO\\\_FHIR\\\_R4.xml
        └── CH\\\_TEST\\\_FAKE\\\_RIS\\\_AE.xml
```

## Quick start

### Prerequisites

* Docker Desktop
* Docker Compose
* Git
* Java support required by the Mirth Administrator client
* A synthetic HL7 sender or receiver for MLLP tests

### 1\. Obtain the project

Clone or download this repository and open a terminal in its root directory.

### 2\. Create the local environment file

From Git Bash, Linux, or macOS:

```bash
cp .env.example .env
```

From PowerShell:

```powershell
Copy-Item .env.example .env
```

Replace every `replace\\\_me` value in `.env` with a local laboratory value. Never commit the resulting `.env` file.

The template contains:

```dotenv
MIRTH\\\_DB\\\_NAME=mirth
MIRTH\\\_DB\\\_USER=replace\\\_me
MIRTH\\\_DB\\\_PASSWORD=replace\\\_me
MIRTH\\\_KEYSTORE\\\_PASSWORD=replace\\\_me
```

### 3\. Validate the Compose configuration

```bash
docker compose config
```

Review the resolved configuration and confirm that no unexpected local value is exposed.

### 4\. Pull and start the services

```bash
docker compose pull
docker compose up -d
docker compose ps
```

Expected core services:

* Mirth Connect
* PostgreSQL
* Mailpit

PostgreSQL and Mailpit should report healthy. Mirth Connect should remain running and expose the configured Administrator port.

### 5\. Open Mirth Connect Administrator

Open:

```text
https://localhost:8443
```

Accept the local laboratory certificate warning if appropriate for your environment, launch the Administrator client, and sign in with your local credentials.

### 6\. Import the channels

In Mirth Connect Administrator:

1. Open **Channels**.
2. Choose **Import Channel**.
3. Select an XML file under `exports/channels/`.
4. Review all connector paths, hostnames, ports, and destination URLs.
5. Repeat for the remaining required channels.
6. Save the imported channels.
7. Deploy only the channels needed for the test being executed.

Do not deploy imported channels without reviewing their environment-specific connector settings.

### 7\. Perform health checks

```bash
docker compose ps
```

Check PostgreSQL:

```bash
docker exec mirth-lab-postgres pg\\\_isready -U mirth -d mirth
```

If you changed the database username or database name, update the command accordingly.

Additional health checks:

* Confirm Mirth Administrator is reachable on port `8443`.
* Confirm each required channel is deployed and started.
* Confirm the required external or laboratory receivers are running.
* Confirm the destination hostname and port match the active test receiver.
* Confirm Mailpit is reachable on port `8025` when testing alerts.

## Normal message validation

For every processed message:

1. Locate the message in the correct Mirth channel.
2. Verify the source connector status.
3. Verify every destination connector status.
4. Inspect the transformed and encoded payloads.
5. Inspect the destination response.
6. Correlate HL7 `MSH-10` with acknowledgment `MSA-2`.
7. Inspect `MSA-1`, `MSA-3`, and `ERR` when an AE or AR response is returned.
8. For FHIR, inspect the HTTP status and transaction-response Bundle.

## Validated test scenarios

|Scenario|Recorded result|Result|
|-|-|-|
|ADT acceptance|`MSA|AA|
|SIU routing|S12 and S15 reached their intended destinations|PASS|
|CSV to ORM|Destination `SENT`; `MSA|AA|
|XML to ORU|ORU000001 produced 3 OBX; ORU000002 produced 4 OBX|PASS|
|Receiver outage|ORU000003 changed from `QUEUED` to `SENT` after recovery|PASS|
|Negative ACK|ORU000004 returned `AE`; destination became `ERROR`|PASS|
|Reprocessing|Original error preserved; new attempt became `SENT` with `AA`|PASS|
|ORU to FHIR|HTTP 200; transaction response contained 7 entries|PASS|
|Missing PID-3.1|Source transformer rejected the message with a controlled validation error|PASS|
|Alert action|Alert counter increased and Mailpit received the notification|PASS|
|Channel exports|Seven channel XML files exported and checksum-verified|PASS|
|Database backup|PostgreSQL dump created and its header validated|PASS|
|Database restore|Functional restoration|NOT TESTED|

The complete sanitized record is available in [the test-evidence document](docs/test-evidence.md).

Raw test payloads and screenshots are intentionally excluded from the public repository.

## Failure and recovery

The laboratory demonstrates two operationally different failure types.

### Connection failure

The destination service or port is unavailable.

Typical evidence:

* Connection refused or timeout
* Destination status `QUEUED`
* No message recorded by the receiver

Recovery procedure:

1. Test network connectivity to the destination.
2. Confirm the destination service is running.
3. Verify the configured hostname and port.
4. Restore connectivity.
5. Wait for the configured queue retry interval.
6. Confirm the message changes from `QUEUED` to `SENT`.
7. Confirm the acknowledgment or application response.

### Application rejection

The TCP connection succeeds, but the receiving application rejects the message with `AE` or `AR`.

Recovery procedure:

1. Preserve the original failed attempt.
2. Inspect `MSA-1`, `MSA-2`, `MSA-3`, and `ERR`.
3. Confirm the acknowledgment refers to the correct message-control ID.
4. Correct the message or receiving application.
5. Evaluate duplicate-processing risk.
6. Reprocess the stored message in a controlled manner.
7. Confirm the new attempt becomes `SENT`.
8. Preserve both attempts for investigation and audit evidence.

Detailed procedures are available in [the operations runbook](docs/operations-runbook.md).

## HL7 ORU to FHIR R4 transformation

`CH05\\\_ORU\\\_FILE\\\_TO\\\_FHIR\\\_R4` reads an HL7 ORU result and builds a FHIR R4 transaction Bundle. The mapping represents the workflow using resources such as:

* `Patient`
* `Encounter`
* `ServiceRequest`
* `Observation`
* `DiagnosticReport`

The test validates:

* Mandatory HL7 fields before submission
* Resource identifiers and references
* Repeating OBX-to-Observation mapping
* FHIR request headers
* HTTP response status
* Transaction-response Bundle entries
* `OperationOutcome` details when the destination rejects a request

This mapping is a laboratory implementation and must be adapted to the profiles, terminology requirements, identifiers, and business rules of a real organization.

## Production-readiness controls

The laboratory includes:

* Configuration externalization
* Environment-specific connector review
* Destination queuing and retry handling
* Controlled validation failures
* MLLP acknowledgment validation
* Operational alerting through Mailpit
* PHI-safe operational logging
* Message-retention and pruning planning
* Preservation of errored and queued messages
* Channel export and SHA-256 checksum generation
* Database-backup preparation
* Restore-test planning
* Operational health checks
* Troubleshooting and escalation guidance
* Sensitive-value review before publication

## PHI-safe operations

Operational logs, alerts, tickets, filenames, and public evidence must not contain patient names, identifiers, dates of birth, accession numbers, or clinical result values.

Prefer operational metadata such as:

* Mirth message ID
* Channel and connector name
* Environment
* Error category
* HTTP status
* HL7 acknowledgment code
* Observation count
* Bundle entry count
* Sanitized exception details

PHI-safe handling is not exclusive to Mirth Connect. It is a general healthcare-technology practice that applies to integration engines, applications, APIs, infrastructure logs, monitoring systems, support tickets, screenshots, email, and incident documentation.

## Privacy and security

Screenshots are intentionally excluded because they may reveal identifiers, clinical values, filesystem paths, network information, private URLs, or sensitive configuration—even when synthetic data was intended.

## Verify channel-export integrity

The SHA-256 checksum file records the reviewed state of every channel export.

Run:

```bash
sha256sum -c checksums/channel-exports.sha256
```

Expected result:

```text
exports/channels/CH01\\\_IN\\\_ADT\\\_MLLP.xml: OK
exports/channels/CH02\\\_IN\\\_SIU\\\_MLLP.xml: OK
exports/channels/CH03\\\_OUT\\\_ORM\\\_MLLP.xml: OK
exports/channels/CH04\\\_OUT\\\_ORU\\\_MLLP.xml: OK
exports/channels/CH04\\\_OUT\\\_ORU\\\_MLLP\\\_RECOVERY.xml: OK
exports/channels/CH05\\\_ORU\\\_FILE\\\_TO\\\_FHIR\\\_R4.xml: OK
exports/channels/CH\\\_TEST\\\_FAKE\\\_RIS\\\_AE.xml: OK
```

If any result is `FAILED`, do not assume the export is the reviewed version. Investigate the difference and regenerate the checksum only after completing a new security and configuration review.

## Container-image policy

The Compose environment uses the official, version-pinned Mirth Connect image:

```yaml
image: nextgenhealthcare/connect:4.5.2
```

## Stop the environment

Stop the running services without removing them:

```bash
docker compose stop
```

Restart them:

```bash
docker compose restart
```

Stop and remove the Compose containers and network:

```bash
docker compose down
```

Do not add `-v` unless you explicitly intend to delete the Compose-managed volumes and understand the recovery impact.

## Operational documentation

* [Interface inventory](docs/interface-inventory.md)
* [Operations runbook](docs/operations-runbook.md)
* [Test evidence](docs/test-evidence.md)
* [Channel-export checksums](checksums/channel-exports.sha256)
* [Screenshot evidence policy](evidence/screenshots/README.md)

## Author

**Roger Panayfo**  
Healthcare Interoperability · Technical Support · Backend Development

## Disclaimer

This repository is provided for educational purposes. It does not provide medical advice, legal advice, compliance certification, or a production-ready healthcare system. Anyone adapting this work is responsible for conducting an independent security, privacy, licensing, interoperability, infrastructure, and regulatory review.

