# Immersive Light And Floating Navigation

Use this reference when the user asks for `沉浸光感`, `沉浸导航`, `悬浮导航`, `悬浮页签`, `HdsNavigation`, `HdsTabs`, `UIDesignKit`, `hdsMaterial`, `systemMaterialEffect`, or HarmonyOS 6.1 visual shell work.

Source case studies:

- `https://developer.huawei.com/consumer/cn/blog/topic/03212369425758084`
- `https://developer.huawei.com/consumer/cn/blog/topic/03211806266023148`

## Design Intent

The Huawei developer blog case study describes a HarmonyOS 6.1 pattern for a "floating navigation + immersive light" tab shell:

- Do not make the bottom tab bar clamp the page content to a hard bottom edge.
- Let content extend behind or below the floating bar so the page feels lighter.
- Keep the glass/material feeling adaptive and stable; do not chase stronger material at the cost of frame drops, heat, or accidental touches.
- Split responsibilities: `HdsNavigation` owns the page shell and system-level navigation/material context; `HdsTabs` owns tab switching and the floating tab bar; a custom floating action button lives in a separate overlay layer.

Prefer this pattern over a plain fixed `Tabs` bottom bar when the UI needs a premium immersive shell and the target SDK supports the related UIDesignKit APIs.

## Imports

Typical imports:

```ts
import { HdsNavigation, HdsNavigationTitleMode, HdsTabs, HdsTabsController, SystemMaterialParams, hdsMaterial } from '@kit.UIDesignKit';
import { deviceInfo } from '@kit.BasicServicesKit';
```

Adapt exact symbols to the installed SDK. If these imports fail, verify HarmonyOS SDK/API level and UIDesignKit availability in DevEco Studio SDK Manager.

## Material Capability And Version Strategy

The HDS material case study targets HarmonyOS 6.1.0(23) and exposes `systemMaterialEffect` through `@kit.UIDesignKit`. The compile SDK must expose these symbols before a runtime check can help. For runtime compatibility, use the version API supported by the installed SDK; the case study uses `deviceInfo.sdkApiVersion >= 23`, while another sample may use `distributionOSApiVersion >= 60100`. These values are different representations; do not mix them or invent a threshold without checking the SDK.

Prefer the system's adaptive choice for normal production UI. Only force immersive/exquisite when the product genuinely needs the stronger visual feedback and the device capability check passes.

```ts
@State materialType: hdsMaterial.MaterialType = hdsMaterial.MaterialType.ADAPTIVE;
@State materialLevel: hdsMaterial.MaterialLevel = hdsMaterial.MaterialLevel.ADAPTIVE;

aboutToAppear(): void {
  this.resolveMaterialEffect();
}

private resolveMaterialEffect(): void {
  // Keep this branch aligned with the SDK version used by the project.
  if (deviceInfo.sdkApiVersion < 23) {
    this.materialType = hdsMaterial.MaterialType.BACKGROUND_BLUR;
    this.materialLevel = hdsMaterial.MaterialLevel.SMOOTH;
    return;
  }

  let materialTypes: Array<hdsMaterial.MaterialType> =
    hdsMaterial.getSystemMaterialTypes();
  if (materialTypes.indexOf(hdsMaterial.MaterialType.IMMERSIVE) < 0) {
    this.materialType = hdsMaterial.MaterialType.BACKGROUND_BLUR;
    this.materialLevel = hdsMaterial.MaterialLevel.SMOOTH;
    return;
  }
  this.materialType = hdsMaterial.MaterialType.ADAPTIVE;
  this.materialLevel = hdsMaterial.MaterialLevel.ADAPTIVE;
}

private buildAdaptiveMaterialEffect(): SystemMaterialParams {
  return {
    materialType: this.materialType,
    materialLevel: this.materialLevel
  };
}
```

For a deliberate high-end mode, initialize `materialType` to `IMMERSIVE` and `materialLevel` to `EXQUISITE`, then downgrade to `BACKGROUND_BLUR` + `SMOOTH` when `IMMERSIVE` is not listed by `getSystemMaterialTypes()`. This path costs more power and can reduce frame pacing on weaker devices.

Material levels commonly used by the case study:

