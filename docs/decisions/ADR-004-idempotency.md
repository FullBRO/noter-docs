# ADR-004: API-Key-Scoped Content-Based Deduplication

- **Status:** Accepted
- **Date:** 2026-11-11

## Context

The `POST /api/v1/notes` endpoint creates notes in PostgreSQL. A client may retry a request when the server successfully creates a note but the response is lost due to a network failure.

Without duplicate protection, retries can unintentionally create multiple identical notes.

A conventional idempotency-key implementation would require clients to generate and submit an additional `Idempotency-Key` header. The server would persist the key, associate it with the authenticated API key, and store a request fingerprint.

For Noter's current use case, this introduces additional client and server responsibilities without sufficient practical benefit.

Instead, identical note payloads submitted using the same API key will be treated as duplicate creation attempts.

API keys represent independent identities. Consequently, different API keys must be able to create identical notes independently, even when controlled by the same person.

## Decision

Noter will use **API-key-scoped content-based deduplication** instead of client-supplied idempotency keys.

### Identity and ownership

An API key represents an identity.

- Each note belongs to exactly one API-key identity.
- Different API keys are independent identities.
- Deduplication is scoped to the API key that submits the request.
- Identical notes submitted using different API keys are treated as independent records.

### Payload normalization

A deterministic SHA-256 hash will be calculated from the note creation payload.

Normalization rules:

- `content`: preserved exactly as submitted.
- `title`: preserved exactly as submitted. An absent or null title is represented consistently as `null`; an empty string remains distinct.
- `tags`: normalized according to the existing tag normalization rules and sorted alphabetically before hashing.

Tag order will not affect deduplication. Equivalent tag sets will produce the same fingerprint.

The serialization format must be deterministic and unambiguous. Optional values must have consistent representations.

### Database model

A separate `NoteFingerprint` table will track deduplication fingerprints.

It will contain:

- `apiKeyId`: foreign key referencing the API key.
- `contentHash`: SHA-256 hash of the normalized payload.
- `noteId`: unique foreign key referencing the created note.
- `createdAt`: fingerprint creation timestamp.

The composite primary key will be `(apiKeyId, contentHash)`.

The unique constraint on `noteId` ensures that a note has at most one fingerprint.

### Duplicate handling

When a request matches an existing fingerprint for the authenticated API key:

- No new note is created.
- The original note is returned.
- The response indicates deduplication through `duplicate: true`.
- The original note's content, title, tags, and creation timestamp remain unchanged.

HTTP status codes:

- `201 Created` — a new note was created.
- `200 OK` — an existing note was returned due to deduplication.

### Database enforcement and concurrency

The database uniqueness constraint is the source of truth for deduplication.

Application-level existence checks alone are insufficient because concurrent requests may attempt to create the same note simultaneously.

Note creation and fingerprint creation must happen in the same database transaction.

If concurrent requests race to create the same fingerprint, the unique constraint prevents duplicate records. The losing transaction must handle the uniqueness conflict and retrieve the existing note.

### API-key revocation

Revoking an API key must not physically delete its notes or fingerprints.

Revocation and data-access authorization are separate concerns. Whether revoked API keys retain read access to previously created notes is outside the scope of this ADR and will be decided separately.

## Consequences

### Positive

- No additional idempotency header or client-side key management.
- Retries with identical payloads do not create duplicate notes under the same API key.
- Different API keys can independently create identical notes.
- PostgreSQL enforces deduplication, including under concurrent requests.
- The design avoids a separate idempotency-record subsystem.

### Negative

- The same API key cannot intentionally create two notes with identical normalized payloads.
- Different API keys belonging to the same person can create identical notes.
- Payload normalization and deterministic serialization become part of the persistence contract.
- SHA-256 collisions are theoretically possible.
- This provides payload-based deduplication, not general-purpose idempotency for arbitrary API operations.

## Alternatives Considered

### Client-generated idempotency keys

Clients submit a unique key for each logical operation and reuse it for retries. The server stores the key, request fingerprint, and resulting note reference.

Rejected because it introduces additional client and server responsibilities without sufficient practical value for Noter's current use case.

### Global content-based deduplication

A unique content hash prevents identical notes from being created regardless of the API key.

Rejected because separate API keys represent independent identities and must be able to create identical notes independently.

### No duplicate protection

Every valid create request creates a new note.

Rejected because network retries can unintentionally create duplicate records.

## Scope

This decision applies to note creation in the current API version.

It does not establish a general idempotency mechanism for future operations.

If Noter later supports intentional duplicate notes under the same API key or requires more general retry semantics, this decision should be reconsidered.
