# Compositor 1.2.11 — macOS 15 Compatibility Guide

This is an **unofficial macOS 15 compatibility branch** of
[robbietilton/Compositor](https://github.com/robbietilton/Compositor).

Upstream Compositor officially targets **macOS 26.5 or later**.
This branch keeps Compositor 1.2.11's image-processing and document code unchanged
and only adds the minimum compatibility fallbacks needed to build for macOS 15.

> [!IMPORTANT]
> This guide is written for people who are not familiar with Xcode or building apps from source.
> Follow the steps in order. You do **not** need to replace an older Xcode installation.

## Tested configuration

This compatibility branch is based on:

- Compositor 1.2.11
- Base commit: `c64183f464b0e234f8b9b7c42695c1fbea2ab553`
- Deployment target: macOS 15.0
- Xcode 26.1.1
- Release build: successful

Before publishing a machine-specific success report, verify the app actually launches on your Mac using the **Run** step below.

## What changed

Only compatibility-related code is changed:

- Deployment target: macOS 26.5 → macOS 15.0
- `ToolbarSpacer` is used only on macOS 26+
- `sharedBackgroundVisibility(.hidden)` is used only on macOS 26+
- `NSPopUpButton.borderShape` is used only on macOS 26+

No changes are made to:

- image processing
- PSD / PSB import
- Camera Raw
- layers or masks
- brushes
- filters
- project/document format

---

# Beginner installation guide

## 0. Before you start

You need:

- a Mac running **macOS 15.6 or later** to build this version
- Xcode 26.1.1
- an internet connection
- about 20 GB or more of free disk space is recommended for Xcode and build files

The built application itself targets **macOS 15.0 or later**.
The macOS 15.6 requirement above is for running Xcode 26.1.1 during the build.

This guide uses the Apple Silicon version of Xcode 26.1.1.
If your Mac uses an Intel processor, do not use an Apple-Silicon-only Xcode archive.

---

## 1. Check your macOS version

Open **Terminal**.

You can find it in:

`Applications → Utilities → Terminal`

Run:

```bash
sw_vers -productVersion
```

For building with this guide, the result should be **15.6 or later**.

Example:

```text
15.7.9
```

You can also check your CPU architecture:

```bash
uname -m
```

For an Apple Silicon Mac, this should show:

```text
arm64
```

---

## 2. Install Xcode 26.1.1 without removing your old Xcode

If you already use an older Xcode such as Xcode 16.4, keep it installed.

Download:

```text
Xcode_26.1.1_Apple_silicon.xip
```

from Apple's official Developer Downloads site.

### Extract the Xcode archive

You can double-click the `.xip` file in Finder.

Or use Terminal:

```bash
cd ~/Downloads
xip -x Xcode_26.1.1_Apple_silicon.xip
```

After extraction, a new `Xcode.app` appears in the Downloads folder.

Rename **the newly extracted copy**:

```bash
cd ~/Downloads
mv Xcode.app Xcode-26.1.1.app
```

Move it to Applications:

```bash
sudo mv Xcode-26.1.1.app /Applications/
```

Enter your Mac login password if requested.

Your Mac can now have both:

```text
/Applications/Xcode.app
/Applications/Xcode-26.1.1.app
```

For example:

- `/Applications/Xcode.app` → your existing Xcode 16.4
- `/Applications/Xcode-26.1.1.app` → Xcode used for this Compositor build

Nothing forces your older projects to migrate to the newer Xcode.

---

## 3. Complete Xcode 26.1.1 first-launch setup

Use Xcode 26.1.1 only for the following commands:

```bash
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
```

Confirm:

```bash
xcodebuild -version
```

You should see:

```text
Xcode 26.1.1
```

If Xcode says that you have not agreed to the license, run:

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer \
xcodebuild -license accept
```

Then complete the first-launch setup:

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer \
xcodebuild -runFirstLaunch
```

Finally, set Xcode 26.1.1 again for the current Terminal window:

```bash
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
```

> [!NOTE]
> `DEVELOPER_DIR` affects only the current Terminal session.
> Closing the Terminal window restores your normal Xcode environment.

---

## 4. Download the macOS 15 branch

Choose a folder where you want to keep the source code.
The following example places it in your home folder.

```bash
cd ~
```

Clone this compatibility branch directly:

```bash
git clone \
  --branch local/macos15-1.2.11 \
  --single-branch \
  https://github.com/YOA/Compositor.git \
  Compositor-macOS15
```

Enter the folder:

```bash
cd ~/Compositor-macOS15
```

Confirm the branch:

```bash
git branch --show-current
```

It should show:

```text
local/macos15-1.2.11
```

---

## 5. Make sure Xcode 26.1.1 is active for this Terminal

Run:

```bash
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
```

Then:

```bash
xcodebuild -version
```

It should report:

```text
Xcode 26.1.1
```

---

## 6. Resolve dependencies

Compositor uses the Sparkle package.

Run:

```bash
xcodebuild \
  -project Compositor.xcodeproj \
  -scheme Compositor \
  -resolvePackageDependencies
```

The first run can take some time because Xcode may download dependencies.

---

## 7. Build the Release version

Run this entire command:

```bash
xcodebuild \
  -project Compositor.xcodeproj \
  -scheme Compositor \
  -configuration Release \
  -destination 'platform=macOS' \
  -derivedDataPath .build-macos15 \
  CODE_SIGNING_ALLOWED=NO \
  build
```

The build can take several minutes.

### Success

If the final lines contain:

```text
** BUILD SUCCEEDED **
```

the build succeeded.

The application is now located at:

```text
.build-macos15/Build/Products/Release/Compositor.app
```

---

## 8. Sign the local build

The build above intentionally disables normal Developer ID signing.

For local use, apply an ad-hoc signature:

```bash
codesign --force --deep --sign - \
.build-macos15/Build/Products/Release/Compositor.app
```

You may see:

```text
Compositor.app: replacing existing signature
```

That message is normal.

Verify the signature:

```bash
codesign --verify --deep --strict \
.build-macos15/Build/Products/Release/Compositor.app
```

If this command prints nothing, verification succeeded.

---

## 9. Verify that the binary targets macOS 15

Run:

```bash
otool -l \
.build-macos15/Build/Products/Release/Compositor.app/Contents/MacOS/Compositor \
| grep -A4 LC_BUILD_VERSION
```

Look for:

```text
minos 15.0
```

If you see `minos 15.0`, the application binary was built with macOS 15 as its minimum deployment target.

---

## 10. Run Compositor

Launch the built application:

```bash
open -n \
.build-macos15/Build/Products/Release/Compositor.app
```

Compositor should open normally.

For a basic smoke test:

1. Create a new canvas.
2. Import a PNG or JPEG.
3. Paint one brush stroke.
4. Add, duplicate, and reorder a layer.
5. Change a layer blend mode.
6. Use the Type tool and open the font picker.
7. Save the project.
8. Close it and reopen it.

If those operations work, the main compatibility-sensitive paths are functioning.

---

## 11. Install Compositor into Applications

Only do this after you have confirmed that the built app launches correctly.

If an older `Compositor.app` already exists in `/Applications`, move it to the Trash first.

Then run:

```bash
sudo ditto \
.build-macos15/Build/Products/Release/Compositor.app \
/Applications/Compositor.app
```

You can now open Compositor from:

- Finder → Applications
- Spotlight
- Launchpad

Or from Terminal:

```bash
open /Applications/Compositor.app
```

---

# Updating or rebuilding

This branch is pinned to Compositor 1.2.11.
Do not assume that a future upstream version can use this exact patch unchanged.

To rebuild the same branch:

```bash
cd ~/Compositor-macOS15
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
rm -rf .build-macos15
```

Then repeat the Release build command.

This compatibility fork does not provide its own signed/notarized update channel.

---

# Apply only the patch instead of using the fork

If you prefer to start from the original upstream source:

```bash
git clone https://github.com/robbietilton/Compositor.git
cd Compositor
git checkout c64183f464b0e234f8b9b7c42695c1fbea2ab553
```

Download the compatibility patch:

```bash
curl -L \
  https://raw.githubusercontent.com/YOA/Compositor/local/macos15-1.2.11/patches/compositor-1.2.11-macos15.patch \
  -o compositor-1.2.11-macos15.patch
```

Apply it:

```bash
git apply compositor-1.2.11-macos15.patch
```

Then follow the build instructions above.

---

# Troubleshooting

## `You have not agreed to the Xcode and Apple SDKs license`

Run:

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer \
xcodebuild -license accept
```

Then:

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer \
xcodebuild -runFirstLaunch
```

---

## `xcodebuild -version` still shows Xcode 16.x

Run:

```bash
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
xcodebuild -version
```

Do not use `sudo xcode-select` unless you intentionally want to change the system-wide default Xcode.

---

## `Xcode-26.1.1.app` cannot be found

Check the installed Xcode applications:

```bash
ls -d /Applications/Xcode*
```

Make sure this path exists:

```text
/Applications/Xcode-26.1.1.app
```

If your Xcode app has a different name, change `DEVELOPER_DIR` to match it.

---

## Git says the destination folder already exists

If you already cloned the repository, do not clone it again.

Use:

```bash
cd ~/Compositor-macOS15
```

Then verify:

```bash
git branch --show-current
```

If necessary:

```bash
git switch local/macos15-1.2.11
```

---

## Sparkle or package resolution fails

Check your internet connection and retry:

```bash
xcodebuild \
  -project Compositor.xcodeproj \
  -scheme Compositor \
  -resolvePackageDependencies
```

Then rerun the build.

---

## The build fails

Copy the last part of the build output.

A useful command is:

```bash
xcodebuild \
  -project Compositor.xcodeproj \
  -scheme Compositor \
  -configuration Release \
  -destination 'platform=macOS' \
  -derivedDataPath .build-macos15 \
  CODE_SIGNING_ALLOWED=NO \
  build 2>&1 | tee build.log
```

This also saves the complete build log as:

```text
build.log
```

When reporting a problem, include:

- macOS version
- Mac model / Apple Silicon or Intel
- `xcodebuild -version`
- the final error section from `build.log`

---

## `codesign` says `replacing existing signature`

This is expected:

```text
Compositor.app: replacing existing signature
```

It is not an error.

---

## `codesign --verify` prints nothing

That means verification succeeded.

---

## The app builds but does not open

Try launching it from Terminal:

```bash
open -n \
.build-macos15/Build/Products/Release/Compositor.app
```

Then inspect recent logs:

```bash
log show --last 5m \
  --style compact \
  --predicate 'process == "Compositor"' \
| tail -200
```

Include that output when reporting the problem.

---

## The binary does not show `minos 15.0`

First verify the branch:

```bash
git branch --show-current
```

It must be:

```text
local/macos15-1.2.11
```

Then remove only the local build folder:

```bash
rm -rf .build-macos15
```

and build again.

---

# Important

This is an unofficial compatibility branch.

The upstream maintainer has chosen to target macOS 26.5+ and does not
currently plan to maintain macOS 15 compatibility.

Please report macOS 15 compatibility issues to this fork rather than
the upstream project.

This build is ad-hoc signed for local use and is **not** an official
Developer ID signed or notarized Compositor release.

## License

Compositor is distributed under the MIT License.

Original copyright:

Copyright (c) 2026 Wonder Assembly LLC.
