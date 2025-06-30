# CLAUDE.md

このファイルは、このリポジトリで作業する際のClaude Code (claude.ai/code) への指針を提供します。

## プロジェクト概要

このプロジェクトは、kintone JavaScript APIの動作確認とテストを目的としたブラウザ拡張機能です。kintoneページで利用可能なすべてのJavaScript APIメソッドを自動検出し、順次実行してDevToolsのConsoleに結果を表示します。

### 最新バージョンの主要機能

- **自動メソッド検出**: 60+個のkintone JavaScript APIメソッドを再帰的に自動検出
- **アプリID自動取得**: `kintone.app.getId()`を最初に実行し、以降のメソッドで活用
- **アプリアイコン取得**: アプリID取得成功時に`kintone.app.getIcons([appId])`を自動実行
- **インテリジェントUI**: 非kintoneページでは適切なガイダンスを表示、ボタンを非表示
- **堅牢なエラーハンドリング**: 403、401、404、500、タイムアウトエラーでも処理継続
- **詳細な統計情報**: 成功・エラー・スキップ件数の表示

## デバッグモード

デフォルトでは、拡張機能は重要なkintone JavaScript API実行結果のみをコンソールに出力します。
詳細なデバッグログが必要な場合は、以下の手順でデバッグモードを有効にしてください：

1. `config/debug.ts` の `DEBUG_MODE` を `true` に変更
2. `public/kintone-bridge.js` の `DEBUG_MODE` を `true` に変更
3. 拡張機能をリビルド（`npm run build` または `npm run dev`）

デバッグモードでは以下のログが追加で出力されます：
- Content script の初期化ログ
- メッセージの送受信ログ
- Bridge script の動作ログ
- kintoneオブジェクトの存在確認ログ
- 内部メソッドの実行ログ（`[内部メソッド]`として明示）

## エラーハンドリング

この拡張機能は403エラー（権限不足）などが発生しても処理を継続するように設計されています：

- **403 Forbidden**: 権限が必要なAPIでも、エラーログを出力して次のメソッドに継続
- **401 Unauthorized**: 認証エラーも同様に継続
- **404 Not Found**: リソースが見つからない場合も継続
- **500 Internal Server Error**: サーバーエラーも継続
- **Timeout**: タイムアウトエラーも継続

すべてのエラーは詳細な情報と共にログに記録され、統計情報に反映されます。

詳細なデバッグログが必要な場合は、以下の手順でデバッグモードを有効にしてください：

1. `config/debug.ts` の `DEBUG_MODE` を `true` に変更
2. `public/kintone-bridge.js` の `DEBUG_MODE` を `true` に変更
3. 拡張機能をリビルド（`npm run build` または `npm run dev`）

デバッグモードでは以下のログが追加で出力されます：

- Content script の初期化ログ
- メッセージの送受信ログ
- Bridge script の動作ログ
- kintoneオブジェクトの存在確認ログ

## 開発コマンド

### 基本的な開発コマンド

- `npm run dev` - 開発モード（Chrome用）で拡張機能を起動
- `npm run dev:firefox` - Firefox用の開発モードで起動
- `npm run build` - Chrome用の本番ビルド
- `npm run build:firefox` - Firefox用の本番ビルド
- `npm run zip` - Chrome用の配布用ZIPファイルを作成
- `npm run zip:firefox` - Firefox用の配布用ZIPファイルを作成
- `npm run compile` - TypeScriptの型チェック（コンパイルなし）

### 開発・デバッグ時の注意事項

- コード変更後は拡張機能の再読み込みが必要です
- 実行結果の確認はDevToolsのConsoleタブで行います
- 対象ドメインは`*.cybozu.com`と`*.kintone.com`のみです

## アーキテクチャとプロジェクト構造

