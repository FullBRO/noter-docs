# ADR-001: Selection of TypeScript and NestJS

- **Status:** Accepted
- **Date:** 2026-11-01

## Context

Noter requires a backend framework and programming language suitable for building a reliable, maintainable API.

The project prioritizes reliability, extensibility, and simplicity over raw performance. It should also provide practical experience with technologies relevant to backend development and the current job market.

The application is expected to remain relatively small, but its architecture should support clear separation of responsibilities as functionality grows.

## Decision

Noter will use **TypeScript with NestJS** as its application framework.

NestJS will provide the application structure through modules, controllers, providers, and dependency injection.

The application will use the Fastify HTTP adapter rather than the default Express adapter.

TypeScript will be configured in strict mode to improve compile-time correctness and maintainability.

## Rationale

- TypeScript provides static typing and compile-time checks.
- NestJS encourages modular architecture and separation of concerns.
- Dependency injection makes application components easier to isolate and test.
- NestJS provides established conventions for validation, guards, exception handling, and API documentation.
- Fastify provides an HTTP platform with a relatively small runtime footprint.
- The combination is relevant to modern Node.js backend development.

## Alternatives Considered

### Express

A lightweight and widely adopted Node.js framework.

Rejected as the primary framework because it provides fewer architectural conventions and requires more decisions about application structure and dependency management.

### Fastify without NestJS

Fastify provides routing, plugins, hooks, and request lifecycle management.

Rejected because Noter benefits from NestJS's application architecture and dependency injection.

### Go

A compiled language with strong concurrency support and a relatively small runtime footprint.

Rejected because the project prioritizes gaining experience with the TypeScript/Node.js backend ecosystem.

### Java

A mature backend ecosystem with extensive enterprise tooling.

Rejected because it would introduce a different runtime and ecosystem without providing sufficient benefit for Noter's scope.

## Consequences

### Positive

- Consistent application structure.
- Stronger compile-time guarantees.
- Built-in dependency injection and modularity.
- Access to the Node.js ecosystem.
- Easier isolation of application components during testing.

### Negative

- NestJS introduces framework abstractions and conventions.
- TypeScript compilation and configuration add development complexity.
- Using NestJS with Fastify requires attention to adapter compatibility and framework-specific integrations.

## Scope

This decision establishes the language and application framework for Noter's backend. It does not prescribe every library or implementation detail used within the application.
