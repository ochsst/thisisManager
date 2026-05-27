# サッカー監督ツール

サッカー監督向けのローカル動作Webアプリ。シングルHTMLファイル（`index.html`）のみで構成。GitHub Pagesへのホスティングを想定。

## ファイル構成

```
PJ6/
└── index.html   # すべてのHTML・CSS・JavaScriptを含む唯一のファイル
```

## 技術スタック

- **シングルHTMLファイル** — 外部ライブラリ・フレームワーク一切不使用
- **データ永続化** — `localStorage`（キー名は下記参照）
- **ピッチ描画** — インラインSVG（viewBox="0 0 500 750"、FIFA比率 105m×68m）
- **対象環境** — PC・スマホ両対応（タッチ＆ドラッグ対応）

## localStorageキー

| キー | 内容 |
|------|------|
| `soccerTool_players` | 選手一覧（JSON配列） |
| `soccerTool_matches` | 試合記録（JSON配列） |
| `soccerTool_pitch`   | ピッチ配置状態（JSON） |

## データ構造

```js
// players
{ id: "uuid", name: "名前", number: 10, position: "MF" }

// matches
{
  id: "uuid", date: "2024-04-01", opponent: "相手チーム名",
  score: "3-1", review: "総評テキスト",
  players: [{ playerId: "uuid", goals: 1, assists: 0, minutes: 90 }]
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
| 4 | 選手統計 | 🚧 未実装 |
| 5 | データ管理（JSONエクスポート/インポート） | 🚧 未実装 |

## index.html の内部構成

### CSS（`<style>`）

セクション順：

1. リセット・CSS変数（`:root`）
2. レイアウト（`#app`、`header`）
3. タブナビ（`#tab-nav`、`.tab-btn`）
4. カード・ボタン・テーブル・バッジ・フォーム
5. モーダル（`.modal-overlay`、`.modal`）
6. 試合記録（`.match-card`、`.mp-row`など）
7. ピッチ配置（`#pitch-wrap`、`.p-icon`、`.bench-icon`など）
8. トースト・スマホ調整

### HTML（`<body>`）

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
| `renderPlayers()` | 選手一覧の再描画 |
| `openPlayerModal(id?)` | 選手追加/編集モーダルを開く |
| `savePlayer()` | 選手の保存（追加/更新） |
| `renderMatches()` | 試合一覧の再描画 |
| `openMatchModal(id?)` | 試合追加/編集モーダルを開く |
| `saveMatch()` | 試合の保存 |
| `viewMatch(id)` | 試合詳細モーダルを開く |
| `initPitch()` | ピッチタブ初期化（タブ切替時に呼ぶ） |
| `renderBench()` | 自チームベンチ再描画 |
| `renderEnemyBench()` | 相手チームベンチ再描画 |
| `renderPitchIcons()` | ピッチ上アイコン再描画 |
| `confirmDeletePlayer(id)` / `confirmDeleteMatch(id)` | 削除確認ダイアログ |
| `closeConfirm()` | 確認モーダルを閉じる |

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
