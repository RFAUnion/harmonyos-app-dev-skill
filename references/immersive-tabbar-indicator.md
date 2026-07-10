# Immersive Light Tab-Bar Indicator

Use this reference when the user asks for the liquid/elastic selected-tab indicator described in the Huawei developer blog, `HdsMiniBarButton`, `hds_button`, custom `HdsTabs` indicator overlays, or a HarmonyOS 6.1 immersive-light TabBar with drag feedback.

Source case study: `https://developer.huawei.com/consumer/cn/blog/topic/03212848303315412`

## What The Blog Optimizes

The case study keeps the immersive-light `HdsNavigation` + `HdsTabs` shell, but improves selected-tab recognition:

- The old filled-versus-outline icon change is not always legible through a translucent material.
- A highlight placed at the same visual level as the icon can obscure the icon during a swipe.
- Add a separate liquid indicator with a translucent, rounded, material-like surface.
- Keep the icon and label inside the indicator, and let the indicator move above the tab bar while the real `HdsTabs` remains the navigation source of truth.

Do not replace `HdsTabs` with a second tab implementation. The custom indicator is a visual and gesture layer that must stay synchronized with the tab controller.

## Dependency And Version Boundary

The blog sample uses these APIs and packages:

```ts
import { HdsNavigation, HdsTabs, ScrollEffectType } from '@hms.hds.hdsBaseComponent';
import { hdsEffect, hdsMaterial, HdsNavigationTitleMode, HdsTabsController } from '@kit.UIDesignKit';
import { deviceInfo } from '@kit.BasicServicesKit';
import { HdsMiniBarButton } from 'hds_button';
```

Treat `HdsMiniBarButton` and the `hds_button` package as an optional dependency from the case study, not as a guaranteed core ArkUI component. Check the installed HarmonyOS SDK, UIDesignKit package, and project dependency configuration before generating imports.

The sample targets HarmonyOS 6.1 / API 60100 or later and guards the shell with `deviceInfo.distributionOSApiVersion >= 60100`. Keep a plain `Tabs` or non-liquid `HdsTabs` fallback for unsupported devices and for projects whose SDK does not expose these symbols.

## Layering Pattern

Use a root `Stack` with two sibling layers:

```text
Stack (bottom aligned)
├── HdsNavigation
│   └── HdsTabs and TabContent pages
└── indicator overlay
    └── HdsMiniBarButton plus icon/label and drag gesture
```

The overlay must be visually above `HdsNavigation` and positioned in the same floating-bar region. Keep the indicator width and the floating tab-bar width aligned. Decide hit testing deliberately: the overlay needs to receive its drag gesture without stealing unrelated page or tab-bar taps.

The blog also configures the title bar with `ScrollEffectType.IMMERSIVE_GRADIENT_BLUR` and ignores selected system safe-area edges for a full-screen effect. Make both behaviors opt-in; they change the usable content region and require explicit content avoidance.

For the base shell, retain the floating-bar settings from `immersive-navigation.md`. The case study additionally uses a fixed 330 vp bar, disables HdsTabs' own animation while the custom indicator owns the motion, and enables handedness adaptation:

```ts
HdsTabs({ controller: this.tabsController, index: this.activeTabIndex })
  .barHeight(59)
  .animationDuration(0)
  .scrollable(false)
  .vertical(false)
  .barPosition(BarPosition.End)
  .barOverlap(true)
  .barFloatingStyle({
    barWidth: { smallWidth: 330, mediumWidth: 330, largeWidth: 330 },
    adaptToHandedness: true,
    barBottomMargin: 28,
    systemMaterialEffect: this.buildAdaptiveMaterialEffect()
  })
```

Use the fixed width only when it matches the product design. For a responsive shell, measure the actual bar and derive indicator anchors from that measurement.

## State Model

Keep a small, explicit state machine instead of deriving the indicator position from several unrelated UI states:

- `activeTabIndex`: the index currently owned by `HdsTabs`.
- `currentIndicatorIndex`: the last snapped indicator anchor.
- `displayIndex`: the icon and label shown inside the moving indicator.
- `dragOffsetX`: the temporary offset during a drag.
- `isMoving`: whether the indicator is in a drag or snap animation.
- `isProgrammaticChange`: prevents a drag-triggered controller update from recursively handling the normal tab change callback.
- `animatedBarWidth` and `animatedBarHeight`: base dimensions plus velocity deformation.

Use one synchronization direction for each event:

1. Tab tap or page swipe: `HdsTabs.onChange` updates `activeTabIndex`, then spring-animates the indicator to that anchor.
2. Drag update: update only the indicator offset and visual deformation; do not change the tab index every frame.
3. Drag end: convert the offset to a slot count, clamp the new index, animate the indicator to the anchor, then call `tabsController.changeIndex(newIndex)` and update `activeTabIndex` once.

Prefer an animation completion callback when the SDK provides one. If a timer is unavoidable, guard it against a second drag, page disappearance, and interrupted animation.

## Responsive Anchors

Do not assume the blog's four-tab, 330 vp geometry is universal. Keep base anchor positions only as a design calibration, then scale them from the actual overlay width:

