# LR2 Score Viewer

LR2のスコアDBから、難易度表のランプ・最小BP・スコアレートと日別のプレイ記録を表示するWebページです。GitHub Pagesで公開すれば、自宅のPCが停止していてもスマートフォンから最後に更新した記録を閲覧できます。

このリポジトリは配布用です。実際のスコア・プレイ履歴・ローカル設定は含みません。初期画面には設定案内を表示し、「デモ」を押した場合だけ架空のデータを表示します。

## できること

- 複数の難易度表を切り替えて、レベル別のランプを集計。グラフを押すとランプの内訳を表示します。
- 譜面一覧にランプ・最小BP・スコアレートを表示。曲名検索、レベル・ランプ絞り込み、並べ替えに対応します。
- 長い曲名は省略し、押すと全文を表示します。
- 日別に総ノーツ数・プレイ数・演奏時間と、ランプ・BP・スコアの更新を表示します。
- 難易度表外の譜面やコースのタイトルもLR2の曲情報DBから取得します。
- スコアDBを日時付きでバックアップできます。

## 1. 必要なもの

Windows、GitHubアカウント、Python 3.10以上、Git、LR2のスコアDBが必要です。PowerShellで以下が実行できることを確認してください。

```powershell
python --version
git --version
```

Pythonは[公式サイト](https://www.python.org/downloads/windows/)、Gitは[公式サイト](https://git-scm.com/downloads/win)から導入できます。Pythonのインストール時にはPATHに追加してください。Pythonの追加ライブラリは不要です。

日別記録には、[BeMusicSeekerのLR2プレイログ機能](https://neeted.github.io/bemusicseeker-unofficial-fork/manual.ja.html#lr2プレイログ)で記録した `bms_lr2_play_history` が必要です。ログがなくても難易度表のランプ・BP・スコアレートは利用できます。導入前の過去プレイは復元できません。

## 2. 自分のリポジトリを作る

[このリポジトリ](https://github.com/1834356/lr2-score-viewer)の「Fork」から自分のアカウントへフォークします。「Use this template」から作成することもできます。リポジトリ名は `lr2-score-viewer` を想定しています。無料のGitHub Pagesを使用する場合は公開リポジトリで作成してください。

PowerShellで、`YOUR_NAME` を自分のGitHubユーザー名に置き換えて実行します。

```powershell
git clone https://github.com/YOUR_NAME/lr2-score-viewer.git
cd lr2-score-viewer
Copy-Item config.example.json config.local.json
```

Gitのコミット用の名前・メールが未設定の場合は、このフォルダ内で設定します。メールにはGitHubの設定画面にある非公開用のnoreplyアドレスも使えます。

```powershell
git config user.name "自分の名前"
git config user.email "自分のメールアドレス"
```

## 3. DBとバックアップ先を設定する

`config.local.json` をメモ帳などで開き、自分の環境に合わせて変更します。

```json
{
  "database": "D:/LR2/LR2files/Database/Score/your-player.db",
  "backupDirectory": "C:/BMSbackup/scoredb",
  "tableUrls": [
    "https://stellabms.xyz/sl/table.html",
    "https://stellabms.xyz/st/table.html",
    "https://bms-ir.org/new/table/16",
    "https://miraiscarlet.github.io/bms/table/genocide_insane/insane_bms.html"
  ]
}
```

- `database`: 使用しているLR2プレイヤーのスコアDB。通常は `LR2files/Database/Score/` 内の `.db` ファイルです。
- `backupDirectory`: バックアップ先のフォルダ。存在しなければ自動作成します。OneDrive内のフォルダも指定できます。バックアップが不要なら、この項目を削除できます。
- `tableUrls`: 表示したい難易度表のURL。初期設定はSatellite、Stella、Favorite、発狂BMS難易度表です。

JSONのパスは例のように `/` で区切ると、そのまま記入できます。`\` を使う場合は `\\` と記入してください。最後の項目にはカンマを付けません。

譜面タイトル用の `song.db` は、スコアDBの `Score` フォルダの1つ上から自動検出します。別の場所の場合は `"songDatabase": "D:/LR2/LR2files/Database/song.db"` を設定に追加してください。

`config.local.json` と元DBはGitの対象外です。自分の設定を `config.example.json` に書き込まないでください。

## 4. 最初の更新を実行する

このフォルダ内の `update-and-backup.cmd` をダブルクリックします。設定したDBをバックアップし、難易度表とスコアをJSONへ書き出して、自分のリポジトリに同期します。初回は自分のGitHubアカウントで認証してください。

PowerShellから実行する場合は次のコマンドです。

```powershell
.\update.ps1 -Push
```

スクリプトはこのフォルダのGitリモート `origin` の `main` に同期します。フォークしたリポジトリをクローンした場合は、自分のリポジトリが更新先になります。

バックアップやデータ出力に失敗すると、以降の更新を中止します。難易度表を取得できなかった場合も、以前の閲覧用JSONを維持します。

## 5. GitHub Pagesで公開する

自分のリポジトリの **Settings → Pages** を開き、以下を設定して保存します。

- Source: **Deploy from a branch**
- Branch: **main**
- Folder: **/(root)**

公開処理が完了すると、Pagesの設定画面に閲覧URLが表示されます。リポジトリ名が `lr2-score-viewer` なら、通常は `https://YOUR_NAME.github.io/lr2-score-viewer/` です。スマートフォンからこのURLを開いてください。

参考: [GitHub Pagesの公開元を設定する](https://docs.github.com/ja/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

## 6. プレイ後に更新する

LR2でプレイした後、`update-and-backup.cmd` を実行します。公開処理の完了後、閲覧ページを再読み込みすると新しい記録を確認できます。自動実行は設定していません。

GitHubに同期せず、バックアップとJSON出力だけ実行する場合:

```powershell
.\update.ps1
```

バックアップだけ実行する場合:

```powershell
.\update.ps1 -BackupOnly
```

バックアップは `<DB名>_YYYYMMDD_HHMMSS_ffffff.db`（日本時間）で保存し、過去のファイルは上書き・削除しません。SQLiteのバックアップ機能で整合性を保ってコピーし、確認を通過したものだけを保存します。

## 難易度表を追加・削除する

`config.local.json` の `tableUrls` を編集して保存し、`update-and-backup.cmd` を実行します。

**追加:** 配列の中に難易度表のURLを追加します。各URLを `"` で囲み、項目間をカンマで区切ってください。

**削除:** 不要なURLの行を削除します。例えばSatelliteとStellaだけにする場合は以下です。

```json
"tableUrls": [
  "https://stellabms.xyz/sl/table.html",
  "https://stellabms.xyz/st/table.html"
]
```

上の断片は設定ファイル内の `tableUrls` 部分を置き換える例です。`database` と `backupDirectory` の設定は残してください。

難易度表は1つ以上必要です。URLは重複させず、MD5を含む一般的な難易度表形式のページ、またはヘッダJSONを指定してください。MD5がない譜面はスコアと照合できないため除外します。対応していない形式や取得できないURLがある場合は、更新時にエラーを表示します。

表の削除でLR2のスコアや譜面ファイルが削除されることはありません。日別記録は難易度表外も含む全譜面を集計し、表を削除した後も履歴を表示します。

## データの公開範囲と計算方法

GitHub Pagesのページと `data/viewer.json` は公開です。公開JSONには曲名・ランプ・BP・EXスコア・総ノーツ数・日別記録などを含みます。元DB、認証情報、ゴースト、ローカルパスは公開JSONに出力しません。更新スクリプトがコミットするファイルは `data/viewer.json` だけです。

スコアレートは `EXスコア ÷ (総ノーツ数 × 2) × 100`、EXスコアは `perfect × 2 + great` で計算します。未プレイや総ノーツ数未取得の場合は「—」を表示します。

日別の打鍵数は `new_totalnotes × player_playcount_delta` の合計です。途中終了・FAILEDでも譜面全体のノーツ数を加算し、総ノーツ数未取得のプレイは除外します。実際の判定数や物理的なキー押下数とは異なります。確定済みログのみ日本時間で集計し、複数の難易度表に所属する譜面も重複集計しません。

ランプ更新は通常ランプの上昇、BP更新は以前の最小BPの減少、スコア更新は以前のEXスコアの上昇です。初回BP・スコア記録は改善に含めません。新規AAAは最大スコアの8/9以上になった譜面です。

## よくある確認事項

- 「未設定」と表示される: 自分のDBを設定し、更新を正常に完了させてください。初期JSONは空です。
- 日別記録が出ない: BeMusicSeeker側のログ設定と、確定済みログがDBにあるかを確認してください。
- 曲名が取得できない: `song.db` の場所を確認し、必要なら `songDatabase` を指定してください。
- GitHubの同期に失敗する: `git remote -v` で自分のリポジトリが `origin` になっているかと、認証・コミット用の名前とメールを確認してください。
- PowerShellがスクリプト実行を禁止する: Windowsの実行ポリシーを確認してください。自身のPCで許可する場合は `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` を使用できます。組織管理のPCでは管理者の設定に従ってください。

## 技術構成と確認

フロントエンドはHTML/CSS/JavaScript、エクスポータとバックアップはPython標準ライブラリだけを使用します。DBは読み取り専用で開き、プレイログ用のテーブルやトリガーは追加しません。ページは同じ公開先の相対パス `data/viewer.json` を読むため、フォーク先のデータを表示します。

```powershell
python -X utf8 -m unittest discover -s tests -p "test_*.py"
node tests/test_daily_groups.js
```
