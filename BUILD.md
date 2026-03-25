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
