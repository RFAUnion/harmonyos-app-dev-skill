# Project Structure

## Root Files

- `AppScope/app.json5`: Application-level metadata such as `bundleName`, app icon, app label, version code/name, vendor, and app-wide settings.
- `build-profile.json5`: Root build configuration. Defines products, SDK versions, runtime OS, signing configs, build modes, and module list.
- `hvigorfile.ts`: Root hvigor script, usually uses `appTasks`.
- `hvigor/hvigor-config.json5`: hvigor model/execution configuration.
- `oh-package.json5`: Project package metadata and dependencies.
- `oh-package-lock.json5`: Locked dependency state.
- `local.properties`: Local SDK path. Do not commit machine-specific paths unless the project policy says otherwise.

## Module Files

Typical entry module:

```text
entry/
├── build-profile.json5
├── hvigorfile.ts
├── oh-package.json5
└── src/main/
    ├── module.json5
    ├── ets/
    │   ├── entryability/EntryAbility.ets
    │   └── pages/Index.ets
    └── resources/
        ├── base/element/
        ├── base/media/
        └── base/profile/
```

## `module.json5`

Use `module.json5` for module-level configuration:

- `name`: Module name, often `entry`.
- `type`: `entry`, `feature`, or library-related module type.
- `mainElement`: Main ability name.
- `deviceTypes`: Supported devices. Use values supported by the configured SDK.
- `pages`: Usually `$profile:main_pages`.
- `abilities`: UIAbility declarations, source entries, labels, icons, start window, exported status, and skills.
- `extensionAbilities`: Extension ability declarations such as backup, service, form, input method, and other extension types.
- `requestPermissions`: Permissions needed at runtime or install time.

Launcher `skills` commonly contain an entity/action pair. Verify exact action names against the installed SDK and current docs before changing launch behavior.

## Resources

Use resource references instead of hardcoded display strings and media paths:

- Strings: `$string:app_name`
- Colors: `$color:start_window_background`
- Media: `$media:layered_image`
- Profiles: `$profile:main_pages`

Pages are usually listed in `resources/base/profile/main_pages.json`:

```json
{
  "src": [
    "pages/Index"
  ]
}
```

Keep `main_pages.json` in sync with actual `.ets` page paths.

## Package Choices

- Use a single `entry` HAP for simple apps.
- Use a `feature` HAP for independently packaged optional features.
- Use HAR for static shared code/resources across modules or projects.
- Use HSP for dynamic shared code/resources and to reduce duplicated packaging in multi-HAP apps.

Avoid adding HAR/HSP just for organization; start with folders/modules until packaging benefits are real.
