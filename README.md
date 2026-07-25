# expo-go-unsigned-build

Builds the [Expo Go](https://github.com/expo/expo/tree/main/apps/expo-go) app from
source as an **unsigned `.ipa`**, so it can be sideloaded with
[SideStore](https://sidestore.io/) / AltStore (which re-sign it with a free
Apple ID at install time and renew it every 7 days).

## Why

The Expo Go on the App Store is stuck on **SDK 54** — Expo has a newer build
sitting in Apple's review queue, see
[Expo Go and the App Store, May 2026](https://expo.dev/changelog/expo-go-and-app-store-may-2026).
The official builds Expo publishes for newer SDKs
(`expo/expo-go-releases`) are **simulator-only** (`DTPlatformName =
iphonesimulator`), so they can't be sideloaded onto a real iPhone. The remaining
official route for a device is `eas go`, which needs a paid Apple Developer
account to register the device.

Building from source and letting SideStore do the signing sidesteps all of that.

## Usage

Actions → **Build Expo Go (unsigned IPA)** → *Run workflow*, pick the
`expo/expo` branch (`sdk-57`, `sdk-56`, …). The `.ipa` lands as a workflow
artifact.

Expect **~1.5–3 h**: the Expo Go Podfile forces `RCT_USE_PREBUILT_RNCORE=0`, so
React Native and Hermes are compiled from source out of the `react-native-lab`
submodule.

## Notes

- Requires **iOS 16.4+** on the device (`IPHONEOS_DEPLOYMENT_TARGET` in the
  Expo Go project).
- Signing is disabled via `CODE_SIGNING_ALLOWED=NO`, and the workflow strips
  `CODE_SIGN_ENTITLEMENTS` / `DEVELOPMENT_TEAM` / provisioning profile
  specifiers from `project.pbxproj` first. Expo Go's entitlements ask for Push,
  Sign in with Apple, Associated Domains and iCloud — none available to a free
  Apple ID. Those features won't work; opening projects over LAN/tunnel does.
- Push notifications inside Expo Go will not work, for the same reason.
