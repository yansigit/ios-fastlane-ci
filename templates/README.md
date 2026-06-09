# Templates — onboard a new iOS app

Copy these into a new app repo, then fill in the `<PLACEHOLDERS>`. See the
top-level [README](../README.md) for the full 6-step onboarding.

| Copy this template | To (in the app repo) | Edit |
|---|---|---|
| `app-fastlane/Gemfile` | `Gemfile` | — |
| `app-fastlane/Fastfile` | `fastlane/Fastfile` | — (imports the shared lanes) |
| `app-fastlane/Appfile` | `fastlane/Appfile` | bundle id |
| `app-fastlane/Matchfile` | `fastlane/Matchfile` | git_url + bundle id |
| `app-fastlane/Snapfile` | `fastlane/Snapfile` | scheme, UI test, devices |
| `app-fastlane/.env.default` | `fastlane/.env.default` | scheme, bundle id, project/workspace |
| `app-fastlane/Framefile.json` | `fastlane/screenshots/Framefile.json` | — |
| `app-fastlane/title.strings` | `fastlane/screenshots/en-US/title.strings` | captions |
| `app-workflows/ci.yml` | `.github/workflows/ci.yml` | scheme, project, test target |
| `app-workflows/ios-release.yml` | `.github/workflows/ios-release.yml` | — |
| `app-workflows/ios-screenshots.yml` | `.github/workflows/ios-screenshots.yml` | — |
| `app-workflows/dependabot.yml` | `.github/dependabot.yml` | — |

You also need, in the app target:

- A shared scheme (Xcode ▸ Product ▸ Scheme ▸ Manage Schemes ▸ Shared).
- `fastlane/frameit/background.png` + an `Inter-Bold.ttf` (frameit caption art).
- A screenshot UI test + `SnapshotHelper.swift` in the UITest target, and a
  DEBUG-only fixture/launch-arg seam so screens render on the simulator without
  hitting the network/camera/model. (See `soniche-ios` or `TelluricAI` for a
  worked example.)
