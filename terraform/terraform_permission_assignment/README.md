# Terraform用のGCPサービスアカウントの権限設定方法
※Terraformのv0.12.16バージョンを使っています。（この記事記載時点の最新バージョンです）

*本記事の目的
Terraform用のGCPサービスアカウント権限設定方法について各種のパターンをご紹介する*

Terraform初めての方はこの記事（[Terraformツールを使ってGCPリソース管理](https://qiita.com/devs_hd/items/6a715fedf5462af420f2)）もご覧ください。

Terraformを使っている時に、GCPにアクセス用のサービスアカウントはどのぐらいまでの権限を付与するのかよく考慮されると思っています。
選択肢は「基本の役割」（Primitive roles）、「事前定義された役割」（Predefined roles）、「カスタムの役割」（Custom roles）となっています。
どれはいいか正解回答はなく、プロジェクトの環境条件により選べられると思っています。


記事の流れ

- サービスアカウントを発行して、Terraform稼働環境に設定
- Primitive roles（基本の役割）の付与方法
- Predefined roles（事前定義された役割）の付与方法
- Custom roles（カスタムの役割）の付与方法


## 1. 　サービスアカウントを発行して、Terraform稼働環境に設定
サービスアカウントのaccount.jsonを準備して、配置すること。

#### 　Terraform専用のサービスアカウント作成
```sh
# set work project
gcloud config set project [PROJECT_ID]

# create service account
gcloud iam service-accounts create terraform-serviceaccount \
  --display-name "Account for Terraform"
```

#### 　サービスアカウントのCredentialファイルを作成して、Terraform稼働環境に設定
```sh
# create service account's credential file
gcloud iam service-accounts keys create {{path_to_save/account.json}} \
  --iam-account terraform-serviceaccount@[PROJECT_ID].iam.gserviceaccount.com

# set environment variable to use sevice account to access gcp
export GOOGLE_CLOUD_KEYFILE_JSON={{path_to_account.json}}
```


## 2. 　Primitive roles（基本の役割）の付与方法
この方法は一番簡単で、広い範囲の権限で設定する。通常は editorまたはownerのロールを付与する。

```sh
# ロールをサービスアカウトに付与。下記のコマンドを実施するため、オーナー権限必要
# editor権限付与
gcloud projects add-iam-policy-binding [PROJECT_ID] \
  --member serviceAccount:terraform-serviceaccount@[PROJECT_ID].iam.gserviceaccount.com \
  --role roles/editor

# またはowner権限付与
gcloud projects add-iam-policy-binding [PROJECT_ID] \
  --member serviceAccount:terraform-serviceaccount@[PROJECT_ID].iam.gserviceaccount.com \
  --role roles/owner
```

## 3. 　Predefined roles（事前定義された役割）の付与方法
GCP中に各種サービスごとの事前定義されたロールを個別付与する方法。

この方法を適用する場合、まずサービスアカウントユーザー（Service Account User）ロールを付与する必要です。
それから必要なサービスを個別に付与する。
例えば、クラウドストレージを利用する場合、「storage.admin」ロールを付与します。

```sh
# サービスアカウントユーザー権限付与。下記のコマンドを実施するため、オーナー権限必要
gcloud projects add-iam-policy-binding [PROJECT_ID] \
  --member serviceAccount:terraform-serviceaccount@[PROJECT_ID].iam.gserviceaccount.com \
  --role roles/iam.serviceAccountUser

# 「storage.admin」ロール権限付与
gcloud projects add-iam-policy-binding [PROJECT_ID] \
  --member serviceAccount:terraform-serviceaccount@[PROJECT_ID].iam.gserviceaccount.com \
  --role roles/storage.admin
```

各種サービスごとの事前定義されたロールはここでご参考
[https://cloud.google.com/iam/docs/understanding-roles](https://cloud.google.com/iam/docs/understanding-roles)


## 4. 　Custom roles（カスタムの役割）の付与方法
事前定義された役割の付与方法より、もっと細かい権限を設定する方法です。
カスタムの役割を定義して、必要な権限のみを付与する。
新しいリソースが追加される場合、そのリソースのサービス用の権限はまだなければ、
ロール定義ファイルに追加して、カスタムの役割を更新する必要です。

必要な権限を "_role/terraform_account_role.yaml"のファイルをまとめます。

```sh
title: "Terraform Account Role"
description: "Terraform Script Running Script Role"
stage: "ALPHA"
includedPermissions:
- iam.serviceAccounts.actAs
- iam.serviceAccounts.get
- iam.serviceAccounts.list
- resourcemanager.projects.get
- resourcemanager.projects.list
- storage.buckets.create
- storage.buckets.update
- storage.buckets.delete
- storage.buckets.list
- storage.buckets.get
- storage.objects.list
- storage.objects.get
```

この定義はサービスアカウントユーザー（Service Account User）権限とストレージのバケツ作成とobjects参照権限があるロールの定義です。
カスタムの役割を作成実施

```sh
# カスタムの役割を作成実施。下記のコマンドを実施するため、オーナー権限必要
gcloud iam roles create terraform_role --project [PROJECT_ID] \
--file ./_role/terraform_account_role.yaml

# 作成できたカスタムの役割を確認
gcloud iam roles describe terraform_role --project [PROJECT_ID]
```


権限を修正・追加する場合、カスタムの役割の更新を実行する必要です。

```sh
# カスタムの役割を更新実施。下記のコマンドを実施するため、オーナー権限必要
gcloud iam roles update terraform_role --project [PROJECT_ID] \
--file ./_role/terraform_account_role.yaml
```

    

本記事の利用ソースコードはこちら
[https://github.com/itdevsamurai/gcp/tree/master/terraform/terraform_permission_assignment](https://github.com/itdevsamurai/gcp/tree/master/terraform/terraform_permission_assignment)


最後まで読んで頂き、どうも有難う御座います!
DevSamurai 橋本


関連記事：
[Terraformツールを使ってGCPリソース管理](https://qiita.com/devs_hd/items/6a715fedf5462af420f2)
[Terraformスクリプトをモジュール化して、GCPの複数環境に適用](https://qiita.com/devs_hd/items/491b72dec2d4c077d977)