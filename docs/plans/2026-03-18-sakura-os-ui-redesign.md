# SAKURA OS UI リデザイン 実装プラン

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** SAKURA OSを3カラム構成に変更し、右半分にSAKURA AI iframeを追加、中央列下部にAIへのクイック入力欄を設置する。

**Architecture:** `#info-area`をflex-column・スクロール可能なセンター列に変換し、absoluteポジションだった子要素を通常フローに移行。右列は新規`#ai-panel`として`sakura-ai/index.html?embed=1`をiframe表示。postMessageでセンター列の入力欄とiframeを連携。

**Tech Stack:** Vanilla JS / CSS / HTML, Firebase Firestore（変更なし）, postMessage API

---

## Task 1: #info-area CSS をflex-column構成に変換

**Files:**
- Modify: `C:\Users\MYPC\Development\SAKURA-OS\sakura-os.html:189-191`（#info-area CSS）
- Modify: `C:\Users\MYPC\Development\SAKURA-OS\sakura-os.html:192-219`（currency-row / news-grid CSS）
- Modify: `C:\Users\MYPC\Development\SAKURA-OS\sakura-os.html:222-225`（#dcal-wrapper CSS）

**Step 1: #info-area を半幅のflex-column に変更**

行189〜191を以下に置き換える：

```css
#info-area {
    position:fixed; left:272px; top:32px; bottom:48px;
    width:calc((100vw - 272px) / 2); z-index:10;
    overflow-y:auto; overflow-x:hidden;
    display:flex; flex-direction:column; gap:0;
}
#info-area::-webkit-scrollbar { width:3px; }
#info-area::-webkit-scrollbar-thumb { background:rgba(255,255,255,0.08); border-radius:2px; }
```

**Step 2: .currency-row をflex先頭に合わせる**

行192を以下に置き換える：

```css
.currency-row { display:flex; gap:12px; justify-content:flex-end;
    padding:14px 14px 0; flex-shrink:0; }
```

（`position:absolute` と `top/right` を削除）

**Step 3: .news-grid を通常フローに変更（縮小）**

行205〜208を以下に置き換える：

```css
.news-grid {
    margin:8px 14px 0; flex-shrink:0;
    display:grid; grid-template-columns:repeat(3,1fr);
    grid-template-rows:repeat(3,80px); gap:6px;
}
```

（`position:absolute; top:104px; left:14px; right:14px` を削除。行高を 110px→80px に縮小）

**Step 4: ニュースカード内テキストを縮小**

行217〜218を以下に置き換える：

```css
.news-src { font-size:11px; font-weight:700; color:#f06292; letter-spacing:0.3px; margin-bottom:4px; }
.news-title { font-size:12px; font-weight:700; color:var(--text); line-height:1.4;
    display:-webkit-box; -webkit-line-clamp:2; -webkit-box-orient:vertical; overflow:hidden; flex:1;
    text-shadow:0 1px 8px rgba(0,0,0,1); }
.news-time { font-size:11px; color:var(--text-muted); margin-top:4px; }
```

**Step 5: #dcal-wrapper を通常フローに変更**

行222〜225を以下に置き換える：

```css
#dcal-wrapper {
    margin:8px 14px 0; flex-shrink:0;
    display:flex; gap:12px; align-items:flex-start;
}
```

（`position:absolute; top:464px; left:14px; right:14px` を削除）

**Step 6: ブラウザで表示確認**

`C:\xampp\htdocs\（または直接ファイルを開く）` でニュース3×3が小さく表示され、カレンダーがその下に続くことを確認。

---

## Task 2: #ai-panel（右列）と AIクイック入力欄を追加

**Files:**
- Modify: `C:\Users\MYPC\Development\SAKURA-OS\sakura-os.html`（CSS追加 + HTML追加）

**Step 1: #ai-panel CSS を追加**

行267（`/* ── Windows ── */` の直前）に追加：

```css
/* ── AI Panel ── */
#ai-panel {
    position:fixed; right:0; top:32px; bottom:48px;
    width:calc((100vw - 272px) / 2); z-index:10;
    display:flex; flex-direction:column;
    border-left:1px solid rgba(255,255,255,0.08);
}
#ai-iframe {
    flex:1; width:100%; border:none; display:block;
}

/* ── AI Quick Input（センター列最下部）── */
#ai-chat-bar {
    flex-shrink:0; margin:10px 14px 12px; display:flex; gap:8px; align-items:flex-end;
}
#os-ai-input {
    flex:1; background:rgba(255,255,255,0.06); border:1.5px solid rgba(255,255,255,0.3);
    border-radius:12px; color:var(--text); font-size:14px; font-family:inherit;
    padding:10px 14px; resize:none; outline:none; line-height:1.5; max-height:80px;
    transition:border-color var(--tr);
}
#os-ai-input:focus { border-color:var(--accent); }
#os-ai-input::placeholder { color:var(--text-muted); }
#os-ai-send {
    background:var(--accent); border:none; border-radius:10px; color:#fff;
    width:40px; height:40px; font-size:18px; cursor:pointer; flex-shrink:0;
    transition:opacity var(--tr), transform var(--tr);
}
#os-ai-send:hover { opacity:0.85; transform:scale(1.05); }
[data-theme="light"] #os-ai-input { background:rgba(0,0,0,0.04); border-color:rgba(0,0,0,0.15); }
```

