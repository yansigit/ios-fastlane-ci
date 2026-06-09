# ios-fastlane-ci

Shared iOS release tooling for every app under [@yansigit](https://github.com/yansigit) —
one copy of the fastlane lanes and GitHub Actions workflows, pulled into each app
so there's no per-app drift to maintain.

It was extracted from the [`soniche-ios`](https://github.com/yansigit/soniche-ios)
setup, which is the reference implementation.

## What's here

```
fastlane/Fastfile                    # shared release / beta / screenshots lanes (ENV-driven)
.github/workflows/ios-ci.yml         # reusable: build + unit tests (per push/PR)
.github/workflows/ios-release.yml    # reusable: archive → TestFlight → draft metadata
.github/workflows/ios-screenshots.yml# reusable: capture + frame + (optional) upload
templates/                           # copy-paste starting files for a new app
```

The design goals, identical to soniche-ios:

- **Keyless auth** — App Store Connect **API key** (`ASC_*`), no Apple ID / password / 2FA.
- **`match` in readonly mode** — signing assets live on each app's own `match`
  branch, installed on CI with the built-in `GITHUB_TOKEN` (no extra PAT).
- **No project mutation on CI** — the build number is read from TestFlight and
  injected at build time via `xcargs`, so it works with `GENERATE_INFOPLIST_FILE`
  projects without touching `project.pbxproj`.
- **Screenshots decoupled from release** — slow/flaky UI tests never gate a build.

## Onboard a new app (≈10 minutes)

1. **Copy the template files** from [`templates/`](templates/) into the app repo
   (see [`templates/README.md`](templates/README.md) for the file-by-file list).

2. **Fill in `fastlane/.env.default`** (committed, no secrets) — this is the only
   app-specific config the shared lanes read:

   ```sh
   FL_SCHEME=YourScheme
   FL_BUNDLE_ID=com.sbyoon.YourApp
   FL_PROJECT=YourApp.xcodeproj      # SPM apps; use FL_WORKSPACE=YourApp.xcworkspace for CocoaPods
   ```

3. **Point the app's `fastlane/Fastfile` at this repo**:

   ```ruby
   import_from_git(
     url: "https://github.com/yansigit/ios-fastlane-ci.git",
     branch: "main",
     path: "fastlane/Fastfile"
   )
   ```

4. **Add three thin caller workflows** under `.github/workflows/` (templates in
   [`templates/app-workflows/`](templates/app-workflows/)) that `uses:` the
   reusable workflows here.

5. **Create the signing assets once, locally** (pushes encrypted certs to the
   app's `match` branch):

   ```sh
   bundle exec fastlane match appstore --readonly false
   ```

6. **Add the repo secrets** (Settings → Secrets and variables → Actions):

   | Secret | What |
   |---|---|
   | `ASC_KEY_ID` | App Store Connect API key id |
   | `ASC_ISSUER_ID` | API key issuer id |
   | `ASC_API_KEY_P8` | `base64 -i AuthKey_XXXX.p8` (the whole file, base64) |
   | `MATCH_PASSWORD` | passphrase that encrypts the `match` certs |

That's it. `ci.yml` runs on every push; trigger **iOS Release** / **iOS Screenshots**
from the Actions tab when you want to ship or refresh the store art.

## Config the shared lanes read (ENV)

| Var | Required | Default | Notes |
|---|---|---|---|
| `FL_SCHEME` | ✅ | — | shared Xcode scheme |
| `FL_BUNDLE_ID` | ✅ | — | app identifier |
| `FL_PROJECT` | one of | — | `App.xcodeproj` (SPM apps) |
| `FL_WORKSPACE` | one of | — | `App.xcworkspace` (CocoaPods apps) |
| `FL_OUTPUT_DIR` | | `build` | where the `.ipa` lands |
| `FL_CONFIGURATION` | | `Release` | build configuration |
| `FL_SCREENSHOTS_PATH` | | `./fastlane/screenshots` | frameit output |

Devices / languages / which UI test to run for screenshots live in the app's
`fastlane/Snapfile` (read by `capture_screenshots`).

## Versioning the shared workflows

Apps reference `@main` for convenience. For reproducible releases, tag this repo
(`v1`, `v1.2.0`) and pin callers to a tag instead — bump deliberately.
