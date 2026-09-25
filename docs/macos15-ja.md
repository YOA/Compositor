# Compositor 1.3 — macOS 15 互換版

これは [robbietilton/Compositor](https://github.com/robbietilton/Compositor) 1.3を
**macOS 15でビルド・実行するための非公式互換ブランチ**です。

公式版は macOS 26.5以降を対象としています。
このForkでは、Compositor 1.3の本体機能には手を加えず、
macOS 15向けの最小限の互換修正だけを追加します。

## 通常の利用方法

このForkの `local/macos15-1.3` ブランチにはmacOS 15向け修正を含めます。

**互換パッチを手動で適用する必要はありません。**

Xcode 26.1.1でこのブランチをビルドしてください。

## 必要環境

- macOS 15.6以降
- Xcode 26.1.1
- インターネット接続

完成したアプリ自体のDeployment Targetは **macOS 15.0** です。

## Xcode 26.1.1をこのターミナルで使う

```bash
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
xcodebuild -version
```

必要に応じて初回セットアップ：

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer xcodebuild -license accept
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer xcodebuild -runFirstLaunch
```

## macOS 15互換版を取得する

```bash
git clone   --branch local/macos15-1.3   --single-branch   https://github.com/YOA/Compositor.git   Compositor-macOS15

cd Compositor-macOS15
```

## Release版をビルドする

```bash
xcodebuild   -project Compositor.xcodeproj   -scheme Compositor   -configuration Release   -destination 'platform=macOS'   -derivedDataPath .build-macos15   CODE_SIGNING_ALLOWED=NO   build
```

`** BUILD SUCCEEDED **` と表示されれば成功です。

## ローカル利用用に署名する

```bash
codesign --force --deep --sign - .build-macos15/Build/Products/Release/Compositor.app
```

## 起動する

```bash
open -n .build-macos15/Build/Products/Release/Compositor.app
```

## Applicationsへ入れる

```bash
sudo ditto .build-macos15/Build/Products/Release/Compositor.app /Applications/Compositor.app
```

# パッチだけ適用する場合

公式upstreamのCompositor 1.3から開始する場合：

```bash
git clone https://github.com/robbietilton/Compositor.git
cd Compositor
git checkout a299f4cf09ed150b3900487467fa1371d3f386bb
```

互換パッチを適用します。

```bash
git apply compositor-1.3-macos15.patch
```

その後、上記の手順でビルドしてください。

# このForkについて

このForkは、Compositor 1.3をmacOS 15でとりあえず試してみたい人向けの
**非公式な互換版**です。

継続的なメンテナンスや、公式Compositorの将来バージョンへの追従は約束していません。

自分で使用する中で修正や改善を行った場合は、
気まぐれにこのForkへ反映・更新することがあります。

macOS 15固有の問題や、このFork固有の不具合については、
基本的に各自での調査・修正をお願いします。
必要に応じてForkして自由に変更してください。

このFork固有の問題を公式upstreamへ問い合わせるのは避けてください。

このFork独自の署名済み・notarization済みリリースや自動アップデート配布はありません。
この手順で作成するビルドはローカル利用向けのad-hoc署名です。

## Base

- Compositor 1.3
- Base commit: `a299f4cf09ed150b3900487467fa1371d3f386bb`
- Deployment Target: macOS 15.0

## License

CompositorはMIT Licenseで配布されています。

Original copyright:

Copyright (c) 2026 Wonder Assembly LLC.
