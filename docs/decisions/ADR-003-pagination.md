# ADR-003: Cursor-Based Pagination

- **Status:** Accepted
- **Date:** 2026-11-01

## Context

The notes listing endpoint must support retrieving notes in pages rather than returning the entire collection in one response.

Notes are ordered newest first. The pagination strategy should provide predictable results as the collection grows and should avoid unnecessary database work.

The API should also avoid exposing implementation-specific database offsets as its primary pagination mechanism.

## Decision

Noter will use **cursor-based pagination** for listing notes.

The listing endpoint will accept:

- `limit`: the maximum number of notes to return.
- `cursor`: the identifier of the last note returned by the previous page.

The default page size is 20, with supported page sizes of 10 and 20.

Notes will be ordered by:

1. `createdAt` descending.
2. `id` descending as a deterministic tie-breaker.

The cursor will reference the last item returned on the current page. Subsequent requests will continue after that item.

The response will include:

- `items`: notes returned in the current page.
- `nextCursor`: cursor for the next page, or `null` when no further page exists.
- `hasMore`: whether additional records are available.

The implementation will fetch one additional record (`limit + 1`) to determine whether another page exists.

## Rationale

- Cursor pagination avoids scanning and skipping increasingly large offsets.
- It provides more predictable performance as the notes collection grows.
- A stable ordering prevents ambiguous pagination when multiple notes share the same creation timestamp.
- The cursor references an actual record rather than an arbitrary numerical position.
- The response contract is explicit about whether more records are available.

## Alternatives Considered

### OFFSET/LIMIT pagination

The database skips a specified number of records and returns the requested page size.

Rejected because large offsets can become increasingly expensive. Inserts and deletions between requests can also shift page boundaries and cause records to be skipped or repeated.

### Timestamp-only cursor

Pagination uses the creation timestamp of the last returned note.

Rejected because multiple notes may share the same timestamp. A timestamp alone does not guarantee a unique position in the ordering.

## Consequences

### Positive

- More predictable pagination as the collection grows.
- Stable ordering through a unique tie-breaker.
- No dependence on large numerical offsets.
- Explicit continuation state in the API response.

### Negative

- Clients must retain and submit the cursor.
- Cursor pagination is less convenient for jumping directly to an arbitrary page.
- Cursor validity depends on the referenced record and the ordering strategy.
- The ordering fields and cursor behavior must remain consistent across requests.

## Scope

This decision applies to the notes listing endpoint.

It does not introduce reverse pagination, arbitrary page navigation, or filtering. Those capabilities may be considered separately if future requirements justify them.
