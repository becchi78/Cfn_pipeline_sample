# Cfn_pipeline_sample

AWS CloudFormation を使用して CI/CD パイプラインを自動デプロイするためのサンプルコードです。
CodePipeline、CodeBuild、S3 Bucket、IAM リソースをデプロイします。

現在は以下のスタックをデプロイするコードが格納されています。

- インフラ
  - ネットワークスタック
- コンテナスタック

## 構成

```bash
Cfn_pipeline_sample/
│
├── common/
│   │── pipeline_iam_inline.yaml ・・・パイプライン実行に必要なIAMを作成するテンプレート（インラインで詳細に記述した版）
│   │── pipeline_iam.yaml ・・・パイプライン実行に必要なIAMを作成するテンプレート（AWS管理ポリシーを使用した版）
│   └── pipeline_s3.yaml ・・・パイプライン実行に必要なS3を作成するテンプレート
│
├── dummy/
│   └── dummy.yaml ・・・ダミーリソースを作成するテンプレート
│
├── param/
│   │── parameters_common.json ・・・pipeline_iam.yamlで使用するパラメータファイル
│   │── parameters_containerstack.json ・・・コンテナスタックを作成するためのパラメータファイル
│   └── parameters_networkstack.json ・・・ネットワークスタックを作成するためのパラメータファイル
│
├── pipeline/
│   │── pipeline_container_sample.yaml ・・・コンテナスタックのCI/CDパイプラインを作成するためのテンプレート
│   └── pipeline_infrastructure_sample.yaml ・・・インフラ（ネットワークスタックやIAMスタックなど）のCI/CDパイプラインを作成するためのテンプレート
└── README.md ・・・このREADME
```

(補足)
pipeline_infrastructure_sample.yaml にネットワークスタックやセキュリティスタックのパラメータファイル（json）を与えることで、サーバレススタックやコンテナスタック以外のパイプラインを作成することができます。

## 前提条件

このコードを使用するための前提条件は以下の通りです。

### 全般

- AWS アカウント
- AWS CLI
- Git

### インフラ

- 作成するスタックのテンプレートファイル

### コンテナスタック

- コンテナビルド用のファイル一式
  - Dockerfile
  - buildspec.yml
  - アプリケーションコード
  - その他
- ECS
  - クラスター
  - タスク定義
  - サービス

## 初回作成時の手順

CodePipeline で共通で使用するリソースを作成します。

### セットアップ

1. このレポジトリをクローンします。

   ```bash
   git clone https://github.com/yourusername/Cfn_pipeline_sample.git
   cd Cfn_pipeline_sample
   ```

2. param ディレクトリ下にパラメータファイルを編集し、必要なパラメータを設定します。
3. AWS CLI が正しく設定されていることを確認します。

   ```bash
   aws configure
   ```

### サブスタック用の S3 Bucket 作成

```bash
aws cloudformation deploy \
  --stack-name PipelineS3Stack \
  --template-file common/pipeline_s3.yaml \
  --capabilities CAPABILITY_IAM
```

### CodePipeline と CodeBuild が使用する IAM role/policies 作成

```bash
aws cloudformation deploy \
  --stack-name PipelineIamStack \
  --template-file common/pipeline_iam.yaml \
  --parameter-overrides file://param/parameters_common.json \
  --capabilities CAPABILITY_NAMED_IAM
```

## インフラ（ネットワークスタック、セキュリティスタック）パイプライン

### ダミースタック の作成

パイプライン作成時に作成対象のスタックが無いとドリフト検知と変更セットのステージ作成でエラーになるため、予めダミーのリソースを同名のスタックで作成しておきます。
既にスタックを作成している場合にはこの手順は必要ありません。

```bash
aws cloudformation deploy \
  --stack-name PipelineNetworkStack \
  --template-file dummy/dummy.yaml \
  --capabilities CAPABILITY_IAM
```

### パイプラインの作成

以下のコマンドでインフラパイプラインスタックを作成します。（ネットワークスタックの場合）

```bash
aws cloudformation deploy \
  --stack-name Pipeline-NetworkStack-CicdStack \
  --template-file pipeline/pipeline_infrastructure_sample.yaml \
  --parameter-overrides file://param/parameters_networkstack.json \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```

## コンテナパイプラインスタック

以下のコマンドでコンテナパイプラインスタックを作成します。

```bash
aws cloudformation deploy \
  --stack-name Pipeline-ContainerStack-CicdStack \
  --template-file pipeline/pipeline_container_sample.yaml \
  --parameter-overrides file://param/parameters_containerstack.json \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```

## 削除

スタックの削除時は以下のコマンドを実行します。

### PipelineS3Stack

```bash
aws cloudformation delete-stack --stack-name PipelineS3Stack
```

### PipelineIamStack

```bash
aws cloudformation delete-stack --stack-name PipelineIamStack
```

### PipelineNetworkStack

```bash
aws cloudformation delete-stack --stack-name PipelineNetworkStack
```

### PipelineNetworkStack-CicdStack

```bash
aws cloudformation delete-stack --stack-name PipelineNetworkStack-CicdStack
```

### Pipeline-ContainerStack-CicdStack

```bash
aws cloudformation delete-stack --stack-name Pipeline-ContainerStack-CicdStack
```

## 注意点

1. S3 バケット名は固定されています。既存のバケットと競合しないように注意してください。
2. CodePipelineArtifactStoreName は自動的に`codepipeline-ap-northeast-1-<アカウントID>`が使用されます。Codepipeline が未実行の場合はこの Bucket が存在しないので、あらかじめ作成する方がいいかもしれません。（勝手に作成されるかも）
