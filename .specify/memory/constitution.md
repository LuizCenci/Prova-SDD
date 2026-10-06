<!--
Sync Impact Report
- Version change: (template, unversioned) → 1.0.0
- Modified principles: none renamed (all principles are new; template placeholders replaced)
- Added sections: Core Principles I–VI, Stack and Architecture Constraints,
  Development Workflow and Quality Gates, Governance
- Removed sections: none
- Deferred items: none
- Follow-up: specs/001-equipment-loan-control/plan.md describes a single Flask application;
  it conflicts with Principle I (microservices) and must be revised before /speckit-tasks.
-->
# Loanner Constitution

## Core Principles

### I. Microservices Architecture
The Loanner is built as a set of small services, each owning one business capability
(for example: identity, equipment catalog, loans). Each service MUST own its data store and
MUST NOT read another service's database. Services MUST communicate only through documented,
versioned contracts (API or events) stored in the repository. Each service MUST be
deployable and testable on its own.
Rationale: independent services let the team evolve, scale and fix one capability without
risking the others.

### II. Security by Default (NON-NEGOTIABLE)
Every user-facing and service-to-service operation MUST be authenticated and authorized by
role. Passwords MUST be stored only as salted hashes; secrets MUST NOT be committed to the
repository. All traffic MUST use encrypted transport. All input MUST be validated on the
server, and state-changing web forms MUST be protected against CSRF. Every loan and return
MUST record the authenticated user who performed it. Personal data MUST be limited to what
the feature needs. Security-relevant events MUST be logged without logging secrets or
personal data beyond what is required.
Rationale: the Loanner holds personal data and legal responsibility for borrowed items; users
must be able to trust it.

### III. Web-First Stack
The user interface MUST be delivered as a web application accessible by a standard browser,
with a responsive layout. The chosen technology of each service MUST be recorded and
justified in that feature's plan; the web stack MUST be shared across the front end so new
members face a single, consistent set of tools.
Rationale: the product's goal is to digitize loan management for users on any device without
installing software.

### IV. Clean Code
Code MUST follow clean code principles: intention-revealing names, small functions with a
single responsibility, no duplicated logic, no dead code, no magic values (use named
constants), and explicit error handling. Business rules MUST live in a layer separate from
transport and persistence so they can be tested without a web server. Code MUST pass the
project's linter and formatter with no warnings before merge.
Rationale: readable code lowers the cost of change and the onboarding time of new members.

### V. Documented Code and Test-First Quality
Every public module, class and function MUST have a documentation comment describing its
purpose, inputs, outputs and errors. Comments MUST explain why, not restate what the code
does. Each requirement in a spec MUST be covered by at least one automated test, and tests
for new behavior MUST be written before the implementation and be seen failing first.
Business rules from the spec (such as "an equipment already on loan cannot be loaned again")
MUST be enforced and tested at the data level as well as in application code.
Rationale: documentation and tests are the executable record of what the system promises.

### VI. Complete Documentation for Onboarding
The whole application MUST be documented so a new team member can run it and make a first
change without help. The repository MUST maintain: a root README (purpose, architecture
overview, how to run and test), one README per service (responsibility, contracts, data it
owns, configuration), an architecture decision record for each significant decision, and a
glossary of domain terms. A change that alters behavior, configuration or contracts MUST
update the related documentation in the same pull request. Documentation MUST be written in
Portuguese, the language of the product's users and team, except for code identifiers.
Rationale: documentation that is not kept current with the code stops being trusted.

## Stack and Architecture Constraints

- Contracts between services MUST be versioned; a breaking change MUST create a new version
  and keep the previous one until its consumers migrate.
- Each service MUST expose a health check and structured logs.
- Configuration MUST come from the environment, never from values fixed in code.
- Dependencies MUST be pinned to explicit versions and reviewed before being added.
- Dates for loans and due dates are civil dates (no time of day), and lateness is counted in
  whole days.

## Development Workflow and Quality Gates

- Work follows the Spec Kit flow: specification, clarification, plan, tasks, implementation.
- Every pull request MUST be reviewed by at least one other member, who verifies compliance
  with this constitution.
- A pull request MUST NOT be merged while tests, linter or formatter fail.
- Plans MUST include a Constitution Check; any violation MUST be recorded and justified in
  the plan's Complexity Tracking table.
- Security-related changes MUST be reviewed with extra attention to Principle II.

## Governance

This constitution supersedes other practices. An amendment requires a written proposal,
approval by the project team, and a migration note for any work already in progress. Versions
follow semantic versioning: MAJOR for removal or redefinition of a principle, MINOR for a new
principle or materially expanded guidance, PATCH for wording clarifications. Compliance is
checked in every plan (Constitution Check) and in every pull request review. Principle II is
non-negotiable and cannot be waived through Complexity Tracking.

**Version**: 1.0.0 | **Ratified**: 2026-10-05 | **Last Amended**: 2026-10-05
