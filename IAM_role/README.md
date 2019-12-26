# GCPのサービスを利用権限のまとめ


*本記事の目的
GCPのIAMロールを理解しづらいだったため、自分の理解を整理する*

GCPのサービス利用権限はIAMロールで決められる。
個別アカウントにロールを付与して、アクセス権限を管理する。


IAMロールは基本３つタイプがあります。

- Primitive roles（基本の役割）
- Predefined roles（事前定義された役割）
- Custom roles（カスタムの役割）

各種のロールと付与方法は下記でまとめます。


## 1. 　Primitive roles（基本の役割）
基本の役割を使ってアクセス権限管理は一番簡単です。
広い範囲の権限を付与することとなるので、広い範囲のユーザーに付与するのはあまりオススメない。

アカウントにロール付与方法

```sh
# 下記のコマンドを実施するため、オーナー権限必要
# 例：editor権限付与
gcloud projects add-iam-policy-binding [PROJECT_ID] \
  --member user:[USER_EMAIL] \
  --role roles/editor

# またはowner権限付与
gcloud projects add-iam-policy-binding [PROJECT_ID] \
  --member user:[USER_EMAIL] \
  --role roles/owner
```

## 2. 　Predefined roles（事前定義された役割）
GCP中に各種サービスごとの事前定義されたロールを個別付与する方法。

必要なサービスを個別に付与する。
例えば、クラウドストレージを利用する場合、「storage.admin」ロールを付与します。

```sh
# storage.admin」ロール権限付与。下記のコマンドを実施するため、オーナー権限必要
gcloud projects add-iam-policy-binding [PROJECT_ID] \
  --member user:[USER_EMAIL] \
  --role roles/storage.admin
```

各種サービスごとの事前定義されたロールはここでご参考
[https://cloud.google.com/iam/docs/understanding-roles](https://cloud.google.com/iam/docs/understanding-roles)


## 3. 　Custom roles（カスタムの役割）
事前定義された役割の付与方法より、もっと細かい権限設定できます。
カスタムの役割を定義して、必要な権限のみを付与する。
権限追加は必要の時、ロール定義ファイルに追加して、カスタムの役割を更新する必要です。

必要な権限を "custom_role_define.yaml"のファイルをまとめます。

```sh
title: "My Custom Role"
description: "For Test User"
stage: "ALPHA"
includedPermissions:
- storage.buckets.create
- storage.buckets.update
- storage.buckets.delete
- storage.buckets.list
- storage.buckets.get
- storage.objects.list
- storage.objects.get
```

この定義はストレージのバケツ作成とobjects参照権限があるロールの定義です。
カスタムの役割を作成実施

```sh
# カスタムの役割を作成実施。下記のコマンドを実施するため、オーナー権限必要
gcloud iam roles create my_custom_role --project [PROJECT_ID] \
--file custom_role_define.yaml

# 作成できたカスタムの役割を確認
gcloud iam roles describe my_custom_role --project [PROJECT_ID]
```


権限を修正・追加する場合、カスタムの役割の更新を実行する必要です。

```sh
# カスタムの役割を更新実施。下記のコマンドを実施するため、オーナー権限必要
gcloud iam roles update my_custom_role --project [PROJECT_ID] \
--file custom_role_define.yaml
```

それから、ユーザーアカウントにロール付与

```sh
gcloud projects add-iam-policy-binding [PROJECT_ID] \
  --member user:[USER_EMAIL] \
  --role projects/[PROJECT_ID]/roles/my_custom_role
```

<br>
ご覧して頂き、どうも有難う御座います!
DevSamurai Ben
