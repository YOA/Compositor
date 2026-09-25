# Compositor 1.3 — macOS 15 Compatibility

This is an **unofficial macOS 15 compatibility branch** of
[robbietilton/Compositor](https://github.com/robbietilton/Compositor) 1.3.

Upstream targets macOS 26.5 or later.
This fork keeps Compositor 1.3's application functionality unchanged
and adds only the minimum compatibility changes needed for macOS 15.

## Recommended use

The `local/macos15-1.3` branch includes the macOS 15 compatibility changes.

**You do not need to apply the patch manually.**

Build this branch with Xcode 26.1.1.

## Requirements

- macOS 15.6 or later
- Xcode 26.1.1
- Internet connection

The built application itself targets **macOS 15.0**.

## Use Xcode 26.1.1 in this Terminal session

```bash
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
xcodebuild -version
```

If needed, complete first-launch setup:

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer xcodebuild -license accept
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer xcodebuild -runFirstLaunch
```

## Clone the macOS 15 branch

```bash
git clone   --branch local/macos15-1.3   --single-branch   https://github.com/YOA/Compositor.git   Compositor-macOS15

cd Compositor-macOS15
```

## Build the Release version

```bash
xcodebuild   -project Compositor.xcodeproj   -scheme Compositor   -configuration Release   -destination 'platform=macOS'   -derivedDataPath .build-macos15   CODE_SIGNING_ALLOWED=NO   build
```

If the final output contains `** BUILD SUCCEEDED **`, the build succeeded.

## Ad-hoc sign the local build

```bash
codesign --force --deep --sign - .build-macos15/Build/Products/Release/Compositor.app
```

## Run

```bash
open -n .build-macos15/Build/Products/Release/Compositor.app
```

## Install into Applications

```bash
sudo ditto .build-macos15/Build/Products/Release/Compositor.app /Applications/Compositor.app
```

# Apply only the patch

To start from upstream Compositor 1.3:

```bash
git clone https://github.com/robbietilton/Compositor.git
cd Compositor
git checkout a299f4cf09ed150b3900487467fa1371d3f386bb
```

Apply the compatibility patch:

```bash
git apply compositor-1.3-macos15.patch
```

Then build using the instructions above.

# About this fork

This is an **unofficial compatibility build** for people who simply want to try
Compositor 1.3 on macOS 15.

There is no commitment to ongoing maintenance or to tracking future upstream releases.

I may occasionally update this fork when I make fixes or improvements for my own use.

For macOS 15-specific or fork-specific issues, please investigate and resolve them yourself,
or fork the repository and modify it as needed.

Please do not report issues specific to this fork to the upstream project.

This fork does not provide its own signed/notarized release or automatic update channel.
Builds created with this guide use an ad-hoc signature for local use.

## Base

- Compositor 1.3
- Base commit: `a299f4cf09ed150b3900487467fa1371d3f386bb`
- Deployment target: macOS 15.0

## License

Compositor is distributed under the MIT License.

Original copyright:

Copyright (c) 2026 Wonder Assembly LLC.
