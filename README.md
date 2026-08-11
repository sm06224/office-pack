# office-pack

**PKC3 の Office 表示(LibreOffice wasm / Qt6)で使う一式を配るだけ**のリポジトリです。
アプリ本体は [sm06224/PKC3](https://github.com/sm06224/PKC3) にあります。

配信先: `https://sm06224.github.io/office-pack/`

## なぜ本体と分けてあるか

GitHub Pages は**同一ユーザーの全リポジトリが `https://<user>.github.io` という
同じ origin** に載ります。つまり別リポジトリにしても PKC3 本体から見て**同一 origin**で、
**CORS は起きません**。そのうえで:

- 更新周期が違う(LibreOffice は年数回 / PKC3 本体は毎日)
- 本体の deploy 時間にも、本体の検品(`check-dist`)にも**一切影響しない**
- 93MB を本体の git 履歴にも deploy artifact にも持ち込まない

🚫 **GitHub Release の資産を JS から直接 fetch する道は塞がっています。**
release download は `Access-Control-Allow-Origin` を 1 つも返さず、`OPTIONS` は 405 です
(2026-08-10 実測。実ブラウザでも `TypeError: Failed to fetch`)。
⚠ `coi-serviceworker` はこれを救いません ── あれが解くのは **COOP/COEP** であって
**CORS** ではないからです。だから「Pages に置く = 同一 origin にする」が要ります。

## 何が置かれるか(約 93MB)

| file | 中身 |
|---|---|
| `soffice.wasm.gz` | LibreOffice 本体 148.9MB → **50.6MB**(gzip -9、2.94x) |
| `soffice.data.gz` | データパッケージ 83.6MB → **26.4MB**(3.17x) |
| `soffice.js` / `qtloader.js` / `soffice.data.js.metadata` | 起動に要る |
| `fonts/BIZUD*.ttf` | **日本語フォント**。LibreOffice の同梱 128 本 / 51.2MiB に **CJK が 1 つも無い**ため |
| `index.html` | 単体で試せる頁(ファイルを選ぶと開く) |

`.gz` は **JS 側が `DecompressionStream` で解く**ので、素のバイト列として配られます。

## 更新のしかた

1. PKC3 の `office-wasm-build` workflow で LibreOffice を焼く(prerelease `lo-wasm-dev` が出る)
2. このリポジトリの **`pages` workflow を手で回す**(Actions → pages → Run workflow)
   - `pkc3_ref` は既定 `main`。⚠ 組み立て script が **main にまだ入っていない間**は、
     作業中の branch 名(例: `claude/pkc3-pr-101-n05trv`)を入れて回す
   - `lo_tag` は既定 `lo-wasm-dev`(PKC3 の prerelease)

⚠ **自動追従はしません**。LibreOffice の版が変わるのは大事なので、明示操作にしてあります。
組み立ての正本は PKC3 の `build/office-wasm/make-pages-bundle.mjs` に**1 つだけ**あり、
この workflow はそれを checkout して呼ぶだけです ── 一覧を 2 か所に持ちません。
