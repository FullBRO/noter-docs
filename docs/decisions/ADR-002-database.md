# ADR-002: Selection of PostgreSQL

- **Status:** Accepted
- **Date:** 2026-11-01

## Context

Noter requires persistent storage for notes, tags, API keys, and their relationships.

The database must provide reliable transactions, data integrity, relational querying, and a straightforward self-hosting model.

The project prioritizes reliability and simplicity over adopting additional infrastructure or optimizing for an unproven scale requirement.

## Decision

Noter will use **PostgreSQL** as its primary and sole persistent data store.

Prisma ORM will be used for database access and schema management, with Prisma Migrate handling database migrations.

PostgreSQL will remain the authoritative source of truth for application data.

## Rationale

- Relational modeling naturally represents notes, tags, API keys, and their relationships.
- Transactions provide atomicity for operations involving multiple records.
- Foreign keys and unique constraints enforce data integrity at the database level.
- PostgreSQL provides mature indexing and query capabilities.
- It supports the consistency requirements of the application without requiring additional data stores.
- It is widely used in production backend systems.

## Alternatives Considered

### SQLite

A lightweight embedded relational database.

Rejected because PostgreSQL offers a more representative production database environment and a stronger fit for the intended self-hosted backend architecture.

### MongoDB

A document-oriented database.

Rejected because Noter's data has clear relational structure and benefits from foreign keys, constraints, and relational transactions.

## Consequences

### Positive

- Strong relational integrity.
- Transactional operations across related entities.
- Mature indexing and query capabilities.
- One persistent source of truth.
- A widely applicable database skill for backend development.

### Negative

- Requires a separately managed database service.
- Database migrations and schema changes require care.
- PostgreSQL deployment and backups introduce operational responsibilities.

## Scope

PostgreSQL is the only persistent data store for the current architecture.

This decision does not prohibit in-memory application data or temporary caching if justified by future requirements, but no additional persistent infrastructure is introduced by default.
