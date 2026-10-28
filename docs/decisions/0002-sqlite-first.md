# 0002. SQLite for Synapse and the bridge while local

Date: 2026-10-09 - Status: accepted, revisit at deployment

## Context

Both Synapse and the bridge can use SQLite or Postgres. Postgres is the recommended choice for anything long-running: SQLite handles concurrent writes badly and Synapse is write-heavy. Moving an existing Synapse from SQLite to Postgres later is possible but is a manual migration.

The user is new to this stack, and Postgres adds a container, a password and one more thing to back up and update.

## Decision

Start with SQLite for both. It is one file each, with no extra service.

## Consequences

- Fewer moving parts for the first working version.
- If the setup grows (many rooms, long history, several users), migrate Synapse to Postgres, ideally before much data has accumulated. Synapse ships a migration script.
- For a single user on a home PC with media retention enabled, SQLite is expected to be fine. Revisit if "database is locked" errors appear or Synapse feels slow.
- A future bot that only reads events does not need a database.
