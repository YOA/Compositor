# Compositor 1.2.11 — macOS 15 互換版 導入ガイド

これは
[robbietilton/Compositor](https://github.com/robbietilton/Compositor)
を **macOS 15でビルド・実行するための非公式互換ブランチ**です。

公式のCompositorは **macOS 26.5以降**を対象としています。
このブランチではCompositor 1.2.11の画像処理やドキュメント処理には手を加えず、
macOS 15でビルドするために必要な最小限の互換処理だけを追加しています。

> [!IMPORTANT]
> この手順書は、Xcodeやソースコードからのアプリビルドに慣れていない方でも
> 順番に進められるように書いています。
> すでに古いXcodeを使っている場合でも、削除や置き換えは不要です。

## 検証対象

この互換ブランチは以下を基準にしています。

- Compositor 1.2.11
- Base commit: `c64183f464b0e234f8b9b7c42695c1fbea2ab553`
- Deployment Target: macOS 15.0
- Xcode 26.1.1
- Releaseビルド: 成功確認済み

各自のMacでの動作確認については、後述の **「Compositorを起動する」** まで実行して確認してください。

## 変更内容

変更しているのはmacOS互換性に関係する部分だけです。

- Deployment Target: macOS 26.5 → macOS 15.0
- `ToolbarSpacer` はmacOS 26以降だけで使用
- `sharedBackgroundVisibility(.hidden)` はmacOS 26以降だけで使用
- `NSPopUpButton.borderShape` はmacOS 26以降だけで使用

以下には変更を加えていません。

- 画像処理
- PSD / PSB読み込み
- Camera Raw
- レイヤー / マスク
- ブラシ
- フィルター
- プロジェクト / ドキュメント形式

---

# 初心者向け インストール手順

## 0. 最初に確認すること

この手順でビルドするには以下が必要です。

- **macOS 15.6以降**のMac
- Xcode 26.1.1
- インターネット接続
- Xcodeとビルドファイル用に、20GB以上の空き容量を推奨

完成したCompositorアプリ自体のDeployment Targetは
**macOS 15.0以降**です。

ここでmacOS 15.6以降を要求しているのは、
Xcode 26.1.1を動かしてビルドするためです。

この手順ではApple Silicon版のXcode 26.1.1を使用します。
Intel Macの場合はApple Silicon専用のXcodeアーカイブを使用しないでください。

---

## 1. macOSのバージョンを確認する

**ターミナル**を開きます。

Finderから開く場合：

`アプリケーション → ユーティリティ → ターミナル`

次を実行します。

```bash
sw_vers -productVersion
```

この手順でビルドする場合、**15.6以降**であることを確認してください。

例：

```text
15.7.9
```

MacがApple Siliconかどうかも確認できます。

```bash
uname -m
```

Apple Silicon Macなら次のように表示されます。

```text
arm64
```

---

## 2. 今のXcodeを残したままXcode 26.1.1を追加する

Xcode 16.4など、すでに古いXcodeを使っている場合でも
**削除する必要はありません**。

Apple公式のDeveloper Downloadsから、

```text
Xcode_26.1.1_Apple_silicon.xip
```

をダウンロードします。

### Xcodeを展開する

Finderで `.xip` ファイルをダブルクリックしても構いません。

ターミナルから展開する場合：

```bash
cd ~/Downloads
xip -x Xcode_26.1.1_Apple_silicon.xip
```

展開するとDownloadsフォルダに新しい `Xcode.app` ができます。

**今展開した方のXcodeだけ**名前を変えます。

```bash
cd ~/Downloads
mv Xcode.app Xcode-26.1.1.app
```

Applicationsへ移動します。

```bash
sudo mv Xcode-26.1.1.app /Applications/
```

パスワードを求められたら、Macへのログインパスワードを入力します。

これで例えば次の2つを共存できます。

```text
/Applications/Xcode.app
/Applications/Xcode-26.1.1.app
```

例：

- `/Applications/Xcode.app` → 普段使っているXcode 16.4
- `/Applications/Xcode-26.1.1.app` → Compositor用のXcode 26.1.1

Xcode 26.1.1を追加しても、
過去のアプリを新しいUIへ強制移行する必要はありません。

---

## 3. Xcode 26.1.1の初回セットアップ

まず、このターミナルだけXcode 26.1.1を使うよう指定します。

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

ライセンス未同意のメッセージが出る場合：

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer \
xcodebuild -license accept
```

続けて初回セットアップを完了します。

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer \
xcodebuild -runFirstLaunch
```

その後、現在のターミナルで再度Xcode 26.1.1を指定します。

```bash
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
```

> [!NOTE]
> `DEVELOPER_DIR` の指定は現在のターミナルセッションだけに有効です。
> ターミナルを閉じれば、普段のXcode環境には影響しません。

---

## 4. macOS 15互換版Compositorをダウンロードする

ホームフォルダにソースコードを置く例です。

```bash
cd ~
```

macOS 15互換ブランチを直接cloneします。

```bash
git clone \
  --branch local/macos15-1.2.11 \
  --single-branch \
  https://github.com/YOA/Compositor.git \
  Compositor-macOS15
```

フォルダへ移動します。

```bash
cd ~/Compositor-macOS15
```

ブランチを確認します。

```bash
git branch --show-current
```

次の表示ならOKです。

```text
local/macos15-1.2.11
```

---

## 5. このターミナルでXcode 26.1.1を使う

もう一度指定しておくと確実です。

```bash
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
```

確認：

```bash
xcodebuild -version
```

次の表示ならOKです。

```text
Xcode 26.1.1
```

---

## 6. 依存パッケージを取得する

CompositorはSparkleパッケージを使用しています。

次を実行します。

```bash
xcodebuild \
  -project Compositor.xcodeproj \
  -scheme Compositor \
  -resolvePackageDependencies
```

初回はパッケージのダウンロードが行われるため、
少し時間がかかる場合があります。

---

## 7. Release版をビルドする

以下をまとめて実行します。

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

数分かかる場合があります。

### 成功した場合

最後の方に、

```text
** BUILD SUCCEEDED **
```

と表示されれば成功です。

完成したアプリは次の場所にあります。

```text
.build-macos15/Build/Products/Release/Compositor.app
```

---

## 8. ローカル利用用に署名する

上のビルドでは通常のDeveloper ID署名を無効にしています。

自分のMacで実行するため、ad-hoc署名を行います。

```bash
codesign --force --deep --sign - \
.build-macos15/Build/Products/Release/Compositor.app
```

次のような表示が出ることがあります。

```text
Compositor.app: replacing existing signature
```

これは正常です。

署名を確認します。

```bash
codesign --verify --deep --strict \
.build-macos15/Build/Products/Release/Compositor.app
```

**何も表示されなければ成功**です。

---

## 9. macOS 15向けバイナリになっているか確認する

次を実行します。

```bash
otool -l \
.build-macos15/Build/Products/Release/Compositor.app/Contents/MacOS/Compositor \
| grep -A4 LC_BUILD_VERSION
```

次の表示を探します。

```text
minos 15.0
```

`minos 15.0` になっていれば、
macOS 15を最低Deployment Targetとしてビルドされています。

---

## 10. Compositorを起動する

次を実行します。

```bash
open -n \
.build-macos15/Build/Products/Release/Compositor.app
```

Compositorの画面が開けば、まず起動成功です。

最低限、次の動作を確認することを推奨します。

1. 新規キャンバスを作成
2. PNGまたはJPEGを読み込む
3. ブラシで1ストローク描く
4. レイヤーを追加・複製・並べ替え
5. Blend Modeを変更
6. Type Toolを使い、フォント選択を開く
7. プロジェクトを保存
8. 一度閉じて再度開く

ここまで問題なければ、
主要な互換性関連の経路は動作していると確認できます。

---

## 11. Applicationsフォルダへインストールする

Compositorが正常に起動することを確認してから行ってください。

すでに `/Applications/Compositor.app` が存在する場合は、
先にFinderからゴミ箱へ移動してください。

その後：

```bash
sudo ditto \
.build-macos15/Build/Products/Release/Compositor.app \
/Applications/Compositor.app
```

これで通常のMacアプリと同様に、

- Finder → アプリケーション
- Spotlight
- Launchpad

から起動できます。

ターミナルから起動する場合：

```bash
open /Applications/Compositor.app
```

---

# 再ビルドについて

このブランチはCompositor 1.2.11を基準にしています。

将来の公式Compositorの新バージョンに、
このパッチがそのまま使えるとは限りません。

同じブランチを再ビルドする場合：

```bash
cd ~/Compositor-macOS15
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
rm -rf .build-macos15
```

その後、Releaseビルドを再度実行してください。

この互換forkには独自の署名済み・notarized済み自動アップデート配布経路はありません。

---

# Forkを使わず、パッチだけ適用する場合

公式upstreamから開始する場合：

```bash
git clone https://github.com/robbietilton/Compositor.git
cd Compositor
git checkout c64183f464b0e234f8b9b7c42695c1fbea2ab553
```

互換パッチをダウンロードします。

```bash
curl -L \
  https://raw.githubusercontent.com/YOA/Compositor/local/macos15-1.2.11/patches/compositor-1.2.11-macos15.patch \
  -o compositor-1.2.11-macos15.patch
```

適用します。

```bash
git apply compositor-1.2.11-macos15.patch
```

その後、この手順書のビルド手順を実行してください。

---

# トラブルシューティング

## `You have not agreed to the Xcode and Apple SDKs license`

次を実行します。

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer \
xcodebuild -license accept
```

続けて：

```bash
sudo DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer \
xcodebuild -runFirstLaunch
```

---

## `xcodebuild -version` がXcode 16.xのまま

次を実行してください。

```bash
export DEVELOPER_DIR=/Applications/Xcode-26.1.1.app/Contents/Developer
xcodebuild -version
```

システム全体のXcodeを変更する必要がない場合は、
`sudo xcode-select` を使う必要はありません。

---

## `Xcode-26.1.1.app` が見つからない

ApplicationsにあるXcodeを確認します。

```bash
ls -d /Applications/Xcode*
```

次のパスが存在するか確認してください。

```text
/Applications/Xcode-26.1.1.app
```

別の名前にした場合は、その名前に合わせて
`DEVELOPER_DIR` を変更してください。

---

## Gitで「フォルダがすでに存在する」と表示される

すでにclone済みなら、もう一度cloneする必要はありません。

```bash
cd ~/Compositor-macOS15
```

ブランチ確認：

```bash
git branch --show-current
```

必要であれば：

```bash
git switch local/macos15-1.2.11
```

---

## Sparkleやパッケージ取得で失敗する

インターネット接続を確認し、もう一度：

```bash
xcodebuild \
  -project Compositor.xcodeproj \
  -scheme Compositor \
  -resolvePackageDependencies
```

その後、再度ビルドしてください。

---

## ビルドが失敗する

ビルドログを保存しながら実行できます。

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

完全なログが、

```text
build.log
```

として保存されます。

問題報告時には以下を添えてください。

- macOSのバージョン
- Macの機種 / Apple SiliconかIntelか
- `xcodebuild -version` の結果
- `build.log` 最後付近のエラー

---

## `codesign` で `replacing existing signature` と表示された

正常です。

```text
Compositor.app: replacing existing signature
```

はエラーではありません。

---

## `codesign --verify` で何も表示されない

正常です。

何も表示されなければ署名検証成功です。

---

## ビルドには成功したがアプリが起動しない

まずターミナルから起動します。

```bash
open -n \
.build-macos15/Build/Products/Release/Compositor.app
```

直近のログを確認できます。

```bash
log show --last 5m \
  --style compact \
  --predicate 'process == "Compositor"' \
| tail -200
```

問題報告時には、この出力を添えてください。

---

## `minos 15.0` になっていない

現在のブランチを確認します。

```bash
git branch --show-current
```

次である必要があります。

```text
local/macos15-1.2.11
```

その後、ローカルのビルドフォルダだけ削除します。

```bash
rm -rf .build-macos15
```

もう一度ビルドしてください。

---

# 重要事項

これは非公式のmacOS 15互換ブランチです。

公式upstreamのメンテナはmacOS 26.5以降を対象とする方針を選択しており、
現在macOS 15互換性を公式に保守する予定はありません。

macOS 15互換版固有の問題は、
公式upstreamではなくこのFork側へ報告してください。

このビルドはローカル利用向けのad-hoc署名です。
公式のDeveloper ID署名・notarization済みCompositorリリースではありません。

## License

CompositorはMIT Licenseで配布されています。

Original copyright:

Copyright (c) 2026 Wonder Assembly LLC.
