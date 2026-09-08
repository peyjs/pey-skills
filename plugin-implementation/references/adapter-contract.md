# Adapter Contract (Bonyan)

Read this whenever the plugin you're building consumes an adapter — an infrastructure plugin receiving one via `adapterType` (see `plugin-types/infrastructure.md`), or a platform plugin bundling one internally (see `plugin-types/platform.md`). This file covers the adapter itself; the Core-injection mechanics live in those two files.

## What Is an Adapter

The layer between the application and anything external that might differ across platforms or change over time. The rest of the application never touches the external technology directly — only through the adapter, via the plugin that consumes it. Swapping the underlying technology (e.g. IndexedDB → PostgreSQL) means changing configuration only; no other code changes.

## Core Principle

**Every adapter of a given kind implements exactly the same set of methods.** A storage adapter for IndexedDB and one for PostgreSQL expose identical `read`/`write`/`delete`/`query` methods. Whatever consumes the adapter never knows or cares which one is active.

This method set is a **formal contract, not a loose example** — the source architecture is explicit that every adapter of a kind implements the set "exactly." A given implementation may expose extra methods beyond the shared set, but only the shared methods are part of the contract the rest of the system may rely on; nothing outside the adapter should call an implementation-specific extra method.

**Delivery mechanism:** the Core never hands an infrastructure plugin an adapter instance. It injects only the configured adapter's **identifier** as a plain string, at `context.config.adapter`. The `bootstrap.json` selection for a type can be a plain string (`"IndexedDBAdapter"`) or an object with an `implementation` field plus extra adapter-specific settings (`{"implementation": "IndexedDBAdapter", "dbName": "my-db"}`) — in the object case, the extra fields land at `context.config.adapterConfig`, separate from `config.adapter` itself. Your plugin must handle both selection forms. See `core/adapters.md` for exact shapes. The plugin is fully responsible for turning the identifier into an actual implementation (e.g. an internal lookup/import keyed by known names).

**Physical location is not prescribed.** An adapter's code may live inside the plugin that consumes it, or in a separate package the plugin imports. Either way, loading and instantiating it is that plugin's responsibility — never the Core's.

## Common Adapter Kinds

| Kind | Base methods | Example implementations |
|---|---|---|
| Storage | `read`, `write`, `delete`, `query` | IndexedDB, SQLite, PostgreSQL, MariaDB |
| Authentication | `login`, `logout`, check-login-status, get-current-user | Email/password, OAuth, biometrics |
| Camera | `requestPermission`, `startPreview`, `capturePhoto`, `stop` | Browser MediaDevices, Capacitor Camera, Tauri camera bridge |
| Microphone | `requestPermission`, `startRecording`, `stopRecording`, `getLevel` | Browser MediaRecorder, Capacitor Voice Recorder, native audio |
| Notifications | display method (platform-specific internally, same external methods) | Browser Notification API, mobile push via Capacitor, desktop via Tauri |
| File system | read/write file methods | File System Access API, Capacitor Filesystem, Tauri fs |
| Geolocation | `getCurrentPosition`, `watchPosition`, `clearWatch` | Browser Geolocation, Capacitor Geolocation, Tauri geolocation |
| Output | generate output from data | PDF, CSV, direct print |

This list is not exhaustive — any dependency on external technology that might change is a candidate for an adapter of a new kind. If you need a kind not listed here, do not invent methods freely: propose the method set explicitly and report it (see Architecture Feedback Clause in `plugin-implementation/SKILL.md`) rather than assuming.

## Adapter Rules

- **No business logic.** An adapter only talks to the outside world — no rules, calculations, or application-level validation. That belongs in the plugin consuming the adapter.
- **Implements the full method set.** If the kind has five defined methods, this adapter has all five, even if a given implementation makes one of them a no-op.
- **Converts its own errors.** Raw errors from the external technology (a database driver, a browser API, a native bridge) must be converted to Bonyan's standard error structure (see `plugin-fundamentals.md`) before they leave the adapter. This is the adapter's primary responsibility, not the consuming plugin's — a note in `plugin-types/infrastructure.md` about the plugin ensuring converted errors is a defensive second check, not a duplicate or competing responsibility. If the adapter did its job, that check should always pass trivially.
- **Replaceable in tests.** Because every adapter of a kind shares the same methods, tests substitute a real adapter with a simple in-memory equivalent implementing that same method set — do not build the adapter in a way that makes this substitution harder (e.g. don't leak implementation-specific state to the consuming plugin).

## Adapter Security

- A **storage adapter** must never persist raw, unprocessed data if the data requires protection — if encryption is needed, that responsibility must be explicit and implemented, not assumed away. **The architecture deliberately does not decide, for any given plugin, whether that plugin's data needs encryption** — this is a per-plugin decision the developer must make and document, not something to default either way. If a plugin handles data that plausibly needs encryption, document that decision explicitly in the plugin's documentation per `plugin-documentation/SKILL.md` (Business Rules or Constraints section) rather than silently encrypting or silently not encrypting.
- An **authentication adapter** holds login credentials securely and never transfers them to any other part of the system — credentials stay inside the adapter's boundary.
- More generally: an adapter is a sensitive point in the system because it's where external data enters. Treat any data it handles as untrusted until validated/sanitized on the way in.