```ts
private readonly BASE_CONTAINER_WIDTH: number = 330;
private readonly BASE_POSITIONS: number[] = [8.5, 85, 162.5, 241];
@State private indicatorContainerWidth: number = this.BASE_CONTAINER_WIDTH;

private getAnchorPositions(): number[] {
  const scale = this.indicatorContainerWidth / this.BASE_CONTAINER_WIDTH;
  return this.BASE_POSITIONS.map((position: number) => position * scale);
}
```

Update `indicatorContainerWidth` from `onAreaChange`. For a different tab count or a non-fixed tab layout, calculate anchors from measured slot centers instead of reusing the four-tab constants. Keep all dimensions in `vp` and give the overlay stable width/height constraints so measurement changes do not jump the layout.

## Liquid Motion

The case study's motion model can be implemented with these stages:

1. On drag start, set `isMoving`, reset the previous offset, smoothed velocity, direction, and pause markers.
2. On drag update, compute per-frame velocity from the offset delta, smooth it, and map it to a bounded deformation amount.
3. Stretch width and compress height while moving in the initial direction; reverse the relationship when the direction changes.
4. Apply resistance at the first and last tab, for example by multiplying out-of-range offset by `0.3`.
5. Find the nearest anchor. While far from an anchor, blur the icon and label slightly; reduce blur near an anchor so the selected state becomes readable.
6. On release, round `dragOffsetX / slotWidth`, clamp the index, and spring-snap the indicator back to zero offset at the new anchor.

Bound deformation with a maximum value and ignore tiny velocities. A practical starting set from the case study is a 0.1 vp/frame velocity threshold, 25 vp/frame normalization base, and 45 vp maximum deformation; tune these values against the actual bar size and device frame pacing.

The indicator can use `HdsMiniBarButton` with a transparent mask and immersive material. Reuse the capability check in `immersive-navigation.md`; do not unconditionally select `IMMERSIVE` + `EXQUISITE` when the device does not support that material level.

If the installed component exposes point-light configuration, the blog's `hdsEffect.PointLightIlluminatedType.BORDER_CONTENT` setting lights both the indicator border and its content. Keep this as a tuning option; if the highlight competes with the icon or label, lower the material level or illuminate only the border.

## Safe Area And Full-Screen Layout

The case study uses full-screen window layout so the custom indicator can extend into the system navigation area. This has an app-wide consequence:

- Apply the window full-screen setting only when the product design requires it and verify the exact `Window` API in the installed SDK.
- If `ignoreLayoutSafeArea` is used, manually avoid status-bar, title-bar, and bottom-navigation collisions.
- Use `expandSafeArea` for the bottom overlay only after checking that the last actionable content is still reachable.
- Keep enough bottom padding for the floating bar, indicator overflow, and any floating action button.
- Test gesture hit regions and content visibility on devices with different navigation modes, screen sizes, and handedness settings.

Do not treat `setWindowLayoutFullScreen(true)` as a cosmetic switch. It changes layout assumptions for every sibling component.

## Material And Accessibility Rules

- Preserve the material hierarchy: indicator, tab bar, and title bar should not all compete with equally strong highlights.
- Keep icon and label contrast readable in both light and dark modes; the liquid effect is not a substitute for a clear selected state.
- Do not use blur as the only state signal. Filled/outline icon changes, label styling, or a restrained tint should remain available.
- Reduce deformation, blur, and animation duration when performance or reduced-motion requirements call for it.
- Keep the indicator's gesture target stable and large enough, while preventing it from covering neighboring tab targets.

## Validation Checklist

- Compile with the exact SDK and dependencies used by the project; validate `@hms.hds.hdsBaseComponent`, `@kit.UIDesignKit`, `@kit.BasicServicesKit`, and `hds_button` imports.
- Verify the API 60100+ branch and the fallback branch on an actual device or supported emulator.
- Tap every tab, swipe the page, drag the indicator, reverse direction mid-drag, drag beyond both edges, and release near each anchor.
- Resize or rotate the window and confirm anchors are recalculated without visual jumps.
- Confirm the indicator and tab controller cannot enter different indices after rapid taps or interrupted animations.
- Check the final list row, bottom buttons, system gestures, title bar, and floating action button for overlap.
- Test light/dark mode, high contrast, reduced motion, and handedness adaptation.
- Profile list-heavy pages and long drags; lower material intensity or deformation complexity if frame pacing drops.

## Common Pitfalls

- Copying the blog's fixed four-tab coordinates into a responsive layout.
- Making the indicator the navigation source of truth instead of synchronizing it with `HdsTabsController`.
- Updating the selected tab on every drag frame, causing page churn and feedback loops.
- Leaving the overlay gesture active after the page disappears.
- Setting full-screen layout without compensating for safe areas.
- Hardcoding `IMMERSIVE` and `EXQUISITE` without checking device material support.
- Using blur or highlight alone to communicate selection, especially on translucent backgrounds.
- Assuming the case-study `hds_button` dependency is present in every DevEco Studio project.
