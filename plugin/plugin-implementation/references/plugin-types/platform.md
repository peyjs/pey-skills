# Plugin Type: Platform

## Definition
Exposes native device and runtime capabilities as Bonyan services. Runs inside Bonyan, after infrastructure plugins, with a manifest like any other plugin — this is not the Runtime Host.

## Examples
Camera, microphone, notifications, geolocation, vibration, clipboard, file system access, battery status, network status.

## Type-Specific Rules

- **No `adapterType`.** This is the rule most likely to be gotten wrong for this plugin type: even though platform plugins are built around a device-capability adapter (e.g., a CameraAdapter), they must **never** receive that adapter via the Core's `adapterType` injection mechanism — that path is exclusive to infrastructure plugins. A platform plugin bundles and consumes its adapter **internally**, as part of its own implementation, and exposes a clean Bonyan service on top (e.g., `CameraService`).
- **Do not confuse with the Runtime Host.** The Runtime Host runs outside Bonyan, before the Core, has no manifest, and only supplies ConfigSource/EnvSource/ModuleLoader. A platform plugin is the opposite in every one of these respects: inside Bonyan, after infrastructure plugins, has a manifest, provides device-capability services.
- **Application code never touches the adapter or the raw device API directly** — only the platform plugin's exposed service (e.g., `CameraService.capturePhoto()`), never `navigator.mediaDevices` or similar, directly from a product/feature plugin or the UI.
- **Error conversion:** Raw errors from the underlying device API/adapter must be converted to Bonyan's standard error structure before leaving the plugin's service methods.
- **Publishing events is fully allowed**, exactly like any other type (see `plugin-fundamentals.md`).
- **No business logic (inferred, not a direct rule).** The architecture never states this explicitly for Platform the way it does for adapters, but it follows from two things together: `architecture.md`'s principle that business logic must stay independent of the platform, and the fact that this plugin wraps an adapter that is itself required to carry no business logic. Expose the device/runtime capability as a clean service; keep any business rule about *when or why* to use it in the product/feature plugin that calls this service.

## Pre-Delivery Checklist
- [ ] Manifest has no `adapterType` field.
- [ ] The device/browser API or adapter is instantiated and held internally by this plugin — not received from the Core.
- [ ] The exposed service name follows `vendor.name.service` and its methods are the only way other plugins or the UI reach this capability.
- [ ] All raw errors from the device API are converted to the standard Bonyan error structure.