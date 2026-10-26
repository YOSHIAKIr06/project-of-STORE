# STORE — Firebase本番デプロイ運用手順

## 目的

このプロジェクトでは、GitHubへ通常のコード変更をpushしただけではFirebase本番環境へデプロイしません。

本番デプロイは、明示的に「デプロイする」と判断したときだけ実行します。

## 全体の流れ

1. STOREのコードをGitHubへ保存する
2. 通常のpushではFirebaseは更新されない
3. 本番公開すると決めたら `.deploy/deploy-request.json` を更新する
4. GitHub Actionsの `Firebase Production Deploy` が起動する
5. `npm ci` を実行する
6. `npm run build` でViteをビルドする
7. `dist/` をFirebase Hostingのliveチャンネルへデプロイする
8. 実行結果・成功／失敗・ログはGitHub Actionsに残る

## 自動で本番公開されない仕組み

ワークフローは次の2通りの場合だけ起動します。

- `main` ブランチの `.deploy/deploy-request.json` が更新されたとき
- GitHub Actions画面から `Run workflow` を手動実行したとき

`src/` や `public/` など通常のソースコードだけを更新しても、本番デプロイは実行されません。

## ChatGPTからデプロイを依頼する場合

ユーザーが明示的に「Firebaseへデプロイして」と依頼した場合、ChatGPTは `.deploy/deploy-request.json` を更新します。

そのコミットがGitHubに残り、GitHub Actionsが本番デプロイを実行します。

これにより、いつ・どのタイミングでデプロイ要求を出したかをGitHubのコミット履歴から追跡できます。

## GitHub側で必要な設定

GitHubの `production` Environment に、次の値を登録します。

### Variable

- `FIREBASE_PROJECT_ID`
  - FirebaseプロジェクトID

### Secret

- `FIREBASE_SERVICE_ACCOUNT`
  - Firebase Hostingへのデプロイに使うサービスアカウントJSON全文

これらが未設定の場合、GitHub Actionsは意図的に失敗し、本番公開は行われません。

## Firebase認証方式

このプロジェクトでは、レガシーな `FIREBASE_TOKEN` は使用しません。

GitHub Actionsからは `FirebaseExtended/action-hosting-deploy@v0` を使用し、`FIREBASE_SERVICE_ACCOUNT` に保存したサービスアカウントJSONで認証します。

認証情報はコードへ直接書かず、GitHub Secretsで管理します。

## Firebase Hosting設定

`firebase.json` では、Viteのビルド成果物である `dist/` を公開対象にしています。

SPAとして動作できるように、すべてのURLを `/index.html` へrewriteしています。

本番デプロイ時はFirebase Hostingの `live` チャンネルへ公開します。

## 初回セットアップ

初回だけ、FirebaseプロジェクトとGitHub Secretsを準備します。

1. FirebaseコンソールでSTORE用プロジェクトを用意する
2. Hostingを有効化する
3. FirebaseプロジェクトIDを確認する
4. デプロイ用サービスアカウントを用意する
5. GitHubの `production` Environmentへ次を登録する
   - Variable: `FIREBASE_PROJECT_ID`
   - Secret: `FIREBASE_SERVICE_ACCOUNT`
6. `.deploy/deploy-request.json` を更新してテストデプロイする

## デプロイ履歴の確認方法

GitHubリポジトリの `Actions` を開き、`Firebase Production Deploy` を選択します。

各実行ごとに次を確認できます。

- 実行日時
- 対象コミット
- 成功／失敗
- ビルドログ
- Firebaseデプロイログ
- 実行サマリー

また、`.deploy/deploy-request.json` のコミット履歴を見れば、明示的にデプロイ要求を出したタイミングも追跡できます。

## 運用ルール

- 通常のコード変更と本番デプロイを分離する
- 本番デプロイは明示的な依頼があった場合のみ行う
- 大きな変更の前後ではGitHub上のコミットを確認する
- デプロイ失敗時はGitHub Actionsのログを確認してから再実行する
- 認証情報はコードへ直接書かず、GitHub Secretsで管理する
- `FIREBASE_SERVICE_ACCOUNT` の内容はREADMEやIssueへ貼らない

## 関連ファイル

- `firebase.json` — Firebase Hosting設定
- `.github/workflows/firebase-production-deploy.yml` — 本番デプロイ用GitHub Actions
- `.deploy/deploy-request.json` — デプロイ要求記録

