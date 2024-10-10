# Cfn_pipeline_sample

以下の CodePipeline を作成するサンプルコードです。

- NetworkStack

## 初回作成時の手順

### 共通リソースの作成

CodePipeline で共通で使用する以下のリソースを作成する。

- サブスタック用の S3 Bucket

```bash
aws cloudformation create-stack \
  --stack-name PipelineNetworkStack-S3Stack \
  --template-body file://common/pipeline_s3.yaml \
  --capabilities CAPABILITY_IAM
```

- CodePipeline と CodeBuild の IAM role/policies

```bash
aws cloudformation create-stack \
  --stack-name PipelineNetworkStack-IamStack \
  --template-body file://common/pipeline_iam.yaml \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```

## Pipeline 作成の手順

### ダミースタック の作成

Pipeline 作成時に作成対象のスタックが無いとドリフト検知と変更セットのステージ作成でエラーになるため、予めダミーのリソースを同名のスタックで作成しておく。

```bash
aws cloudformation create-stack \
  --stack-name PipelineNetworkStack \
  --template-body file://dummy/dummy.yaml \
  --capabilities CAPABILITY_IAM
```

### Pipeline の作成

CICDスタックをデプロイする。

```bash
aws cloudformation create-stack \
  --stack-name PipelineNetworkStack-CicdStack \
  --template-body file://pipeline/pipeline_networkstack_sample.yaml \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```

## 削除

```bash
aws cloudformation delete-stack --stack-name PipelineNetworkStack-S3Stack
```

```bash
aws cloudformation delete-stack --stack-name PipelineNetworkStack-IamStack
```

```bash
aws cloudformation delete-stack --stack-name PipelineNetworkStack-CicdStack
```

注意点：

このテンプレートは既存のリソースを置き換えるものです。既存のリソースを保持したい場合は、テンプレートの適用前に必要な調整を行ってください。
CodeStarConnectionArn はパラメータとして定義されていますが、デフォルト値が設定されています。必要に応じて、このパラメータ値を変更してください。
S3 バケット名は固定されています。既存のバケットと競合しないように注意してください。
