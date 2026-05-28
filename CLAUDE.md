# THIS IS 監督

サッカー監督向けのローカル動作Webアプリ。シングルHTMLファイル（`index.html`）のみで構成。GitHub Pagesへのホスティングを想定。

## ファイル構成

```
PJ6/
└── index.html   # すべてのHTML・CSS・JavaScriptを含む唯一のファイル
```

## 技術スタック

- **シングルHTMLファイル** — 外部ライブラリ・フレームワーク一切不使用
- **データ永続化** — `localStorage`（キー名は下記参照）
- **認証** — クライアントサイドログイン画面（`sessionStorage` でセッション管理）
- **ピッチ描画** — インラインSVG（viewBox="0 0 500 750"、FIFA比率 105m×68m）
- **対象環境** — PC・スマホ両対応（タッチ＆ドラッグ対応）

## ログイン認証

- **ID**: `thisis` / **パスワード**: `football`
- `sessionStorage` キー: `thisis_manager_auth`（値 `'1'` でログイン済み）
- ページ読み込み時に `sessionStorage` を確認し、未認証なら `#login-screen` を表示
- GitHub Pages はサーバーサイド認証不可のため、クライアントサイド実装

## localStorageキー

| キー | 内容 |
|------|------|
| `soccerTool_players` | 選手一覧（JSON配列） |
| `soccerTool_matches` | 試合記録（JSON配列） |
| `soccerTool_pitch`   | ピッチ配置状態（JSON） |

## データ構造

```js
// players
{ id: "uuid", name: "名前", number: 10, position: "MF", note: "備考テキスト" }

// matches
{
  id: "uuid", date: "2024-04-01", opponent: "相手チーム名",
  score: "3-1", review: "総評テキスト",
  players: [{ playerId: "uuid", start: true, goals: 1, assists: 0, minutes: 90 }]
}

// pitch（localStorage: soccerTool_pitch）
{
  allyColor: "#2563eb", enemyColor: "#dc2626",
  icons: [
    { id:"uuid", team:"ally",  playerId:"uuid", loc:"pitch"|"bench", px:0.5, py:0.5 },
    { id:"uuid", team:"enemy", num:1,            loc:"pitch"|"bench", px:0.5, py:0.5 }
  ]
}
```

## 実装フェーズ進捗

| Phase | 機能 | 状態 |
|-------|------|------|
| 1 | 骨格・選手管理 | ✅ 完了 |
| 2 | ピッチ配置 | ✅ 完了 |
| 3 | 試合記録 | ✅ 完了 |
| 4 | 選手統計 | ✅ 完了 |
| 5 | データ管理（JSONエクスポート/インポート） | ✅ 完了 |

## index.html の内部構成

### CSS（`<style>`）

セクション順：

1. ログイン画面（`#login-screen`、`.login-box`）
2. リセット・CSS変数（`:root`）
3. レイアウト（`#app`、`header`）
4. タブナビ（`#tab-nav`、`.tab-btn`）
5. カード・ボタン・テーブル・バッジ・フォーム
6. モーダル（`.modal-overlay`、`.modal`）
7. 先発/途中トグル（`.mp-start-toggle`）
8. 背番号ステッパー（`.number-stepper`、`.stepper-btn`）
9. データ管理・選手統計
10. 試合記録（`.match-card`、`.mp-row`など）
11. ピッチ配置（`#pitch-wrap`、`.p-icon`、`.p-icon.rect`、`.bench-icon`など）
12. ピッチアイコン ツールチップ・ドラッグゴースト
13. トースト・スマホ調整（`@media (max-width: 600px)`）

### HTML（`<body>`）

- `#login-screen` — ログイン画面オーバーレイ（`#app` の前に配置）
- `header` — ロゴ・タイトル
- `#tab-nav` — 5タブボタン（`data-tab` 属性でパネルIDと対応）
- `#main` — 5つの`.tab-panel`セクション
- モーダル群（選手・試合・試合詳細・削除確認）
- `#toast-container`

### JavaScript（`<script>`）

主要なグローバルオブジェクト・関数：

