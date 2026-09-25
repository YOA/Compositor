# Compositor 1.2.11 — macOS 15 互換版

これは [robbietilton/Compositor](https://github.com/robbietilton/Compositor) を
**macOS 15でビルド・実行するための非公式互換ブランチ**です。

公式版は macOS 26.5以降を対象としています。
このForkでは、Compositor 1.2.11の画像処理やドキュメント処理には手を加えず、
macOS 15向けの最小限の互換修正だけを追加しています。

## どちらを使えばよいですか？

### 通常はこちら

このForkにはmacOS 15向け修正がすでに入っています。

**互換パッチを手動で適用する必要はありません。**

`local/macos15-1.2.11` ブランチをcloneして、Xcode 26.1.1でビルドしてください。

### 上級者向け

公式upstreamのCompositor 1.2.11をそのまま使いたい場合は、
Forkを使わず互換パッチだけを適用できます。

→ [パッチだけ適用する場合](#パッチだけ適用する場合)

---

# 初心者向け手順

## 1. 必要環境

ビルドには以下が必要です。

- macOS 15.6以降
- Xcode 26.1.1
- インターネット接続

完成したCompositorアプリ自体のDeployment Targetは **macOS 15.0** です。

この手順ではApple Silicon版Xcode 26.1.1を使用します。

---

## 2. Xcode 26.1.1を追加する

すでにXcode 16.4などを使っている場合でも、削除する必要はありません。

Apple公式のDeveloper Downloadsから

```text
Xcode_26.1.1_Apple_silicon.xip
```

をダウンロードして展開します。

Finderでダブルクリックしても構いません。

ターミナルで展開する場合：

```bash
cd ~/Downloads
xip -x Xcode_26.1.1_Apple_silicon.xip
```

展開された `Xcode.app` を名前変更します。

```bash
cd ~/Downloads
mv Xcode.app Xcode-26.1.1.app
```

Applicationsへ移動します。

```bash
sudo mv Xcode-26.1.1.app /Applications/
```

これで既存Xcodeと共存できます。

```text
/Applications/Xcode.app
/Applications/Xcode-26.1.1.app
```

---

## 3. Xcode 26.1.1をこのターミナルで使う

```bash
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
```

確認：

```bash
xcodebuild -version
```

次のように表示されればOKです。

```text
Xcode 26.1.1
```

ライセンス未同意と表示された場合：

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer xcodebuild -license accept
```

続けて：

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer xcodebuild -runFirstLaunch
```

`DEVELOPER_DIR` は現在のターミナルだけに有効です。
普段使っているXcodeの設定は変更されません。

---

## 4. macOS 15互換版を取得する

```bash
cd ~

git clone   --branch local/macos15-1.2.11   --single-branch   https://github.com/YOA/Compositor.git   Compositor-macOS15

cd ~/Compositor-macOS15
```

確認：

```bash
git branch --show-current
```

次の表示ならOKです。

```text
local/macos15-1.2.11
```

---

## 5. Release版をビルドする

```bash
xcodebuild   -project Compositor.xcodeproj   -scheme Compositor   -configuration Release   -destination 'platform=macOS'   -derivedDataPath .build-macos15   CODE_SIGNING_ALLOWED=NO   build
```

最後に

```text
** BUILD SUCCEEDED **
```

と表示されれば成功です。

完成したアプリ：

```text
.build-macos15/Build/Products/Release/Compositor.app
```

---

## 6. ローカル利用用に署名する

```bash
codesign --force --deep --sign - .build-macos15/Build/Products/Release/Compositor.app
```

次の表示は正常です。

```text
Compositor.app: replacing existing signature
```

---

## 7. 起動する

```bash
open -n .build-macos15/Build/Products/Release/Compositor.app
```

Compositorの画面が開けば起動成功です。

必要に応じて、新規キャンバス作成・画像読み込み・ブラシ・レイヤー操作・保存/再オープンなどを確認してください。

---

## 8. Applicationsへ入れる

正常に起動することを確認した後：

```bash
sudo ditto .build-macos15/Build/Products/Release/Compositor.app /Applications/Compositor.app
```

以後はFinderの「アプリケーション」やSpotlightから起動できます。

---

# パッチだけ適用する場合

公式upstreamのCompositor 1.2.11を使いたい場合：

```bash
git clone https://github.com/robbietilton/Compositor.git
cd Compositor
git checkout c64183f464b0e234f8b9b7c42695c1fbea2ab553
```

互換パッチをダウンロードします。

```bash
curl -L   https://raw.githubusercontent.com/YOA/Compositor/local/macos15-1.2.11/patches/compositor-1.2.11-macos15.patch   -o compositor-1.2.11-macos15.patch
```

適用：

```bash
git apply compositor-1.2.11-macos15.patch
```

その後、上記のXcode 26.1.1でビルドしてください。

---

# このForkについて

このForkは、Compositor 1.2.11をmacOS 15でとりあえず試してみたい人向けの
**非公式な互換版**です。

継続的なメンテナンスや、公式Compositorの将来バージョンへの追従は約束していません。

自分で使用する中で修正や改善を行った場合は、
気まぐれにこのForkへ反映・更新することがあります。

macOS 15固有の問題や、このFork固有の不具合については、
基本的に各自での調査・修正をお願いします。
必要に応じてForkして自由に変更してください。

このFork固有の問題を公式upstreamへ問い合わせるのは避けてください。

このFork独自の署名済み・notarization済みリリースや自動アップデート配布はありません。
ソースコードにはupstreamの更新機構が残っていますが、この互換Forkの配布経路ではありません。

この手順で作成するビルドはローカル利用向けのad-hoc署名です。

## License

CompositorはMIT Licenseで配布されています。

Original copyright:

Copyright (c) 2026 Wonder Assembly LLC.
