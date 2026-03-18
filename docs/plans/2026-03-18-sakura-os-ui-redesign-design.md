# SAKURA OS UI リデザイン 設計書

## 概要

SAKURA OS のデスクトップUIを3カラム構成に変更し、右側にSAKURA AI会話タイムラインを追加する。

## レイアウト構成

```
┌──────────┬────────────────────┬────────────────────┐
│          │  CENTER            │  RIGHT             │
│  LEFT    │  為替カード        │                    │
│  PANEL   │  ニュース 3×3      │  SAKURA AI         │
│  272px   │  （小サイズ）      │  iframe            │
│          │  カレンダー        │  （会話タイムライン │
│          │  本日の作業予定    │   + 入力UI）        │
│          │  ────────────      │                    │
│          │  💬 AIチャット入力 │                    │
└──────────┴────────────────────┴────────────────────┘
```

## 変更詳細

### 1. #info-area の分割

現在の `#info-area`（left:272px〜right:0）を2分割：

- `#center-area`：left:272px、width: calc((100vw - 272px) / 2)
- `#ai-panel`：right:0、width: calc((100vw - 272px) / 2)

### 2. ニュースカード縮小（CENTER）

| 項目 | 変更前 | 変更後 |
|------|--------|--------|
| カード高さ | 110px | 80px |
| タイトルフォント | 14px | 12px |
| タイトル行数 | 3行 | 2行 |
| グリッドtop | top:104px | top:80px |
| グリッド高さ | 3×110px + gap ≒ 360px | 3×80px + gap ≒ 260px |

### 3. カレンダー・作業予定（CENTER）

- top 位置を新しいニュースグリッド高さに合わせて上に詰める（top:464px → top:370px 程度）
- 横幅はCENTER幅に合わせて調整

### 4. AIチャット入力欄（CENTER最下部）

- 作業予定の直下に配置
- `<textarea>` + 送信ボタン
- 送信時：`#ai-iframe` に `postMessage({ type: 'send', text })` を送信

### 5. SAKURA AI iframeパネル（RIGHT）

- `<iframe id="ai-iframe" src="/system/sakura-ai/?embed=1">`
- top:32px（ニュースティッカー下）、bottom:48px（タスクバー上）
- 幅・高さ：RIGHT列を全て使用

### 6. sakura-ai/index.html 側の対応

`?embed=1` パラメータ受信時：
- ヘッダー部分を非表示（またはコンパクト化）
- `window.addEventListener('message')` でテキスト受信 → 入力欄にセット → 自動送信

## postMessage プロトコル

```js
// SAKURA OS → SAKURA AI
{ type: 'send', text: 'メッセージ本文' }
```

```js
// sakura-ai/index.html 受信処理
window.addEventListener('message', (e) => {
  if (e.data?.type === 'send' && e.data.text) {
    // メッセージ欄にテキストをセットして送信
  }
});
```

## 変更ファイル

1. `C:\Users\MYPC\Development\SAKURA-OS\sakura-os.html` — レイアウト・CSS・JS変更
2. `C:\xampp\htdocs\sakura-ai\index.html` — embed対応・postMessage受信
