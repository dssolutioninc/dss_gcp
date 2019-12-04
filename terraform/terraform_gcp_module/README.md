# [Terraform](https://www.terraform.io/)スクリプトをモジュール化して、GCPの複数環境に適用
※Terraformのv0.12.16バージョンを使っています。（この記事記載時点の最新バージョンです）

本記事の目的
・Terraformスクリプトをモジュール化して、GCPの開発環境、テスト環境、本番環境に適用する方法のご紹介

Terraformは初めての方はこの記事（[Terraformツールを使ってGCPリソース管理](https://qiita.com/devs_hd/items/6a715fedf5462af420f2)）もご覧ください。


## 1.　　Terraformスクリプト作成
この記事では、GKEクラスタ、ストレージBucket、Pubsub Topic＆Subscriptionを例としてデプロイスクリプトを作成します。

Terraformスクリプトフォルダの構成

```sh
terraform_script_folder
├── _modules
│   ├── cluster
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   ├── pubsub
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   └── storage
│       ├── main.tf
│       ├── outputs.tf
│       └── variables.tf
├── dev
│   ├── account.json
│   └── terraform.tfstate
├── dev.tfvars
├── main.tf
├── prod
│   ├── account.json
│   └── terraform.tfstate
├── prod.tfvars
├── staging
│   ├── account.json
│   └── terraform.tfstate
├── staging.tfvars
└── variables.tf
```

フォルダ構成の説明

- _modulesフォルダ：リソース種類のごとに共有定義スクリプトを格納する
- dev、staging、prodのフォルダ：開発、ステージング、本番の環境用にアクセス用のアカウントファイルとStateファイルを格納
- dev.tfvars、staging.tfvars、prod.tfvarsのファイル：各種環境によるパラメータ設定ファイル


## 2.　　Terraformスクリプト作成
#### 2.1　　共有モジュールスクリプト（_modules）

_modules/storage/main.tf

```sh
resource "google_storage_bucket" "sample-bucket" {
  name          = var.name
  location      = var.region
  storage_class = var.storage_class

  labels = {
    app = var.app
    env = var.env
  }
}
```

_modules/storage/variables.tf

```sh
resource "google_storage_bucket" "sample-bucket" {
  name          = var.name
  location      = var.region
  storage_class = var.storage_class

  labels = {
    app = var.app
    env = var.env
  }
}
```

#### 2.2　　メインスクリプト

main.tf

```sh
terraform {
  required_version = "~>0.12.14"
}

## project ##
provider "google" {
  project     = var.project
  region      = var.region
}

## storage buckets ##
module "storage" {
  source = "./_modules/storage"

  name   = var.storage_name
  region = var.region
  app    = var.app
  env    = var.env
}

## cluster ##
module "cluster" {
  source = "./_modules/cluster"

  name  = var.cluster_name
  zone  = var.zone
  node_count = var.node_count
  initial_node_count = var.initial_node_count
  node_pool_name = var.node_pool_name
  node_machine_type = var.node_machine_type
}

## topic & subscription ##
## the first one
module "the-first-topic" {
  source = "./_modules/pubsub"

  topic_name  = var.first_topic_name
  subscription_name  = var.first_subscription_name

  app    = var.app
  env    = var.env
}

## the second one
module "the-second-topic" {
  source = "./_modules/pubsub"

  topic_name  = var.second_topic_name
  subscription_name  = var.second_subscription_name

  app    = var.app
  env    = var.env
}
```

variables.tf

```sh
## global variables
variable "project" {}
variable "app" {}
variable "env" {}
variable "region" {
  default = "asia-northeast1"
}
variable "zone" {
  default = "asia-northeast1-b"
}

## storage variables
variable "storage_name" {}

## cluster variables
variable "cluster_name" {}
variable "node_count" {}
variable "initial_node_count" {}
variable "node_pool_name" {}
variable "node_machine_type" {}

## pubsub variables
variable "first_topic_name" {}
variable "first_subscription_name" {}
variable "second_topic_name" {}
variable "second_subscription_name" {}
```

#### 2.3　　環境ごとパラメータファイル

開発環境（dev.tfvars）

```sh
## global variables
project = "project-dev"
env 	= "dev"
app 	= "sample-app"
region 	= "asia-northeast1"
zone 	= "asia-northeast1-b"


## storage variables
storage_name = "dev_private_bucket_abc123"

## cluster variables
cluster_name        = "sample-cluster"
initial_node_count  = 1
node_count          = 2
node_pool_name      = "sample-node-pool"
node_machine_type   = "n1-standard-1"

## pubsub variables
first_topic_name            = "sample-first-topic"
first_subscription_name     = "sample-first-topic-sub"
second_topic_name           = "sample-second-topic"
second_subscription_name    = "sample-second-topic-sub"
```

ステージング環境（staging.tfvars）

```sh
## global variables
project = "project-staging"
env 	= "staging"
app 	= "sample-app"
region 	= "asia-northeast1"
zone 	= "asia-northeast1-b"


## storage variables
storage_name = "staging_private_bucket_abc123"

## cluster variables
cluster_name        = "sample-cluster"
initial_node_count  = 1
node_count          = 2
node_pool_name      = "sample-node-pool"
node_machine_type   = "n1-standard-1"

## pubsub variables
first_topic_name            = "sample-first-topic"
first_subscription_name     = "sample-first-topic-sub"
second_topic_name           = "sample-second-topic"
second_subscription_name    = "sample-second-topic-sub"
```

本番環境（prod.tfvars）

```sh
## global variables
project = "project-prod"
env 	= "prod"
app 	= "sample-app"
region 	= "asia-northeast1"
zone 	= "asia-northeast1-b"


## storage variables
storage_name = "prod_private_bucket_abc123"

## cluster variables
cluster_name        = "sample-cluster"
initial_node_count  = 1
node_count          = 2
node_pool_name      = "sample-node-pool"
node_machine_type   = "n1-standard-1"

## pubsub variables
first_topic_name            = "sample-first-topic"
first_subscription_name     = "sample-first-topic-sub"
second_topic_name           = "sample-second-topic"
second_subscription_name    = "sample-second-topic-sub"
```

## 3.　　環境別にデプロイ実施
#### 3.1　　開発環境

```sh
# 専用の環境変数にCredentialファイルを設定する
$ export GOOGLE_CLOUD_KEYFILE_JSON=path_to/dev/account.json

# tfファイルを適用する前に必ず差分を確認する
cd [TERRAFORM_FOLDER]
terraform plan -var-file="dev.tfvars" -state=./dev/terraform.tfstate

# planの結果が想定通りなら、tfファイルを適用する
terraform apply -var-file="dev.tfvars" -state=./dev/terraform.tfstate
```

#### 3.2  ステージング環境

```sh
# 専用の環境変数にCredentialファイルを設定する
$ export GOOGLE_CLOUD_KEYFILE_JSON=path_to/staging/account.json

# tfファイルを適用する前に必ず差分を確認する
cd [TERRAFORM_FOLDER]
terraform plan -var-file="staging.tfvars" -state=./staging/terraform.tfstate

# planの結果が想定通りなら、tfファイルを適用する
terraform apply -var-file="staging.tfvars" -state=./staging/terraform.tfstate
```

#### 3.3  本番環境

```sh
# 専用の環境変数にCredentialファイルを設定する
$ export GOOGLE_CLOUD_KEYFILE_JSON=path_to/prod/account.json

# tfファイルを適用する前に必ず差分を確認する
cd [TERRAFORM_FOLDER]
terraform plan -var-file="staging.tfvars" -state=./prod/terraform.tfstate

# planの結果が想定通りなら、tfファイルを適用する
terraform apply -var-file="staging.tfvars" -state=./prod/terraform.tfstate
```

本記事の利用ソースコードはこちら
[https://github.com/itdevsamurai/gke/tree/master/terraform_gcp_module](https://github.com/itdevsamurai/gke/tree/master/terraform_gcp_module)


最後まで読んで頂き、どうも有難う御座います!
DevSamurai 橋本


関連記事：[Terraformツールを使ってGCPリソース管理](https://qiita.com/devs_hd/items/6a715fedf5462af420f2)