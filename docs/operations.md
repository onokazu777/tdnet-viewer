# 運用・障害対応

## 通常運用

本リポジトリの更新は、`tdnet_get`のGitHub Actionsが自動で行います。本リポジトリ側で定期ジョブを動かす必要はありません。

更新元スケジュール（JST）:

- 平日15:35
- 平日17:05
- 平日20:05
- 平日23:55

確認先:

- 公開Viewer: https://onokazu777.github.io/tdnet-viewer/
- データ更新Actions: https://github.com/onokazu777/tdnet_get/actions
- 本リポジトリのコミット: https://github.com/onokazu777/tdnet-viewer/commits/main

成功時のコミットメッセージ例:

```text
Auto update: 20260717
Auto update: 20260714 20260717
```

## 手動でデータを更新する

本リポジトリを直接編集して日次データを足す運用は想定していません。欠落日がある場合は`tdnet_get`側を手動実行します。

1. https://github.com/onokazu777/tdnet_get/actions を開く
2. `Daily XBRL Update`を選ぶ
3. `Run workflow`
4. branchは`main`
5. `target_date`に対象日を入れる
6. 実行後、本リポジトリに`Auto update: ...`コミットが付くか確認
7. GitHub Pages反映を待ち、Viewerの最新日を確認

`target_date`の例:

```text
空欄                 今日
20260717             1日
202607               1か月
20260714 20260717    日付範囲
```

## 成功確認

1. `tdnet_get`のActionsが成功している
2. 特に`Push to viewer repo`が成功、または`No new data`である
3. 本リポジトリの最新コミット日時が想定どおり
4. https://onokazu777.github.io/tdnet-viewer/ で最新日が一覧に出る

Pages反映には数分かかることがあります。古い画面が残る場合は`Ctrl+F5`で再読み込みします。

## よくある障害

### Viewerの日付が止まっている

確認順:

1. `tdnet_get`の直近Actions実行を見る
2. workflowが無効化されていないか見る
3. 失敗ステップのログを見る
4. 本リポジトリの最新コミットを見る
5. Pagesの構築完了を待つ

過去の典型原因:

- `tdnet_get`が公開リポジトリで60日間無活動扱いになり、scheduled workflowが停止した

対処:

1. `tdnet_get`で`Daily XBRL Update`を`Enable workflow`
2. 欠けた日付を手動`Run workflow`
3. 本リポジトリへのpushとPages反映を確認

### `tdnet-viewer`へのpushが失敗する

`tdnet_get`の`Clone viewer repo`または`Push to viewer repo`を確認します。

主な原因:

- `VIEWER_PAT`の期限切れ
- PATに本リポジトリへの書き込み権限がない
- 本リポジトリの権限変更

対処:

1. 新しいPATを発行し、本リポジトリへの読み書きを付与
2. `tdnet_get`のRepository secret `VIEWER_PAT`を更新
3. 対象日を手動再実行

Secret設定:

https://github.com/onokazu777/tdnet_get/settings/secrets/actions

### 一覧は出るが詳細が開かない

確認:

- 該当行の`detail`ファイルが`data/detail/`にあるか
- ブラウザの開発者ツールで`data/detail/...`のHTTPステータス

詳細は未配置時のみコピーされるため、過去にコピー失敗したファイルは再実行でも上書きされない場合があります。そのときは対象のdetailファイル有無をActionsログとリポジトリ実体の両方で確認します。

### 表題リンクのPDFが開けない

TDnetのPDF URLは公開から約30日程度で無効になることがあります。Viewerの表題リンクはTDnet直リンクです。長期保管分は`tdnet_get`がGoogle Driveへ保存しています。

### クラウド全文検索で古い日が出ない

`data/text`は180日保持です。期限切れファイルはActionsのマージ時に削除され、`data/text/index.json`からも消えます。XBRL一覧（`index.json`）の保持期間とは別です。

### ローカルで`index.html`を直接開くとデータが出ない

`file://`では`fetch`が制限されることがあります。次で確認します。

```powershell
python -m http.server 8080
```

http://localhost:8080/

## 本リポジトリで手動変更してよいもの

| 対象 | 方針 |
|---|---|
| `README.md` / `docs/` | ドキュメント更新は手動コミットしてよい |
| `index.html` | UI変更は手動。データ生成には影響しない |
| `data/*` | 原則Actions任せ。手編集は不整合の原因になる |

`index.html`を変更してpushした場合も、GitHub Pagesが再構築します。

## 認証情報

本リポジトリにはSecretsを置きません。push用トークンは`tdnet_get`の`VIEWER_PAT`のみです。

値をREADME、Issue、PR、チャット、Actionsログへ貼らないでください。漏えい時はPATを失効させて再発行します。

## 定期点検

月1回程度:

- Viewerの最新日が直近の平日開示に追随しているか
- `tdnet_get`の`Daily XBRL Update`が成功しているか
- 本リポジトリに`Auto update`コミットが付いているか
- `VIEWER_PAT`の有効期限
- クラウド検索用に`data/text/index.json`の`updated`が新しいか

## 既知の注意点

- 本リポジトリ単体ではデータ再生成できません。
- `data/search`と`data/text`はViewer画面の入力ではありません。
- `data/stock_cache.json`は現行Viewer未使用で、現行Actionsマージでも更新されません。
- `④_xbrl_viewer.py`（`tdnet_get`内）はローカルExcel用Streamlitで、本GitHub Pages版とは別実装です。
