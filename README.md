# Cfn_pipeline_sample

以下の CodePipeline を作成するサンプルコードです。

- NetworkStack
- SecurityStack
- Ec2Stack

## 初回作成時の手順

### 共通リソースの作成

CodePipeline で共通で使用する以下のリソースを作成する。

- CodePipeline と CodeBuild の IAM role/policies
- サブスタック用と Artifact 用の S3

```bash
aws cloudformation create-stack \
  --stack-name PipelineCommmonResouceStack \
  --template-body file://common/common.yaml \
  --parameters file://param/parameters-common.json \
  --capabilities CAPABILITY_IAM
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

ダミーリソースと同名で目的のスタックをデプロイする。

```bash
aws cloudformation update-stack \
  --stack-name PipelineNetworkStack \
  --template-body file://pipeline/networkstack-pipeline-sample.yaml \
  --parameters file://param/parameters-common.json \
  --capabilities CAPABILITY_IAM
```

注意点：

このテンプレートは既存のリソースを置き換えるものです。既存のリソースを保持したい場合は、テンプレートの適用前に必要な調整を行ってください。
CodeStarConnectionArn はパラメータとして定義されていますが、デフォルト値が設定されています。必要に応じて、このパラメータ値を変更してください。
S3 バケット名は固定されています。既存のバケットと競合しないように注意してください。
