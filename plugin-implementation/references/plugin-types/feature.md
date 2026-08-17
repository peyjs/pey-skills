# Plugin Type: Feature

## Definition
Provides an optional capability the application can work without. The application's primary purpose does not depend on this plugin being present.

## Examples
Search, sharing, advanced charts, CSV import.

## Type-Specific Rules

- **No `adapterType`.** Exclusive to infrastructure plugins.
- **Optionality is the defining trait.** Design this plugin so its absence degrades a specific capability, not the application as a whole. If the requirements describe this plugin as something another part of the system cannot function at all without, that's a signal it may not actually belong to this type — report it rather than silently proceeding (see "Requirement vs. Architecture Conflict" in `plugin-implementation/SKILL.md`).
- **Publishing events is fully allowed**, exactly like any other type (see `../plugin-fundamentals.md`).

## Pre-Delivery Checklist
- [ ] Manifest has no `adapterType` field.
- [ ] Nothing in this plugin's own design assumes it is indispensable to another specific plugin — if the requirements imply that, it was flagged per the rule above rather than built silently.