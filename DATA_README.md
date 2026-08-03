# データ供給の仕組み（同梱曲）

game.html は、起動時に `songs.json`（manifest）を fetch して「同梱曲」を一覧に加えます。
ブラウザはフォルダの中身を自動スキャンできないため、**曲の一覧は必ず manifest に列挙**します。
個々のファイル名は下の命名規則から自動で組み立てて取得します。

ユーザーが選曲画面の [+] で追加した曲（IndexedDB 保存）と、この同梱曲は同じリストに統合表示されます。
スコアは系統を問わず localStorage（キー `rhythmScores`）に一元保存されます。

## 参照先（DATA_BASE）

- 既定は `./`（game.html と同じ場所）。
- 変更したい場合、ブラウザのコンソールで
  `localStorage.setItem('rhythmDataBase','https://例.com/data/')`
  のように設定すれば、ローカルHTTP → GitHub Pages → CDN へ切替できます（末尾の `/` を忘れずに）。

## フォルダ構成

```
（game.html と同じ場所）
├─ game.html
├─ player.html
├─ songs.json                 ← manifest（曲一覧）
└─ songs/
   ├─ song01/
   │  ├─ audio.ogg            （曲名は自由。manifest の "audio" と一致させる）
   │  ├─ jacket.png           （任意。"jacket" と一致）
   │  ├─ chart_e.json         （Easy 譜面。"diffs.Easy.chart" と一致）
   │  ├─ chart_h.json         （Hard 譜面）
   │  └─ chart_m.json         （Master 譜面）
   └─ song02/
      └─ ...
```

## manifest（songs.json）の形式

`songs.example.json` を参照。要点だけ：

- `id` … 曲ごとの一意ID。**フォルダ名になる**ので、英数字とアンダースコア推奨（日本語や空白は避ける）。
- `source` … 曲自体の入手方法。`free`（最初から所持）/ `shop`（ショップで購入）/ `partner` / `event`。省略時は `free`。`free` 以外は購入・解禁するまで選曲画面に出ません。
- `title` / `artist` / `charter` … 表示用のメタ情報（ここは日本語でOK）。
- `audio` … 音源ファイル名（`songs/<id>/` 内）。
- `jacket` … ジャケット画像名（任意。無ければ「ジャケット準備中」）。
- `diffs` … 存在する難易度だけ書く（`Easy` / `Hard` / `Master`）。各 `level` と `chart`（譜面ファイル名）。
  - `unlock` … 任意。付けるとその難易度は「未解禁」になり、選曲画面から解禁が必要。`{"cost": 5000}` で解禁に必要な通貨(₡)を指定。解禁は「₡支払い」または「難易度解禁チケット1枚」の2通り。`unlock` を書かなければ最初から解禁済み。

## プレイ背景・パートナー背景

プレイ中の背景は 1280×720 の座標系で合成されます。曲ジャケットは固定位置
（x=90, y=120, 410×410px）に cover 表示で重なります。設定「プレイ背景」で選べます：

- デフォルト … 黒地＋固定位置ジャケット
- 黒一色 … 真っ黒（ジャケットなし）
- パートナー背景 … 解禁済みパートナーの `bg`（1280×720）＋固定位置ジャケット

パートナーの立ち絵・アイコン・背景（partners.json の `chara`/`icon`/`bg`）は **`images/<ファイル名>`** に置きます（images.json と同じフォルダ）。ノーツスキン画像は `skins/`、曲のジャケット等は `songs/<id>/` です。
`partner_bg_mask.png` は、ジャケットが重なる領域（x=90,y=120,410×410）を黒で示した
白黒2値マスクです。背景を描く際、この黒い部分はジャケットで隠れるので、重要な絵は
避けて配置してください。`partner_bg_guide.png` は枠線つきの位置合わせ用テンプレートです。

## 注意点（本番配信で効く）

- **拡張子は小文字で統一**。GitHub Pages は大文字小文字を区別します（`.PNG` は事故のもと）。
- 譜面 JSON は編集エディタの書き出し、または選曲画面（デバッグ）の「この譜面をDL」で作れます。
- `file://`（HTMLをダブルクリック）では fetch が使えず、同梱曲は読めません。**ローカルHTTPで確認**してください。

## ローカルHTTPでの確認方法

game.html のあるフォルダで、次のいずれかを実行してブラウザで `http://localhost:8000/game.html` を開きます。

```
# Python がある場合
python -m http.server 8000

# Node がある場合
npx serve .
```
