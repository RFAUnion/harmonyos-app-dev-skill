# Tooling And DevEco Studio

## Local Tool Discovery

Check common tools:

```bash
command -v ohpm hvigor hdc node java
find /Applications -maxdepth 3 -iname '*DevEco*' -print
```

DevEco Studio on macOS may bundle tools under:

```text
/Applications/DevEco-Studio.app/Contents/tools/ohpm/bin/ohpm
/Applications/DevEco-Studio.app/Contents/tools/hvigor/bin/hvigorw.js
/Applications/DevEco-Studio.app/Contents/sdk/default/openharmony/toolchains/hdc
/Applications/DevEco-Studio.app/Contents/jbr/Contents/Home
```

Paths and versions vary. Inspect the installed app before assuming.

## Build Workflow

Use:

```bash
ohpm install
hvigor clean assembleApp
```

If `hvigor` is not on `PATH`, run the bundled script with DevEco's Node:

```bash
JAVA_HOME=/Applications/DevEco-Studio.app/Contents/jbr/Contents/Home \
DEVECO_SDK_HOME=/path/to/sdk/default \
/Applications/DevEco-Studio.app/Contents/tools/node/bin/node \
/Applications/DevEco-Studio.app/Contents/tools/hvigor/bin/hvigorw.js clean assembleApp --no-daemon
```

Use project-specific paths from `local.properties` and DevEco SDK Manager.

## SDK Manager Issues

If hvigor reports SDK management mode changes, missing SDK components, or wrong component placement:

1. Prefer opening DevEco Studio and updating SDKs through `Tools > SDK Manager`.
2. Avoid deleting SDK folders under `/Applications` unless the user explicitly asks.
3. If command-line verification is still needed, report the exact SDK path and error; do not hide SDK Manager blockers.

## Signing

Unsigned packages can often compile but cannot be installed normally.

Use DevEco Studio for signing:

- Automatic signing: good for debug scenarios.
- Manual signing: needed for release, restricted permissions, multi-user shared keys, some cross-device/cross-app debugging, and publishing.

Do not invent signing profiles. If signing requires login, certificates, profiles, or restricted permissions, ask the user to authenticate or confirm the specific action.

## Device And Emulator

Check connected devices:

```bash
hdc list targets
```

If empty, open DevEco Studio Device Manager or connect a real device with developer mode enabled.

Use DevEco Studio for:

- Creating projects from official templates.
- SDK Manager.
- ArkUI Preview.
- Simulator/device run.
- Debug signing.
- Log, layout inspector, database inspector, profiler.

Use direct file edits for most code changes; use DevEco UI for environment, signing, preview, and device workflows.
