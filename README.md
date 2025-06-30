# kintone JavaScript API テスター

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/r3-yamauchi/kintone-js-api-tester)

[kintone JavaScript API](https://cybozu.dev/ja/kintone/docs/js-api/) を自動検出して順次実行し、
DevTools の Console に結果を表示するブラウザ拡張機能です。

## 🎯 概要

この拡張機能は [kintone JavaScript API](https://cybozu.dev/ja/kintone/docs/js-api/) を自動的に検出・実行して、
各APIの動作確認や戻り値の調査を簡単に行えるツールです。

### 主な機能

- **自動メソッド検出**: kintoneオブジェクトを再帰的に探索し、利用可能なすべてのメソッドを自動検出（通常60+個）
- **アプリID自動取得**: 最初に`kintone.app.getId()`を実行してアプリIDを取得・記憶し、以降のメソッドで活用
- **アプリアイコン取得**: アプリID取得成功時に`kintone.app.getIcons([appId])`を自動実行
- **インテリジェントなUI**: kintone画面以外では適切なガイダンスを表示し、ボタンを非表示
- **堅牢なエラーハンドリング**: 403、401、404、500エラーが発生しても処理を継続し、詳細なエラー情報を出力
- **詳細なログ出力**: 各メソッドの実行結果をDevToolsのConsoleに見やすく表示
- **統計情報**: 成功・エラー・スキップの件数を表示
- **デバッグモード**: 開発時の詳細なログ出力を制御

## 🚀 使用方法

### 1. インストール

1. [https://github.com/r3-yamauchi/kintone-js-api-tester/releases](https://github.com/r3-yamauchi/kintone-js-api-tester/releases) から [kintone-js-api-tester-1.0.1-chrome.zip](https://github.com/r3-yamauchi/kintone-js-api-tester/releases/download/1.0.1/kintone-js-api-tester-1.0.1-chrome.zip) をダウンロード
2. ダウンロードしたzipファイルを展開
3. Chrome拡張機能管理画面（`chrome://extensions/`）で「デベロッパーモード」を有効化
4. 「パッケージ化されていない拡張機能を読み込む」で zipファイルを展開したフォルダを選択

### 2. 基本的な使い方

1. kintoneアプリページを開く（例：`https://your-domain.cybozu.com/k/123/`）
2. 拡張機能アイコンをクリック
3. 「Run All kintone Methods」ボタンをクリック
4. DevToolsのConsoleタブで実行結果を確認

### 3. 実行例

```
====================================
🚀 kintone JavaScript API 実行開始
====================================

📋 利用可能なメソッド数: 63

🔧 初期設定: アプリID取得中...
✅ アプリID取得成功: 768

🔧 実行: kintone.app.getIcons() でアプリアイコン取得中...
✅ kintone.app.getIcons([768]) 結果:
   [
     {
       "app": "768",
       "url": "https://example.cybozu.com/k/api/app/icon?app=768&key=APP39"
     }
   ]

📝 kintone.app.get() 実行中...
✅ kintone.app.get() 結果:
   {
     "id": "768",
     "name": "プロセス管理",
     "description": "",
     "code": null,
     "numberPrecision": {
       "digits": 16,
       "places": 4,
       "roundingMode": "HALF_EVEN"
     },
     "enableComments": true
   }

====================================
🎉 kintone JavaScript API 実行完了
✅ 成功: 44件
❌ エラー: 0件
⏭️ スキップ: 19件
📊 合計: 63件
====================================
```

## 🏗️ アーキテクチャ

### 技術スタック

- **フレームワーク**: [WXT](https://wxt.dev/) - Web Extension Toolkit
- **言語**: TypeScript
- **ビルドツール**: Vite（WXTに統合）
- **対象ブラウザ**: Chrome (Manifest V3), Firefox
- **パッケージサイズ**: ~46KB

### プロジェクト構造

```
kintone-js-api-tester/
├── entrypoints/              # 拡張機能のエントリーポイント
│   ├── background.ts         # バックグラウンドスクリプト
│   ├── content.ts           # コンテンツスクリプト（クロスコンテキスト通信）
│   └── popup/               # ポップアップUI
│       ├── index.html       # ポップアップHTML
│       ├── main.ts         # ポップアップメインロジック
│       └── style.css       # ポップアップスタイル（警告・エラー対応）
├── public/                  # 公開アセット
│   ├── kintone-bridge.js   # kintone JavaScript API実行ブリッジスクリプト（~18KB）
│   └── icon/               # 拡張機能アイコン
├── config/                  # 設定ファイル
│   └── debug.ts            # デバッグモード設定
├── wxt.config.ts           # WXT設定ファイル
├── package.json            # npm設定
├── CLAUDE.md              # 開発者向け詳細仕様
└── README.md              # このファイル
```

### 通信フロー

```
┌─────────────┐    browser.tabs.sendMessage    ┌─────────────────────┐
│  Popup UI   │ ──────────────────────────────▶ │  Content Script     │
│ (isolated)  │                                │    (isolated)       │
└─────────────┘                                └─────────────────────┘
                                                         │
                                                window.postMessage
                                                         ▼
                                               ┌─────────────────────┐
                                               │  Bridge Script      │
                                               │   (main world)      │
                                               │                     │
                                               │ ▶ kintone.getDomain()│
                                               │ ▶ kintone.app.get() │
                                               │ ▶ kintone.getIcons() │
                                               │ ▶ ...（60+メソッド） │
                                               └─────────────────────┘
```

### 実行フロー

1. **UI初期化**: ページドメインをチェックし、適切なUIを表示
2. **メソッド検出**: kintoneオブジェクトを再帰探索（最大10層）
3. **アプリID取得**: `kintone.app.getId()`を最初に実行
4. **アプリアイコン取得**: アプリID取得成功時に`kintone.app.getIcons([appId])`を実行
5. **順次実行**: 各メソッドを300-500ms間隔で実行（負荷軽減）
6. **エラーハンドリング**: エラー発生時も処理を継続
7. **統計表示**: 実行完了時に詳細な統計情報を表示

### セキュリティ機能

- **Content Security Policy**: スクリプトインジェクション攻撃を防止
- **Origin検証**: クロスコンテキスト通信の安全性強化
- **メッセージ検証**: 不正なデータやコマンドをブロック
- **深度制限**: 無限ループやスタックオーバーフローを防止
- **権限最小化**: 必要最小限の権限で動作

## 🛠️ 開発

### 開発コマンド

```bash
# 依存関係のインストール
npm install

# 開発モード（Chrome用）
npm run dev

# 開発モード（Firefox用）
npm run dev:firefox

# 本番ビルド
npm run build

# Firefox版ビルド
npm run build:firefox

# 配布用ZIPファイル作成
npm run zip

# Firefox版ZIPファイル作成
npm run zip:firefox

# 型チェック
npm run compile
```

### デバッグモード

詳細なデバッグログが必要な場合：

1. `config/debug.ts` の `DEBUG_MODE` を `true` に変更
2. `public/kintone-bridge.js` の `DEBUG_MODE` を `true` に変更
3. 拡張機能をリビルド（`npm run build`）

デバッグモードでは以下のログが追加で出力されます：
- Content script の初期化ログ
- メッセージの送受信ログ
- Bridge script の動作ログ
- 内部メソッドの実行ログ（`[内部メソッド]`として明示）

### 開発時の注意点

1. **コード変更時**: 拡張機能を再読み込みしてください
2. **デバッグ**: DevToolsのConsoleでログを確認できます
3. **対象ドメイン**: `*.cybozu.com` と `*.kintone.com` でのみ動作します
4. **TypeScript**: 厳密な型定義でコード品質を保証

### エラーハンドリング

この拡張機能は以下のエラーを適切に処理し、処理を継続します：

- **403 Forbidden**: 権限不足 - ログ出力して継続
- **401 Unauthorized**: 認証エラー - ログ出力して継続
- **404 Not Found**: リソースが見つからない - ログ出力して継続
- **500 Internal Server Error**: サーバーエラー - ログ出力して継続
- **Timeout**: タイムアウト（10秒） - ログ出力して継続

すべてのエラーは詳細な情報と共にログに記録され、統計情報に反映されます。

## 📚 関連リンク

- [kintone JavaScript API リファレンス](https://cybozu.dev/ja/kintone/docs/js-api/)
- [WXT Framework](https://wxt.dev/)
- [ブラウザ拡張機能開発ガイド](https://developer.chrome.com/docs/extensions/)

## 📄 ライセンス

このプロジェクトはMITライセンスの下で公開しています。詳細は [LICENSE](LICENSE) ファイルをご覧ください。
