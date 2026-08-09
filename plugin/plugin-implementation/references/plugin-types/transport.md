# Plugin Type: Transport

## Definition
Manages message transfer between the system and the outside world.

## Examples
HTTP, WebSocket, push notification transport.

## Type-Specific Rules

- **No `adapterType`.** Exclusive to infrastructure plugins. If this plugin needs an adapter-level capability, it must go through an infrastructure plugin's service.
- **Distinguish from Platform notification services.** The low-level device notification capability (showing a native notification to the user) is a Platform Plugin concern (`NotificationService`). A Transport plugin handles the network-level delivery of remote messages (e.g., receiving a push payload over a socket) — the two are complementary, not the same responsibility. Do not merge them into one plugin unless a documented reason exists.
- **No business logic.** This plugin moves messages; it does not interpret or apply business rules to their content. Business logic that reacts to a transported message belongs in a product or feature plugin, which this plugin should notify via a domain event or a service call, not by executing that logic itself.
- **Serialization/mapping is in scope; meaning is not.** Encoding, decoding, framing, and mapping between the external protocol's wire format and a plain internal data shape is exactly what "managing message transfer" requires — this plugin does that. What it must not do is decide what a message *means* for the business domain (that's the business-logic line above). Example: parsing a JSON payload into a plain object is this plugin's job; deciding that payload represents "an order that should now be marked paid" is not.
- **Error conversion:** Raw network/protocol errors must be converted to the standard Bonyan error structure before leaving this plugin's boundary.
- **Publishing events is fully allowed**, exactly like any other type (see `plugin-fundamentals.md`).

## Pre-Delivery Checklist
- [ ] Manifest has no `adapterType` field.
- [ ] No business rule, validation, or data-shaping decision lives inside this plugin beyond what's needed to move the message.
- [ ] Raw connection/protocol errors are converted to the standard Bonyan error structure.
- [ ] If this plugin overlaps with a Platform notification service, the boundary between the two is explicit and documented.