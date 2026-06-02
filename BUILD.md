# ビルド手順 (macOS / Linux 向け)

## 前提条件

以下のバージョン管理ツールがインストールされていること:

- [pyenv](https://github.com/pyenv/pyenv)
- [nodenv](https://github.com/nodenv/nodenv)
- [rbenv](https://github.com/rbenv/rbenv)

## 1. ランタイムのインストール

プロジェクトルートに `.python-version`, `.node-version`, `.ruby-version` が配置済みです。

```bash
pyenv install 2.7.18
nodenv install 6.17.1
rbenv install 2.3.8
```

## 2. Ruby gems のインストール

Compass のインストールに必要ですが、古い gem の依存関係でバージョン制約があるため、以下の順でインストールしてください。

```bash
gem install ffi -v 1.15.5
gem install rb-inotify -v 0.10.1
gem install multi_json -v 1.15.0
gem install compass
```

## 3. npm グローバルツールのインストール

```bash
npm install -g grunt-cli@1.2.0
npm install -g electron-packager@7.7.0
nodenv rehash
```

> **注意**: grunt-cli@1.5.0 以降、electron-packager@8.x 以降は Node 6 非対応の構文 (spread operator, async/await) を使用しているため、上記のバージョンに固定してください。

## 4. package.json の差し替え

`usb_dev` モジュールは Windows 専用 (`Windows.h` が必要) のため、macOS / Linux では `package_darwin.json` を使用します。

```bash
cp package.json package.json.bak
cp package_darwin.json package.json
```

## 5. npm install

```bash
npm install
```

## 6. grunt build

```bash
grunt build --platform=darwin --force
```

TypeScript の型警告が 29 個出ますが、すべて `non-emit-preventing` (JS 出力はブロックされない) ため、`--force` で続行して問題ありません。

## 7. www ディレクトリへのファイルコピー

```bash
cp main.js www/
cp -r node_modules www/
```

## 8. パッケージ化

### Apple Silicon (arm64) ネイティブビルド ※推奨

Electron 1.4.10 は Intel (x64) 専用で、Apple Silicon Mac では Rosetta 2 経由でしか動きません
(将来の macOS で Rosetta が廃止されると動作しなくなります)。
ネイティブ arm64 で動かすため、Electron をネイティブ arm64 ビルドが存在する **13.6.9** に上げます。

> Electron 11〜13 は旧来の `remote` モジュールを内蔵しているため、`main.js` に
> `webPreferences` (`nodeIntegration` / `contextIsolation:false` / `enableRemoteModule`) を
> 追加するだけで、レンダラー側の既存コード (`require("electron").remote`, `require("fs-extra")` 等) は
> 無修正で動作します。`main.js` の `app.makeSingleInstance` は Electron 4 で削除されたため
> `app.requestSingleInstanceLock` に置き換え済みです。

パッケージ化には Node 16 以上が必要です (grunt ビルドの Node 6 とは別に用意します)。

```bash
# パッケージ化専用にモダンな Node を用意 (TS ビルドの Node 6 とは別)
nodenv install 22.22.3

cd www
NODENV_VERSION=22.22.3 npx @electron/packager@18.3.6 . HuisUICreator \
  --platform=darwin --arch=arm64 --electron-version=13.6.9 \
  --overwrite \
  --ignore="node_modules/(grunt.*|electron-rebuild)" \
  --ignore="\.git" \
  --ignore="Service References" \
  --ignore="docs" \
  --ignore="obj" \
  --ignore="tests/.*" \
  --ignore="platforms" \
  --ignore="-x64\$" \
  --ignore="-ia32\$" \
  --ignore="-arm64\$"
```

成功すると `www/HuisUICreator-darwin-arm64/HuisUICreator.app` が生成されます。
`file HuisUICreator.app/Contents/MacOS/HuisUICreator` で `arm64` と表示されれば
ネイティブビルド成功です。

### Intel (x64) ビルド ※旧来 / Rosetta 動作

```bash
cd www
electron-packager . HuisUICreator \
  --platform=darwin --arch=x64 --version=1.4.10 \
  --ignore="node_modules/(grunt*|electron-rebuild)" \
  --ignore=".git" \
  --ignore="Service References" \
  --ignore="docs" \
  --ignore="obj" \
  --ignore="tests/*" \
  --ignore="www" \
  --ignore="platforms" \
  --ignore="-x64\$" \
  --ignore="-ia32\$"
```

成功すると `www/HuisUICreator-darwin-x64/HuisUICreator.app` が生成されます。
