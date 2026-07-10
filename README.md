# HarmonyOS App Dev Skill

A Codex skill for native HarmonyOS/OpenHarmony application development.

It helps Codex build, modify, review, debug, and publish HarmonyOS apps using ArkTS, ArkUI, the Stage model, UIAbility, DevEco Studio, ohpm, hvigor, app/module JSON5 configuration, signing/debugging workflows, and AppGallery Connect release automation. It covers HarmonyOS 6.1 immersive navigation patterns such as `HdsNavigation`, `HdsTabs`, floating tabs, HDS immersive light material, and liquid tab indicators with `HdsMiniBarButton`, plus HarmonyOS 7 spatialization planning, responsive layouts for phones/tablets/foldables/multi-window apps, and one-shot/shared-element transitions.

## Install

Clone this repository into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/RFAUnion/harmonyos-app-dev-skill.git ~/.codex/skills/harmonyos-app-dev
```

Restart Codex or start a new session so the skill is discovered.

## Use

Ask Codex about native HarmonyOS app work, for example:

```text
Use $harmonyos-app-dev to add a HarmonyOS 6.1 floating navigation shell with immersive light material to this ArkTS app.
```

## References

The skill summarizes official Huawei developer documentation and Huawei developer case studies. It keeps official links in `SKILL.md` for live verification when exact API signatures, SDK version support, signing, or device behavior matters.

## License

MIT
