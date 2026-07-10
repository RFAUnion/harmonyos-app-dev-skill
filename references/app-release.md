# HarmonyOS App Release And Publishing

Use this reference when the user asks to build a release package, configure signing, upload to AppGallery Connect (AGC), submit an app for review, publish a test or production track, automate release with CI/CD, or fix an app-store rejection.

Source case study: `https://developer.huawei.com/consumer/cn/blog/topic/03217070922107125`

Official entry points:

- `https://developer.huawei.com/consumer/cn/appgallery/devstart/`
- `https://developer.huawei.com/consumer/cn/app/submit/`
- `https://developer.huawei.com/consumer/cn/doc/overview/AppGallery-connect`
- `https://developer.huawei.com/consumer/cn/codelab/AGCPreparation-connectapi/`

## Contents

- [Operating Rules](#operating-rules)
- [Release Inputs](#release-inputs)
- [Project Preflight](#project-preflight)
- [Signing](#signing)
- [Build And Inspect](#build-and-inspect)
- [AGC Upload And Review](#agc-upload-and-review)
- [Automation](#automation)
- [Rejection Loop](#rejection-loop)
- [Post-Release](#post-release)
- [Checklist](#checklist)

## Operating Rules

Treat publishing as a sequence of separately authorized actions:

1. Inspect the project and AGC state.
2. Build and validate a signed release artifact locally.
3. Upload the artifact to AGC or a selected test track.
4. Submit the configured version for review.
5. Release to the selected countries/regions and track.

The first two steps are local and normally safe to automate. Uploading, submitting review, and releasing change external state. Perform them only when the current user request clearly authorizes that action and the authenticated AGC account is available. Before a production release, show the bundle name, version name/code, artifact path and hash, target track, countries/regions, and whether the action is upload, submit, staged rollout, or full release.

Never claim that an app is published merely because a package was built or uploaded. Verify the AGC operation status and, after release, verify the public listing or distribution status.

## Release Inputs

Before starting, collect or inspect:

- The local project path and the intended product/build variant.
- AGC project, app ID, bundle name, signing profile, and target device types.
- Release version name and an incremented integer version code.
- A release signing configuration and access to its `.cer`, `.p12`, and `.p7b` material, or DevEco automatic signing with the correct AGC profile.
- Application name, short introduction, full description, category, supported languages, icon, screenshots, video if required, update notes, and privacy-policy URL.
- Target distribution countries/regions, pricing or free status, test track, staged rollout policy, and release timing.
- Test evidence for the declared device types, privacy flow, permissions, network failure, dark mode, and account deletion where applicable.

If any release input is missing, prepare the local artifact and a blocking checklist instead of guessing store metadata or submitting an incomplete version.

## Project Preflight

Inspect these files before modifying anything:

```text
AppScope/app.json5
build-profile.json5
oh-package.json5
entry/src/main/module.json5
entry/src/main/resources/base/media/
entry/src/main/resources/base/element/
```

Verify the following:

- `bundleName` exactly matches the AGC app package name. Treat it as a long-lived identifier; changing it creates a different app and does not migrate users, ratings, or download history.
- The product's `versionCode` is a new integer greater than the last submitted version. Never reuse or decrement it.
- `versionName` and release notes match the actual user-visible changes.
- The declared `deviceTypes` match the tested support surface. If tablets or foldables are declared, test their layout rather than only stretching the phone UI.
- Launcher icon and start-window resources exist, are referenced through resources, and match the application identity. Use layered foreground/background assets when the project and current AGC requirements call for them.
- Release builds do not contain test endpoints, debug menus, placeholder text, fake data, verbose logs, or unhandled empty/error states.

Do not hardcode current store field limits from a community blog. Treat the blog's suggestions for description length, screenshot count, dimensions, and icon formatting as a preflight hint, then verify the current AGC form and official review requirements at submission time.

## Signing

Use automatic signing for local debugging only when appropriate. For a store release, verify that the artifact is signed by the profile registered for the same AGC app.

Manual signing usually involves:

- `.cer` certificate;
- `.p12` keystore;
- `.p7b` signing profile;
- key alias and passwords;
- a release entry in `build-profile.json5`.

Keep signing files outside the repository or in an encrypted CI secret store. Never commit certificates, keystores, profiles, passwords, client secrets, or API tokens to Git. Do not print them in build logs. If a signing profile is rotated, confirm whether AGC requires an update of the app signing configuration before uploading a new package.

For a signing mismatch, compare the app package name, signing certificate/profile, product variant, and artifact actually selected for upload. Rebuilding with a different local profile is not a fix unless that profile is registered for the AGC app.

## Permission And Privacy Preflight

For every sensitive permission in `module.json5`:

- request only what the current feature needs;
- give a concrete user-facing `reason` that describes the feature and data flow;
- provide the correct `usedScene` abilities and timing;
- request optional permissions when the user reaches the feature, not on first launch without context;
- verify the permission is actually used and can be denied gracefully.

Before collecting personal or device data, show the app's privacy agreement and wait for consent according to the current compliance requirements. Make the privacy policy reachable inside the app and ensure its URL works without an authenticated session. Provide account cancellation/deletion where the app has accounts, and keep data collection, sharing, retention, and third-party SDK disclosures synchronized with the actual build.

Do not use a generic reason such as “need camera permission.” Explain the user action and purpose, such as taking a product photo for an upload. A permission that cannot be demonstrated in the submitted build is a review risk.

## Build And Inspect

Prefer the project's existing hvigor tasks and product names. Inspect available tasks before assuming a command:

```bash
./hvigorw --tasks
ohpm install
./hvigorw clean assembleHap -p product=release --no-daemon
```

For a multi-module store artifact, use the project task that produces the required signed `.app`; a signed `.hap` may be appropriate for module testing but may not be the package AGC expects for the final store release. Confirm the output format in the current AGC upload form and the project's configured product.

After building:

- locate the actual output under the project's build output directory;
- verify the artifact exists, is signed, and belongs to the intended product;
- record its SHA-256 hash and size;
- inspect the package metadata if the available DevEco/HarmonyOS tools support it;
- retain obfuscation mapping files and release symbols in protected storage;
- install the same artifact on representative real devices and run the primary user paths.

The blog mentions commands such as `assembleHap` and an `agconnect publish` example. Treat the latter as illustrative until the installed official AGC CLI/API is verified. Do not invent or silently substitute a third-party upload command.

## AGC Upload And Review

The normal portal flow is:

1. Sign in to AppGallery Connect with a verified developer or team account.
2. Create the project and app if they do not exist, or open the existing app.
3. Confirm package name, app ID, language, category, device types, and signing profile.
4. Upload the signed package to the software-package area.
5. Create a version and select the uploaded package.
6. Complete application information, distribution information, privacy declarations, screenshots, update notes, and any required qualifications.
7. Choose internal/invite/public testing or formal release as appropriate.
8. Run AGC checks, resolve errors, submit for review, and monitor the review result.

Keep test distribution separate from production release. Uploading to a test channel can be a useful automated smoke gate; formal review and full release should remain explicit decisions unless the user specifically authorized a release policy that covers them.

## Automation

For CI/CD, use this staged pipeline:

```text
tag or manual dispatch
  -> install dependencies
  -> lint and unit tests
  -> build signed release artifact
  -> package and metadata preflight
  -> upload to AGC test/draft channel
  -> collect operation status
  -> human approval
  -> submit review or release to the selected track
```

The official AGC Connect API requires an API client created in AGC under the project/team access settings. Store its client ID and secret in CI secrets or a system keychain, scope access to the minimum project, and rotate it when needed. Prefer the current official Connect API or an officially documented CLI over commands copied from an article.

When Codex automates a release, it should:

- inspect the repository and refuse to build from an unexpected dirty state unless the user approves it;
- verify version monotonicity and bundle-name identity;
- build and test before any network upload;
- generate a release report with artifact hash and preflight results;
- upload only to the explicitly selected app and track;
- return AGC operation IDs/status and stop on errors;
- ask for confirmation before submit/release when the current request did not already authorize that exact action.

Do not put AGC credentials in the skill, project files, shell history, prompts, or generated reports.

## Rejection Loop

When AGC rejects a version, preserve the rejection text and map it to the responsible area:

- permission or privacy declaration;
- missing, blank, or broken functionality;
- package/signing mismatch;
- metadata, icon, screenshot, or category mismatch;
- device compatibility or layout failure;
- crash, ANR, network timeout, or unavailable backend;
- policy, qualification, copyright, or content issue.

Make the smallest corrective change, increment `versionCode`, rebuild a fresh signed artifact, rerun the full preflight, and resubmit. Do not upload the same rejected artifact under a different description. Keep a short changelog of rejection reason, fix, test evidence, and new version code.

## Post-Release

After release, check the listing, download/install flow, first launch, privacy prompt, login, primary feature path, and crash/performance dashboards. Watch crash rate, startup time, page loading, battery/thermal behavior, user reviews, and compliance changes. Update the privacy policy and store metadata whenever data collection, permissions, third-party SDKs, or user-facing functionality changes.

For staged rollout, define an observation window and rollback/stop criteria before increasing the rollout percentage. A successful review is not proof that the release is healthy on every declared device.

## Checklist

- [ ] AGC app/project and bundle name match the local project.
- [ ] App ID and release signing profile are correct.
- [ ] `versionCode` increased; `versionName` and notes match the build.
- [ ] Signed `.app` or `.hap` output matches the target AGC upload flow.
- [ ] Artifact hash, size, product, and signature were recorded.
- [ ] Sensitive permissions are minimal, justified, and gracefully denied.
- [ ] Privacy agreement, policy URL, data disclosures, and account deletion are ready.
- [ ] Icon, start window, screenshots, descriptions, category, language, and regions are current.
- [ ] Main flows, empty/error/network cases, dark mode, and declared devices were tested.
- [ ] Release logs/debug code/test endpoints are removed or disabled.
- [ ] Obfuscation mapping/symbol files are retained securely if obfuscation is enabled.
- [ ] Target track and external action are explicitly confirmed.
- [ ] AGC upload/review/release status was verified after the operation.
