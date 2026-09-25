# Compositor 1.2.11 — macOS 15 Compatibility

This is an **unofficial macOS 15 compatibility branch** of
[robbietilton/Compositor](https://github.com/robbietilton/Compositor).

Upstream Compositor targets macOS 26.5 or later.
This fork keeps Compositor 1.2.11's image-processing and document code unchanged
and only adds the minimum compatibility changes needed for macOS 15.

## Which method should I use?

### Recommended

This fork already includes the macOS 15 compatibility changes.

**You do not need to apply the patch manually.**

Clone `local/macos15-1.2.11` and build it with Xcode 26.1.1.

### Advanced users

If you prefer to use the original upstream Compositor 1.2.11 source,
you can apply only the compatibility patch.

→ [Apply only the patch](#apply-only-the-patch)

---

# Beginner installation guide

## 1. Requirements

To build:

- macOS 15.6 or later
- Xcode 26.1.1
- Internet connection

The built Compositor app itself targets **macOS 15.0**.

This guide uses the Apple Silicon build of Xcode 26.1.1.

---

## 2. Install Xcode 26.1.1 alongside your existing Xcode

You do not need to remove an older Xcode such as Xcode 16.4.

Download:

```text
Xcode_26.1.1_Apple_silicon.xip
```

from Apple's Developer Downloads and extract it.

You can double-click the `.xip` in Finder.

Or use Terminal:

```bash
cd ~/Downloads
xip -x Xcode_26.1.1_Apple_silicon.xip
```

Rename the newly extracted Xcode:

```bash
cd ~/Downloads
mv Xcode.app Xcode-26.1.1.app
```

Move it to Applications:

```bash
sudo mv Xcode-26.1.1.app /Applications/
```

You can now keep both:

```text
/Applications/Xcode.app
/Applications/Xcode-26.1.1.app
```

---

## 3. Use Xcode 26.1.1 in this Terminal session

```bash
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
```

Confirm:

```bash
xcodebuild -version
```

Expected:

```text
Xcode 26.1.1
```

If Xcode reports that the license has not been accepted:

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer xcodebuild -license accept
```

Then:

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer xcodebuild -runFirstLaunch
```

`DEVELOPER_DIR` only affects the current Terminal session.
It does not replace your normal Xcode setup.

---

## 4. Download the macOS 15 branch

```bash
cd ~

git clone   --branch local/macos15-1.2.11   --single-branch   https://github.com/YOA/Compositor.git   Compositor-macOS15

cd ~/Compositor-macOS15
```

Confirm:

```bash
git branch --show-current
```

Expected:

```text
local/macos15-1.2.11
```

---

## 5. Build the Release version

```bash
xcodebuild   -project Compositor.xcodeproj   -scheme Compositor   -configuration Release   -destination 'platform=macOS'   -derivedDataPath .build-macos15   CODE_SIGNING_ALLOWED=NO   build
```

Success:

```text
** BUILD SUCCEEDED **
```

Built app:

```text
.build-macos15/Build/Products/Release/Compositor.app
```

---

## 6. Ad-hoc sign the local build

```bash
codesign --force --deep --sign - .build-macos15/Build/Products/Release/Compositor.app
```

This message is normal:

```text
Compositor.app: replacing existing signature
```

---

## 7. Run Compositor

```bash
open -n .build-macos15/Build/Products/Release/Compositor.app
```

If the app opens, the build is working.

If needed, test creating a canvas, importing an image, painting, layer operations, and save/reopen.

---

## 8. Install into Applications

After confirming the app launches correctly:

```bash
sudo ditto .build-macos15/Build/Products/Release/Compositor.app /Applications/Compositor.app
```

You can then launch it from Applications or Spotlight.

---

# Apply only the patch

If you prefer the original upstream source:

```bash
git clone https://github.com/robbietilton/Compositor.git
cd Compositor
git checkout c64183f464b0e234f8b9b7c42695c1fbea2ab553
```

Download the patch:

```bash
curl -L   https://raw.githubusercontent.com/YOA/Compositor/local/macos15-1.2.11/patches/compositor-1.2.11-macos15.patch   -o compositor-1.2.11-macos15.patch
```

Apply it:

```bash
git apply compositor-1.2.11-macos15.patch
```

Then build with Xcode 26.1.1 as described above.

---

# About this fork

This is an **unofficial compatibility build** for people who simply want to try
Compositor 1.2.11 on macOS 15.

There is no commitment to ongoing maintenance or to tracking future upstream releases.

I may occasionally update this fork when I make fixes or improvements for my own use.

For macOS 15-specific or fork-specific issues, please investigate and resolve them yourself,
or fork the repository and modify it as needed.

Please do not report issues specific to this fork to the upstream project.

This fork does not provide its own signed/notarized release or automatic update channel.
The upstream update mechanism remains in the source, but it is not a distribution channel for this compatibility fork.

Builds created with this guide use an ad-hoc signature for local use.

## License

Compositor is distributed under the MIT License.

Original copyright:

Copyright (c) 2026 Wonder Assembly LLC.
