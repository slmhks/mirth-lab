# Mirth Lab Operations Runbook

## 1. Purpose

This runbook describes how to start, validate, troubleshoot,
recover, and safely stop the Mirth Connect laboratory.

## 2. Start the environment

docker compose up -d
docker compose ps

## 3. Health checks

- Confirm Mirth Administrator is available on port 8443.
- Confirm PostgreSQL reports healthy.
- Confirm required destination receivers are running.
- Confirm each required channel is deployed and started.

## 4. Normal message validation

1. Locate the channel message.
2. Verify the source status.
3. Verify each destination status.
4. Inspect the transformed or encoded payload.
5. Inspect the destination response.
6. Correlate HL7 MSH-10 with ACK MSA-2.
7. For FHIR, inspect the HTTP status and transaction-response Bundle.

## 5. Connection failure

Symptoms:

- Destination status QUEUED
- Connection refused or timeout
- Receiver did not record the message

Actions:

1. Test the destination port.
2. Confirm the destination service is running.
3. Verify the configured hostname and port.
4. Restore connectivity.
5. Wait for the queue retry interval.
6. Confirm QUEUED changes to SENT.
7. Confirm the acknowledgment or HTTP response.

## 6. Negative acknowledgment

Symptoms:

- TCP connection succeeds
- Receiver returns AE or AR
- Destination becomes ERROR or QUEUED according to configuration

Actions:

1. Preserve the original failed attempt.
2. Inspect MSA-1, MSA-2, MSA-3 and ERR.
3. Confirm the ACK refers to the correct message-control ID.
4. Correct the message or receiving application.
5. Reprocess the stored message.
6. Confirm the new attempt becomes SENT.
7. Preserve both attempts for audit evidence.

## 7. FHIR failure

1. Inspect the HTTP status.
2. Inspect the OperationOutcome or transaction response.
3. Verify Content-Type and Accept headers.
4. Validate resource references and identifiers.
5. Correct the transformation or endpoint configuration.
6. Reprocess only after evaluating duplicate risk.

## 8. PHI-safe troubleshooting

Do not place patient names, identifiers, dates of birth,
accession numbers, or clinical result values in operational logs,
alert subjects, tickets, filenames, or screenshots.

Use:

- Mirth message ID
- Channel name
- Connector name
- Environment
- Error category
- HTTP status
- ACK code
- Observation count
- Bundle entry count

## 9. Backup

Backup file:

backups/mirthdb-day3.sql

The backup must remain outside the public evidence package.
A successful pg_dump does not prove recoverability.

## 10. Restore validation

Status: NOT TESTED

A future restore test must:

1. Restore into an isolated PostgreSQL instance.
2. Connect an isolated Mirth instance.
3. Confirm users, channels and server configuration.
4. Deploy channels safely.
5. Process synthetic test messages.
6. Confirm responses and message persistence.

## 11. Escalation information

Collect:

- Timestamp and timezone
- Environment
- Channel and connector
- Mirth message ID
- Error category
- Relevant status code or ACK code
- Sanitized exception
- Recent configuration changes
- Recovery actions already attempted
