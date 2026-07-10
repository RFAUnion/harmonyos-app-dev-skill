# Stage Model And UIAbility

## UIAbility Role

`UIAbility` is a component with UI and system lifecycle callbacks. It usually owns a `WindowStage` and loads the first ArkUI page.

For most apps, prefer one `UIAbility` plus multiple pages. Add more abilities only for separate tasks, separate windows, or system-level component boundaries.

## Common Lifecycle

Core callbacks:

- `onCreate(want, launchParam)`: Ability instance is created. Do one-time lightweight initialization.
- `onWindowStageCreate(windowStage)`: Main window stage is created. Load UI content here.
- `onForeground()`: Ability enters foreground.
- `onBackground()`: Ability enters background.
- `onWindowStageDestroy()`: Window resources are being destroyed.
- `onDestroy()`: Ability is destroyed.
- `onNewWant(want, launchParam)`: Existing ability receives a new Want.

Keep lifecycle callbacks lightweight. Move expensive work to async flows, background tasks, workers, or app services as appropriate.

## Standard EntryAbility Pattern

```ts
import { AbilityConstant, ConfigurationConstant, UIAbility, Want } from '@kit.AbilityKit';
import { hilog } from '@kit.PerformanceAnalysisKit';
import { window } from '@kit.ArkUI';

const DOMAIN = 0x0000;

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    try {
      this.context.getApplicationContext().setColorMode(ConfigurationConstant.ColorMode.COLOR_MODE_NOT_SET);
    } catch (err) {
      hilog.error(DOMAIN, 'EntryAbility', 'Failed to set colorMode. Cause: %{public}s', JSON.stringify(err));
    }
    hilog.info(DOMAIN, 'EntryAbility', 'Ability onCreate');
  }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        hilog.error(DOMAIN, 'EntryAbility', 'Failed to load content. Cause: %{public}s', JSON.stringify(err));
        return;
      }
      hilog.info(DOMAIN, 'EntryAbility', 'Succeeded in loading content.');
    });
  }
}
```

Adapt imports and callback signatures to the installed SDK if compiler errors indicate API changes.

## `module.json5` Ability Declaration

Typical UIAbility declaration:

```json5
{
  "name": "EntryAbility",
  "srcEntry": "./ets/entryability/EntryAbility.ets",
  "description": "$string:EntryAbility_desc",
  "icon": "$media:layered_image",
  "label": "$string:EntryAbility_label",
  "startWindowIcon": "$media:startIcon",
  "startWindowBackground": "$color:start_window_background",
  "exported": true,
  "skills": [
    {
      "entities": ["entity.system.home"],
      "actions": ["ohos.want.action.home"]
    }
  ]
}
```

Verify launcher action/entity names against current docs and existing project conventions. Different templates or SDK versions may use slightly different action strings.

## Backup Extension

Some templates add a backup extension ability. Keep it only if the app needs backup/restore integration or the generated project relies on it. Remove unused extension abilities only after deleting their source files and profile metadata.
