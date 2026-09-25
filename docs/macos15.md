# Compositor 1.2.11 — macOS 15 Compatibility

This is an unofficial compatibility patch for
[robbietilton/Compositor](https://github.com/robbietilton/Compositor).

Upstream Compositor officially targets macOS 26.5 or later.
This patch allows Compositor 1.2.11 to build and run on macOS 15
without modifying the image-processing or document code.

## Tested environment

- Compositor 1.2.11
- Base commit:
  `c64183f464b0e234f8b9b7c42695c1fbea2ab553`
- macOS 15
- Xcode 26.1.1
- Release build: successful
- Application launch: successful

## Changes

Only macOS compatibility code is changed:

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

## Build from this branch

Clone this fork:

```bash
git clone https://github.com/YOA/Compositor.git
cd Compositor
git checkout local/macos15-1.2.11
