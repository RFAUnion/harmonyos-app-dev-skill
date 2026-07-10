---
name: harmonyos-app-dev
description: Native HarmonyOS/OpenHarmony app development with ArkTS, ArkUI, Stage model, UIAbility, DevEco Studio, ohpm, hvigor, app/module JSON5, signing, debugging, HDS immersive light, liquid tab indicators, responsive multi-device layouts, spatialization planning, one-shot transitions, and build troubleshooting. Use for building, modifying, debugging, reviewing, or scaffolding HarmonyOS apps, especially when the user mentions 鸿蒙应用开发, 原生鸿蒙, HarmonyOS NEXT, HarmonyOS 7, ArkTS, ArkUI, DevEco Studio, UIAbility, Stage模型, HAP, HAR, HSP, module.json5, app.json5, ohpm, hvigor, 沉浸光感, 沉浸导航, 悬浮导航, 悬浮页签, 液态指示器, 液态TabBar, 多端适配, 平板适配, 折叠屏适配, 多窗口, 空间化, HdsNavigation, HdsTabs, HdsMiniBarButton, UIDesignKit, hds_button, hdsMaterial, systemMaterialEffect, MaterialType.ADAPTIVE, SymbolGlyphModifier, barFloatingStyle, setWindowLayoutFullScreen, 一镜到底, 共享元素转场, geometryTransition, NodeController, NodeContainer, customNavContentTransition, or componentSnapshot.
---

# HarmonyOS App Development

Use this skill for native HarmonyOS/OpenHarmony application work. Prefer the official stack unless the existing project clearly uses another pattern:

- Stage model
- ArkTS
- ArkUI declarative UI
- UIAbility
- DevEco Studio, ohpm, and hvigor

Do not treat this skill as a static API dump. HarmonyOS documentation and SDK behavior change quickly; when exact API signatures, version support, permissions, or release requirements matter, verify against the official Huawei docs or the installed SDK before finalizing.

## Workflow

1. Inspect the project before changing files.
   - Check `AppScope/app.json5`, root `build-profile.json5`, `oh-package.json5`, `hvigor/hvigor-config.json5`, and each module's `src/main/module.json5`.
   - Find ArkTS sources under `entry/src/main/ets` or other module `src/main/ets`.
   - Reuse the existing module names, resource naming, and page routing style.

2. Choose the right reference only when needed.
   - For platform model and vocabulary, read `references/overview.md`.
   - For folders, config, packages, and module types, read `references/project-structure.md`.
   - For ArkTS and ArkUI page work, read `references/arkts-arkui.md`.
   - For UIAbility, lifecycle, startup, and page loading, read `references/ability-stage.md`.
   - For preferences, KV, relational data, and sharing, read `references/data-storage.md`.
   - For DevEco Studio, SDK Manager, ohpm, hvigor, signing, and device debugging, read `references/tooling-devstudio.md`.
   - For HarmonyOS 6.1 immersive light, floating/immersive navigation, `HdsNavigation`, `HdsTabs`, and `hdsMaterial`, read `references/immersive-navigation.md`.
   - For liquid tab indicators, `HdsMiniBarButton`, custom tab-bar overlays, drag/snap behavior, and API 60100+ full-screen safe-area handling, read `references/immersive-tabbar-indicator.md`.
   - For HarmonyOS 7 spatialization planning, phone/tablet/foldable/multi-window layouts, workbench/list-detail/form/empty/settings priorities, read `references/multidevice-layout.md`.
   - For one-shot / shared-element / shared-container page transition animation, `geometryTransition`, `NodeController`, `NodeContainer`, `customNavContentTransition`, and `componentSnapshot`, read `references/one-shot-transition.md`.

3. Edit narrowly.
   - Put UI screens in `.ets` files under `src/main/ets/pages` unless the project uses Navigation destinations elsewhere.
   - Put app startup/window code in the relevant `UIAbility` file.
   - Put app metadata in `AppScope/app.json5`.
   - Put module components, pages, permissions, device types, and extension abilities in `module.json5`.
   - Put strings, colors, media, and profile JSON under `src/main/resources`.

4. Validate with the local toolchain when available.
   - Run `ohpm install` if dependencies changed or `oh_modules` is missing.
   - Run `hvigor clean assembleApp` or the project-specific task.
   - If command-line hvigor fails due to SDK Manager, Java, or signing configuration, report the exact blocker and use DevEco Studio for SDK/signing/device steps.

5. Keep runtime constraints visible.
   - Unsigned HAP/APP packages may build but cannot be installed normally.
   - Auto signing is for debugging; publishing and restricted permissions generally require manual signing/profile configuration.
   - Simulator behavior can differ from real devices for APIs, sensors, media, concurrency, and distributed capabilities.

## Official Documentation Entry Points

Use these official pages as canonical starting points when live verification is needed:

- Documentation center: `https://developer.huawei.com/consumer/cn/doc/`
- Application development guide: `https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide`
- First ArkTS app: `https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/start-with-ets-stage`
- ArkUI overview: `https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-overview`
- ArkTS overview: `https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-overview`
- Ability Kit overview: `https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/abilitykit-overview`
- Tooling overview: `https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-tools-overview`
- Floating navigation and immersive light case study: `https://developer.huawei.com/consumer/cn/blog/topic/03212369425758084`
- Immersive light tab-bar indicator optimization: `https://developer.huawei.com/consumer/cn/blog/topic/03212848303315412`
- HDS immersive light material guide: `https://developer.huawei.com/consumer/cn/blog/topic/03211806266023148`
- HarmonyOS 7 spatialization and multi-device layout guide: `https://developer.huawei.com/consumer/cn/blog/topic/03218972610058011`
- One-shot transition best practice: `https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-one-shot-to-the-end`
