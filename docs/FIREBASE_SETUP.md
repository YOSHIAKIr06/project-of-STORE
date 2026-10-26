# STORE — Firebase初回セットアップ手順

この文書は、STOREをGitHub ActionsからFirebase Hostingへ本番デプロイできるようにするための初回設定手順です。

## 1. Firebaseプロジェクトを用意する

FirebaseコンソールでSTORE用のFirebaseプロジェクトを作成するか、既存のSTORE用プロジェクトを使用します。

確認しておく値：

- Firebase Project ID

この値はGitHubの `production` Environmentに `FIREBASE_PROJECT_ID` として登録します。

## 2. Firebase Hostingを有効化する

FirebaseコンソールでHostingを有効化します。

GitHub側では `firebase.json` により、Viteのビルド成果物 `dist/` を公開する設定になっています。

## 3. デプロイ用サービスアカウントを用意する

Google Cloud / Firebase側で、GitHub ActionsからFirebase Hostingへデプロイするためのサービスアカウントを用意します。

サービスアカウントのJSONキーを取得したら、そのJSON全文をGitHub Secretへ登録します。

重要：JSONキーはGitHubリポジトリの通常ファイル、README、Issue、チャットの公開部分などへ保存しないでください。

## 4. GitHubのproduction Environmentを用意する

GitHubリポジトリ `YOSHIAKIr06/project-of-STORE` を開きます。

Settings → Environments → New environment から `production` を作成します。

すでに `production` が存在する場合は、それを使用します。

## 5. Environment Variableを登録する

`production` EnvironmentのVariablesに次を登録します。

Name:

`FIREBASE_PROJECT_ID`

Value:

FirebaseのProject ID

## 6. Environment Secretを登録する

`production` EnvironmentのSecretsに次を登録します。

Name:

`FIREBASE_SERVICE_ACCOUNT`

Value:

サービスアカウントJSON全文

## 7. テストデプロイ

初回設定後、 `.deploy/deploy-request.json` を更新するとGitHub Actionsの `Firebase Production Deploy` が実行されます。

通常の `src/` や `public/` の変更だけではFirebase本番環境は更新されません。

## 8. デプロイ確認

GitHubのActions画面で `Firebase Production Deploy` を開きます。

確認項目：

- Build成功
- Firebase Hosting deploy成功
- 対象コミット
- 実行日時
- 実行ログ

## 日常運用

普段のコード変更：GitHubへ保存のみ。

本番公開するときだけ：`.deploy/deploy-request.json` を更新してデプロイ要求を記録。

これにより、コード変更履歴と本番公開履歴を分離して管理できます。
