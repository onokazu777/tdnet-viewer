# プログラム一覧

本リポジトリにPythonやワークフローはありません。実行可能な本体は`index.html`のみです。データ生成プログラムは[`tdnet_get`](https://github.com/onokazu777/tdnet_get)側にあります。

## 本リポジトリ

### `index.html`

- 目的: XBRL財務指標の一覧表示と詳細モーダル表示
- 実行場所: ユーザーのブラウザ（GitHub PagesまたはローカルHTTPサーバー）
- 入力:
  - `data/index.json`
  - `data/detail/*.json`（必要時）
- 出力: ブラウザ画面のみ（ファイルは書き出しません）
- 主な機能:
  - 日付・コード・増収率・営業利益率・PBR・予想PER・配当利回りでのソート
  - 会社名・コード・表題の部分一致検索
  - 日付フィルタ
  - 会社名クリックで詳細JSONを読み込み、シートをタブ表示
  - 表題からTDnet PDFを別タブで開く
- 実装形態: 単一HTML内のCSS + JavaScript（外部JSライブラリなし）

#### 画面操作と処理対応

| UI | 処理 |
|---|---|
| ソートセレクト | `sortCol` / `sortAsc`を更新して再描画 |
| ヘッダークリック | 同じ列なら昇降順切替。コードは初期昇順、他は降順 |
| 検索入力 | 200msデバウンス後に再描画 |
| 日付セレクト | `date_raw`で絞り込み |
| 会社名 | `openDetail` → `fetch('data/detail/' + detail)` |
| 表題リンク | `pdf_url`があれば外部リンク |
| モーダル閉じる | ボタン、オーバーレイクリック、`Escape` |

#### 詳細表示

1. `detail.sheets`のキーをタブにする
2. `分析サマリー`は会社情報グリッドと【見出し】単位の表に変換
3. その他シートは先頭行をヘッダ、以降をデータ行として表表示
4. 金額・増減率などの列は数値整形と正負色分けを行う

## データ生成側（`tdnet_get`）

本リポジトリへ書き込む処理の対応関係です。詳細は`tdnet_get`のドキュメントを参照してください。

| 成果物 | 生成プログラム | Actionsでの扱い |
|---|---|---|
| `data/index.json` | `⑤_export_json.py` | 既存一覧と`detail`キーでマージ |
| `data/detail/*.json` | `⑤_export_json.py` | 未配置ファイルのみコピー |
| `data/search/*.csv` | `②a②bは２つフリーワード検索.py` | `pdf_tmp`からコピー |
| `data/text/text_*.json` | `⑥_pdf_text_extractor.py` | `text_data`からコピー |
| `data/text/index.json` | Actions内のマージスクリプト | `text_*.json`から再生成 |
| 古い`text_*.json`削除 | Actions内のマージスクリプト | 180日より前を削除 |

関連ワークフロー:

- `.github/workflows/daily_update.yml`（`Daily XBRL Update`）
  - clone → マージ/コピー → `git commit` / `git push`
- `.github/workflows/keepalive.yml`
  - `tdnet_get`側のスケジュール維持用。本リポジトリは更新しません

## 依存関係

本リポジトリ:

- なし（ブラウザの標準機能のみ）

データ生成側（参考）:

- Python 3.11、pandas、openpyxl、yfinance など（`tdnet_get/requirements.txt`）
