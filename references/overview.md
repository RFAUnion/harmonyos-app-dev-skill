# Overview

## Development Model

Prefer the current native HarmonyOS app model:

- Stage model for application structure.
- ArkTS as the default app language.
- ArkUI declarative UI for screens and components.
- Ability Kit for app model, lifecycle, inter-component interaction, and package/runtime behavior.
- Kit-oriented SDK capabilities for system, media, graphics, application services, AI, and other domains.

The older FA model exists in documentation but is not the default choice for new app work. Use it only when an existing project is already FA-based or the user explicitly asks for it.

## Common App Concepts

- `UIAbility`: An application component with UI. It is a system-scheduled unit and usually owns a window.
- Page: An ArkUI screen, usually an `.ets` file under `src/main/ets/pages`.
- `WindowStage`: The UI window stage created for a `UIAbility`.
- `HAP`: Harmony Ability Package. The basic install/run unit for application functionality.
- `HAR`: Static shared package for reusable code/resources. Not independently installed.
- `HSP`: Dynamic shared package for reusable code/resources. Loaded with host app packages.
- Kit: Capability group exposed by HarmonyOS SDK, such as Ability Kit, ArkUI, ArkData, Network Kit, Media Library Kit.

## Practical Defaults

For small and medium apps, use one `entry` HAP with one `UIAbility` and multiple pages. Add more UIAbility components only when the app needs separate recent-task entries, multiple windows, or clearly separate system-scheduled tasks.

Use multiple HAPs only when the app has meaningful dynamic feature modules or extension ability needs. Use HAR/HSP for code/resource sharing, not as a replacement for pages inside a simple app.

## Version Awareness

Always confirm the project's configured SDK/API before using APIs:

- Root `build-profile.json5`: `compileSdkVersion`, `compatibleSdkVersion`, `targetSdkVersion`, `runtimeOS`.
- DevEco SDK Manager: installed OpenHarmony/HarmonyOS SDK components.
- Module `deviceTypes`: must match the SDK/runtime capability set.

Documentation examples may target a specific DevEco Studio or API version. Adapt examples to the installed SDK and project model.
