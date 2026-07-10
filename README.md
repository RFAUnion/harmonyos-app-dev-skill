# HarmonyOS App Dev Skill

A Codex skill for native HarmonyOS/OpenHarmony application development.

It helps Codex build, modify, review, and debug HarmonyOS apps using ArkTS, ArkUI, the Stage model, UIAbility, DevEco Studio, ohpm, hvigor, app/module JSON5 configuration, signing/debugging workflows, HarmonyOS 6.1 immersive navigation patterns such as `HdsNavigation`, `HdsTabs`, floating tabs, and immersive light material, plus one-shot/shared-element transitions using `geometryTransition`, `NodeController`, `NodeContainer`, and `Navigation.customNavContentTransition`.

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

The skill summarizes official Huawei developer documentation and keeps official links in `SKILL.md` for live verification when exact API signatures, SDK version support, signing, or device behavior matters.

## License

MIT