- `ADAPTIVE`: system-selected balance for ordinary UI.
- `EXQUISITE`: strongest light and reflection feedback; use selectively.
- `GENTLE`: softer light response.
- `SMOOTH`: lowest-cost fallback, typically paired with `BACKGROUND_BLUR`.

## Floating Tab Bar

Use `HdsTabs` for the tab controller and floating bottom bar. The key combination is:

- `barPosition(BarPosition.End)` to place the bar at the bottom.
- `barMode(BarMode.Fixed)` for a stable bottom tab shell.
- `barOverlap(true)` so the bar floats over the content instead of reserving hard layout space.
- `barFloatingStyle(...)` for side margin, bottom margin, thermal control, and system material.

```ts
private tabsController: HdsTabsController = new HdsTabsController();
@State activeTab: number = 0;

HdsTabs({ controller: this.tabsController, index: this.activeTab }) {
  // TabContent(...)
}
.vertical(false)
.barPosition(BarPosition.End)
.barMode(BarMode.Fixed)
.scrollable(false)
.animationDuration(240)
.barHeight(60)
.barOverlap(true)
.barFloatingStyle({
  barSideMargin: 18,
  barBottomMargin: 28,
  thermoCtrl: true,
  systemMaterialEffect: this.buildAdaptiveMaterialEffect()
})
```

Keep content bottom padding large enough that important controls do not sit under the floating bar and floating action button:

```ts
private getTabContentBottomPadding(): number {
  return 176;
}
```

Tune the padding for the actual bar height, bottom margin, safe area, and overlay button footprint.

## HDS Tab Item Styling

The material effect is only one part of selected-state clarity. For system Symbol icons, use `SymbolGlyphModifier` to keep normal and selected glyphs distinct and use multi-color rendering when the design calls for it:

```ts
interface MenuItem {
  normal: SymbolGlyphModifier;
  selected: SymbolGlyphModifier;
  label: string;
}

const item = new BottomTabBarStyle({
  normal: menuItem.normal,
  selected: menuItem.selected
}, menuItem.label).labelStyle({
  selectedColor: $r('app.color.primary_blue')
});
```

Keep the selected icon/label contrast readable in light and dark modes. The material highlight, blur, or point-light response must not be the only signal that a tab is selected. When adding a custom liquid indicator, route the detailed gesture and overlay rules through `immersive-tabbar-indicator.md`.

## Page Shell

Wrap the page in `HdsNavigation` when building this visual shell. Keep title mode compact for immersive home/discovery screens unless the app needs a full standard title area.

```ts
HdsNavigation() {
  Stack() {
    this.buildTabs()
    this.buildFloatingActionButton()
  }
}
.mode(NavigationMode.Stack)
.titleMode(HdsNavigationTitleMode.MINI)
```

If the project already uses a different navigation architecture, adapt the shell instead of rewriting unrelated routing.

## Floating Action Button Layer

Do not put the primary floating action button inside the tab bar when it needs independent expansion, left/right docking, grip-awareness, or custom animation. Place it as a top `Stack` overlay and drive it with state such as `activeTab` and docking side.

Implementation guidance:

- Keep hit testing and visual bounds explicit so the floating button does not steal tab gestures.
- Use `activeTab` to update label/action per tab.
- Keep docking logic separate from tab switching logic.
- If using hand-grip or posture sensing, hide it behind a small service/helper and keep manual fallback behavior for testing.

## Validation Checklist

- Verify the SDK/API level supports `@kit.UIDesignKit` and `hdsMaterial`.
- Compile with hvigor after adding UIDesignKit imports.
- Test on a real device when possible; material effects, thermal control, and safe-area behavior may differ from Preview or Simulator.
- Check light/dark modes and high-contrast accessibility.
- Scroll long content behind the floating bar and confirm the last actionable row is not obscured.
- Check tap targets around the floating bar and floating action button for accidental touches.
- Profile frame pacing on list-heavy pages; reduce material intensity or animation complexity if scrolling drops frames.

## Common Pitfalls

- Hardcoding immersive material without checking `getSystemMaterialTypes()`.
- Placing the custom floating action button inside `HdsTabs`, which couples its layout and animations to tab internals.
- Forgetting bottom padding after `barOverlap(true)`.
- Mixing page content, tab navigation, and material selection in one component until the shell is hard to modify.
- Assuming the Preview result exactly matches device material rendering.