このプロジェクトは[WXT](https://wxt.dev/)フレームワークを使用したブラウザ拡張機能です。

### 主要な機能

- **自動メソッド検出**: kintoneオブジェクトを再帰的に探索して利用可能なメソッドを検出（通常60+個）
- **アプリID自動取得**: 最初に`kintone.app.getId()`を実行してアプリIDを取得・記憶し、以降のメソッドで活用
- **アプリアイコン取得**: アプリID取得成功時に`kintone.app.getIcons([appId])`を自動実行
- **インテリジェントUI**: 非kintoneページでは適切なガイダンスメッセージを表示し、ボタンを非表示
- **堅牢なエラーハンドリング**: 403、401、404、500、タイムアウトエラーが発生しても処理を継続
- **安全な実行**: 副作用のあるメソッドや引数が必要なメソッドは自動でスキップ
- **詳細なログ出力**: 各メソッドの実行結果をDevToolsのConsoleに見やすく表示
- **統計情報**: 成功・エラー・スキップの件数を表示
- **内部メソッド制御**: デバッグモード以外では内部処理メソッドのログを非表示

### ディレクトリ構造

```text
kintone-js-api-tester/
├── entrypoints/              # 拡張機能のエントリーポイント
│   ├── background.ts         # バックグラウンドスクリプト
│   ├── content.ts           # コンテンツスクリプト（クロスコンテキスト通信）
│   └── popup/               # ポップアップUI
│       ├── index.html       # ポップアップHTML
│       ├── main.ts         # ポップアップメインロジック
│       └── style.css       # ポップアップスタイル
├── public/                  # 公開アセット
│   ├── kintone-bridge.js   # kintone JavaScript API実行ブリッジスクリプト
│   └── icon/               # 拡張機能アイコン
├── wxt.config.ts           # WXT設定ファイル
└── package.json            # npm設定
```

### 重要なファイルの説明

#### `entrypoints/popup/main.ts`

- ポップアップUIのメインロジック（動的UI生成）
- kintoneメソッドの実行統制とエラーハンドリング
- アプリID自動取得とアプリアイコン取得機能
- 実行結果のフォーマットと表示
- 非kintoneページでのガイダンス表示制御

#### `entrypoints/content.ts`

- コンテンツスクリプト（isolated world）
- ポップアップとbridge scriptの間のメッセージ中継
- browser.tabs.sendMessageとwindow.postMessageの橋渡し

#### `public/kintone-bridge.js`

- メインワールドで実行されるブリッジスクリプト（~18KB）
- kintoneオブジェクトに直接アクセスしてAPIを実行
- メソッドの動的検出とDOM要素の文字列変換
- DevToolsコンソールへの実行結果ログ出力
- 内部メソッドとkintone JavaScript APIメソッドの適切な区別表示
- HTTPエラー（403、401、404、500）の詳細情報付きエラーハンドリング

#### `wxt.config.ts`

- マニフェスト設定（permissions、web_accessible_resources）
- kintoneドメインでのみ動作するよう制限

#### `config/debug.ts`

- デバッグモードの一元管理
- ログ出力関数の定義（debugLog、debugWarn、debugError、apiLog）
- TypeScript型安全性を確保

#### `entrypoints/popup/style.css`

- ポップアップUIのスタイル定義
- 警告・エラーメッセージ用の専用スタイル
- ライトモード・ダークモード対応

### 技術的アーキテクチャ

#### クロスコンテキスト通信フロー

```text
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
                                               │ ▶ ...               │
                                               └─────────────────────┘
```

#### なぜこの複雑な構造が必要なのか

- **セキュリティ制約**: ブラウザ拡張機能のコンテンツスクリプトは分離されたワールドで実行され、ページのJavaScriptオブジェクトに直接アクセスできません
- **kintoneアクセス**: kintoneオブジェクトはWebページのメインワールドにのみ存在するため、bridge scriptの注入が必要です
- **通信方式**: window.postMessageを使用してコンテキスト間でデータを安全にやり取りします

### WXT設定とマニフェスト

- TypeScriptとViteベースのビルドシステム
- モジュールパスエイリアス: `@/` はプロジェクトルートを指す
- WXTが自動的に`manifest.json`を生成し、ビルドプロセスを管理
- `.wxt/` ディレクトリには自動生成されるファイルが含まれる（gitignore対象）

### 重要な技術的実装詳細

#### メソッド検出アルゴリズム

- kintoneオブジェクトを再帰的に探索（最大10層まで）
- 関数型のプロパティのみを検出（通常60+個）
- Promise型など問題のあるオブジェクトは除外
- 無限ループ防止のための深度制限
- 実行時に動的検出のため、新しいkintone JavaScript APIも自動対応

#### スキップ対象メソッド

以下のメソッドは引数が必要または副作用があるため自動的にスキップされます：

**副作用があるメソッド（`skipMethods`）：**
- `kintone.api()` - 引数が必要
- `kintone.events.on()` - イベントハンドラーが必要
- `kintone.oauth.*` - OAuth設定が必要
- `kintone.plugin.*` - プラグイン設定が必要
- `Promise` - コンストラクタ
- `proxy` - プロキシ設定

**引数が必要なメソッド（`problematicMethods`）：**
- `kintone.app.getFieldElements(fieldCode)` - フィールドコードが必要
- `kintone.app.getAssignedApps()` - 引数が必要
- `kintone.app.getLookupTargetAppId()` - 引数が必要
- `kintone.app.getRelatedRecordsTargetAppId()` - 引数が必要

**既に実行済みのメソッド：**
- `kintone.app.getId()` - 最初に実行済み
- `kintone.app.getIcons()` - アプリID取得時に実行済み

#### エラーハンドリングとパフォーマンス

- **DOM要素処理**: DOM要素は安全に文字列表現に変換
- **包括的例外処理**: 全段階で多層のtry-catch構造を実装
- **HTTPエラー対応**: 403、401、404、500エラーを詳細分類して適切にログ出力
- **処理継続**: エラー発生時も必ず次のメソッドに進む設計
- **メモリ管理**: イベントリスナーの適切なクリーンアップ
- **タイムアウト**: 設定可能なタイムアウト（デフォルト10秒）
- **負荷制限**: メソッド間300-500ms間隔でシステム負荷を軽減（エラー時は少し長め）
- **適応的待機**: エラー発生状況に応じて待機時間を調整

## 開発時の注意事項

### コード変更時

1. 拡張機能の再読み込みが必要です
2. TypeScriptの型チェックは`npm run compile`で実行
3. ビルドエラーは`.wxt/`ディレクトリを削除して再ビルドすることで解決する場合があります
4. デバッグモードの変更時は、TypeScriptとJavaScript両方のファイルを同期する必要があります

### デバッグとテスト

1. kintoneページ（`*.cybozu.com`または`*.kintone.com`）でのみ動作します
2. DevToolsのConsoleでAPI実行結果を確認します
   - 非デバッグモード: kintone JavaScript APIの実行結果のみ
   - デバッグモード: 内部処理も含む詳細ログ
3. 非kintoneページでは適切なガイダンスが表示されることを確認します
4. エラーハンドリングのテストには403エラーが発生しやすいAPIを使用します
5. ネットワークタブでAPI通信を監視できます

### セキュリティと品質保証

#### セキュリティ対策

- **Content Security Policy (CSP)**: スクリプトインジェクション攻撃を防止
- **Origin検証**: クロスコンテキスト通信の安全性強化
- **メッセージ検証**: 不正なデータやコマンドをブロック
- **深度制限**: 無限ループやスタックオーバーフローを防止
- **権限最小化**: 必要最小限の権限で動作

#### コード品質

- **TypeScript型安全性**: `any`型を排除し厳密な型定義
- **エラーハンドリング**: 包括的な例外処理とログ出力
- **メモリ管理**: リスナーの適切なクリーンアップ
- **パフォーマンス**: 非同期処理とタイムアウト管理

#### 本番環境での使用注意

- APIキーや認証情報はログに出力されない
- 副作用のあるAPIは自動的にスキップ
- 適切な権限管理とモニタリングが必要
- HTTPエラー（403、401、404、500）は正常な動作として扱われる
- 内部メソッドのログは本番環境では非表示

### ビルドとデプロイ

```bash
# 品質チェック
npm run compile  # TypeScript型チェック
npm run build    # Chrome版ビルド
npm run build:firefox  # Firefox版ビルド

# 配布用パッケージ作成
npm run zip      # Chrome版ZIP
npm run zip:firefox  # Firefox版ZIP
```

## コンソール出力仕様

この拡張機能は、kintone JavaScript APIの実行結果をkintoneページのDevToolsコンソールに出力します。

### ログ出力の種類

1. **メソッド実行ログ**: `📝 kintone.methodName() 実行中...`
2. **成功結果ログ**: `✅ kintone.methodName() 結果:`
3. **エラーログ**: `❌ kintone.methodName() エラー:`
4. **スキップログ**: `⏭️ methodName - スキップ（理由）`
5. **統計情報**: 実行完了時の成功・エラー・スキップ件数

### デバッグモード別出力

- **非デバッグモード** (`DEBUG_MODE = false`): kintone JavaScript APIの実行結果のみ
- **デバッグモード** (`DEBUG_MODE = true`): 内部処理を含む詳細ログ

### 内部メソッドの処理

以下の内部メソッドは、非デバッグモードではログが非表示となります：
- `logInitialSetup` - 初期設定ログ
- `logAppIdSuccess` - アプリID取得成功ログ
- `logAppIdFailure` - アプリID取得失敗ログ
- `logAppIdError` - アプリID取得エラーログ
- `logIconsStart` - アイコン取得開始ログ
- `showExecutionSummary` - 実行結果サマリー

デバッグモードでは、これらのメソッドに`[内部メソッド]`のプレフィックスが付いて表示されます。