| 名前 | 役割 |
|------|------|
| `DB` | localStorageのCRUDラッパー |
| `PITCH` | ピッチ状態のCRUDラッパー |
| `uuid()` | UUID生成 |
| `showToast(msg, type)` | トースト通知 |
| `escHtml(s)` | XSS対策エスケープ |
| `positionBadge(pos)` | ポジションバッジHTML生成 |
| `doLogin(e)` | ログイン処理（sessionStorage書き込み） |
| `stepNumber(delta)` | 背番号ステッパー増減 |
| `updateStepperBtns(val)` | ステッパーボタンの有効/無効更新 |
| `renderPlayers()` | 選手一覧の再描画 |
| `openPlayerModal(id?)` | 選手追加/編集モーダルを開く |
| `savePlayer()` | 選手の保存（追加/更新） |
| `renderMatches()` | 試合一覧の再描画 |
| `openMatchModal(id?)` | 試合追加/編集モーダルを開く |
| `renderMatchPlayersList(existing)` | 試合モーダル内の選手リスト描画 |
| `toggleMatchPlayer(id, checked)` | 選手チェックボックス切替 |
| `toggleStartBtn(id)` | 先発/途中出場トグル |
| `saveMatch()` | 試合の保存 |
| `viewMatch(id)` | 試合詳細モーダルを開く |
| `renderStats()` | 選手統計タブの再描画 |
| `calcStats()` | 統計データ計算 |
| `renderDataOverview()` | データ管理タブの概要描画 |
| `exportData()` | JSONエクスポート |
| `initPitch()` | ピッチタブ初期化（タブ切替時に呼ぶ） |
| `renderBench()` | 自チームベンチ再描画 |
| `renderEnemyBench()` | 相手チームベンチ再描画 |
| `renderPitchIcons()` | ピッチ上アイコン再描画 |
| `toggleIconShape()` | ピッチアイコンの丸/長方形切替 |
| `createDragGhost(color, num, name)` | ドラッグゴースト生成 |
| `removeDragGhost()` | ドラッグゴースト削除 |
| `moveGhost(x, y)` | ドラッグゴースト移動 |
| `confirmDeletePlayer(id)` / `confirmDeleteMatch(id)` | 削除確認ダイアログ |
| `closeConfirm()` | 確認モーダルを閉じる |

### 主要なモジュールレベル変数

| 変数 | 内容 |
|------|------|
| `pitchState` | `PITCH.load()` で初期化されたピッチ状態 |
| `pitchIconShape` | `'circle'` または `'rect'`（デフォルト: `'circle'`） |
| `_dragCtx` | ドラッグ中の状態オブジェクト |

## SVGピッチ寸法メモ

viewBox `0 0 500 750`、フィールド領域 `x=30〜470, y=20〜730`。

| 要素 | 座標・寸法 |
|------|-----------|
| 外枠 | x=30, y=20, w=440, h=710 |
| ハーフウェーライン | y=375 |
| センターサークル | cx=250, cy=375, r=60 |
| 上ペナルティエリア | x=120, y=20, w=260, h=112 |
| 上ゴールエリア | x=191, y=20, w=118, h=37 |
| 上ペナルティスポット | (250, 94) |
| 上ペナルティアーク | M 204 132 A 60 60 0 0 0 296 132 |
| 下ペナルティエリア | x=120, y=618, w=260, h=112 |
| 下ゴールエリア | x=191, y=693, w=118, h=37 |
| 下ペナルティスポット | (250, 656) |
| 下ペナルティアーク | M 204 618 A 60 60 0 0 1 296 618 |
| 上ゴール | x=227, y=4, w=46, h=16 |
| 下ゴール | x=227, y=730, w=46, h=16 |

## コーディングルール

- **コメントは書かない**（理由が非自明な場合のみ1行で）
- **外部ライブラリ禁止** — CDN含め一切不使用
- **XSS対策** — ユーザー入力を表示する際は必ず `escHtml()` を使う
- **モーダルの開閉** — `.open` クラスの付け外しで制御
- **タブ切り替え** — `.active` クラスの付け外し。ピッチタブは切替時に `initPitch()` を呼ぶ
- **データ更新後** — 必ず対応する `render*()` 関数を呼んで再描画する

## UI仕様

- **言語** — 日本語UI
- **用語** — 「自チーム / 相手チーム」（「味方 / 敵」は使わない）
- **カラー変数** — `--primary: #1a6b2f`（緑系）、`--danger: #d9363e`（赤）
- **タッチ対応** — スマホでのドラッグは `touchstart/touchmove/touchend` を使用
- **バリデーション** — 入力エラーは `showToast(msg, 'error')` で通知
- **スマホ対応** — `@media (max-width: 600px)` でモーダルパディング縮小・テーブル列非表示・mp-row折り返し
- **ピッチアイコン** — 丸（44×44px）と長方形（72×34px）を `toggleIconShape()` で切替可能。長方形時は名前6文字まで表示
