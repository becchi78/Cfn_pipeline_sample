# Cfn_pipeline_sample

AWS CloudFormation を使用して CI/CD パイプラインを自動デプロイするためのサンプルコードです。
CodePipeline、CodeBuild、S3 Bucket、IAM リソースをデプロイします。

現在は以下のスタックをデプロイするコードが格納されています。

- インフラストラクチャースタック
  - NetworkStack

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
│   └── parameters_networkstack.json ・・・NetworkStackを作成するためのパラメータファイル
│
├── pipeline/
│   └── pipeline_infrastructure_sample.yaml ・・・インフラストラクチャスタックのCI/CDパイプラインを作成するためのテンプレート
└── README.md ・・・このREADME
```

## 前提条件

このコードを使用するための前提条件は以下の通りです。

- AWS アカウント
- AWS CLI
- Git
- 作成するリソースのテンプレートファイル

## セットアップ

1. このレポジトリをクローンします。

   ```bash
   git clone https://github.com/yourusername/Cfn_pipeline_sample.git
   cd Cfn_pipeline_sample
   ```

2. param/parameters_networkstack.json ファイルを編集し、必要なパラメータを設定します。
3. AWS CLI が正しく設定されていることを確認します：

   ```bash
   aws configure
   ```

## 初回作成時の手順

### 共通リソースの作成

CodePipeline で共通で使用する以下のリソースを作成する。

- サブスタック用の S3 Bucket

```bash
aws cloudformation deploy \
  --stack-name PipelineS3Stack \
  --template-file common/pipeline_s3.yaml \
  --capabilities CAPABILITY_IAM
```

- CodePipeline と CodeBuild の IAM role/policies

```bash
aws cloudformation deploy \
  --stack-name PipelineIamStack \
  --template-file common/pipeline_iam.yaml \
  --parameter-overrides file://param/parameters_common.json \
  --capabilities CAPABILITY_NAMED_IAM
```

## Pipeline 作成の手順

### ダミースタック の作成

Pipeline 作成時に作成対象のスタックが無いとドリフト検知と変更セットのステージ作成でエラーになるため、予めダミーのリソースを同名のスタックで作成しておく。

```bash
aws cloudformation deploy \
  --stack-name PipelineNetworkStack \
  --template-file dummy/dummy.yaml \
  --capabilities CAPABILITY_IAM
```

### Pipeline の作成

CICD スタックをデプロイする。

```bash
aws cloudformation deploy \
  --stack-name Pipeline-NetworkStack-CicdStack \
  --template-file pipeline/pipeline_infrastructure_sample.yaml \
  --parameter-overrides file://param/parameters_networkstack.json \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```

```bash
aws cloudformation deploy \
  --stack-name Pipeline-ContainerStack-CicdStack \
  --template-file pipeline/pipeline_container_sample.yaml \
  --parameter-overrides file://param/parameters_containerstack.json \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```

## 削除

リソースの削除時は以下のコマンドを実行します。

```bash
aws cloudformation delete-stack --stack-name PipelineS3Stack
```

```bash
aws cloudformation delete-stack --stack-name PipelineIamStack
```

```bash
aws cloudformation delete-stack --stack-name PipelineNetworkStack
```

```bash
aws cloudformation delete-stack --stack-name PipelineNetworkStack-CicdStack
```

```bash
aws cloudformation delete-stack --stack-name Pipeline-ContainerStack-CicdStack
```

## 注意点

1. S3 バケット名は固定されています。既存のバケットと競合しないように注意してください。
2. CodePipelineArtifactStoreName は自動的に`codepipeline-ap-northeast-1-691348252728`が使用されます。Codepipeline が未実行の場合はこの Bucket が存在しないので、あらかじめ作成する方がいいかもしれません。（勝手に作成されるかも）