**Step 2: #ai-chat-bar HTML を info-area内の最下部に追加**

行487（`</div><!-- /info-area -->` の直前）に追加：

```html
    <div id="ai-chat-bar">
        <textarea id="os-ai-input" rows="1" placeholder="💬 SAKURA AIに話しかける...（Enter送信）"></textarea>
        <button id="os-ai-send" title="送信">➤</button>
    </div>
```

**Step 3: #ai-panel HTML を info-areaの直後に追加**

行488（`<!-- Windows Container -->` の直前）に追加：

```html
<!-- AI Panel -->
<div id="ai-panel">
    <iframe id="ai-iframe"
        src="/system/sakura-ai/?embed=1"
        frameborder="0"
        allow="microphone"
        loading="lazy">
    </iframe>
</div>
```

**Step 4: osAiSend() JS関数を追加**

ファイル末尾の `</script>` の直前に追加：

```js
// ── AI Quick Input ─────────────────────────────
const osAiInput = document.getElementById('os-ai-input');
const osAiFrame = document.getElementById('ai-iframe');

osAiInput.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' && !e.shiftKey) {
        e.preventDefault();
        osAiSend();
    }
});
osAiInput.addEventListener('input', () => {
    osAiInput.style.height = 'auto';
    osAiInput.style.height = Math.min(osAiInput.scrollHeight, 80) + 'px';
});

function osAiSend() {
    const text = osAiInput.value.trim();
    if (!text) return;
    osAiFrame.contentWindow.postMessage({ type: 'send', text }, '*');
    osAiInput.value = '';
    osAiInput.style.height = 'auto';
}
```

**Step 5: ブラウザで確認**

- 右半分にiframeが表示されること
- 中央列最下部に入力欄があること

---

## Task 3: sakura-ai/index.html に embed対応 + postMessageリスナー追加

**Files:**
- Modify: `C:\xampp\htdocs\sakura-ai\index.html`

**Step 1: embed CSS を追加**

`</style>` タグの直前（CSSの末尾）に追加：

```css
/* ── Embed Mode（?embed=1） ── */
body.embed-mode #sidebar,
body.embed-mode .sidebar-overlay { display:none !important; }
body.embed-mode #chat-area { margin-left:0 !important; width:100% !important; }
```

**Step 2: embed検出 + postMessageリスナーをJS初期化ブロックに追加**

`fetchModels();` の直前（行1414付近）に追加：

```js
// embed mode
if (new URLSearchParams(location.search).has('embed')) {
    document.body.classList.add('embed-mode');
}

// postMessage受信（SAKURA OSからのメッセージ）
window.addEventListener('message', (e) => {
    if (e.data?.type === 'send' && e.data.text && !isLoading) {
        sendMessage(e.data.text);
    }
});
```

**Step 3: ブラウザで `/system/sakura-ai/?embed=1` を開いてサイドバーが消えることを確認**

**Step 4: git commit (SAKURA-AI)**

```bash
cd C:\xampp\htdocs\sakura-ai
git add index.html
git commit -m "feat: embed mode対応 + SAKURA OSからのpostMessage受信"
git push origin master
```

---

## Task 4: SAKURA-OS git commit & push

**Files:**
- `C:\Users\MYPC\Development\SAKURA-OS\sakura-os.html`

**Step 1: バージョン番号を更新**

行8: `<meta name="version" content="1.5.9">` → `content="1.6.0"`
行8: `<title>SAKURA OS v1.5.9</title>` → `v1.6.0`

**Step 2: git commit & push**

```bash
cd C:\Users\MYPC\Development\SAKURA-OS
git add sakura-os.html
git commit -m "feat: v1.6.0 - 3カラム構成・SAKURA AI chat panel追加"
git push origin master
```

**Step 3: WinSCPでアップロード**

- `sakura-os.html` → `/system/sakura-os.html`
- `sakura-ai/index.html` → `/system/sakura-ai/index.html`

---

## 確認チェックリスト

- [ ] ニュース3×3が縮小されてセンター列に収まる
- [ ] カレンダー・作業予定がニュース直下に表示される
- [ ] センター列最下部にAI入力欄がある
- [ ] 右半分にSAKURA AI iframeが表示される
- [ ] OS入力欄からEnterでAI iframeにメッセージが届く
- [ ] `?embed=1` でサイドバーが消える
- [ ] ダーク／ライトテーマどちらでも崩れない
