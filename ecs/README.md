# log-notification-cloud-formation
ログ出力設定のCfn化

# 各テンプレートの説明
## ecr.yml
ECRが構築される。

## log-notification.yml
ECS環境が構築され、CloudWatchへログが出力されるように設定される。

# CloudFormation内のパラメータの説明
## ecr.yml
- RepositoryName: 作成するリポジトリの名前

## log-notification.yml
- VpcId: 使用するVPC
- ALBPort: ALBリスナーのポート
- ALBSecurityGroupId: ALBに関連付けるセキュリティグループ
- ALBSubnetIds: ALBに関連付けるサブネット(2つ以上指定が必要)
- CloudWatchLogGroupMultiLinePattern: CloudWatch Logsで、複数行を単一イベントとして処理するための正規表現を定義
- ECSImageName: ECSで使用するイメージ名
- ECSContainerPort: ECSコンテナの公開ポート
- ECSTaskCPUUnit: ECSタスクに適用されるCPUユニットのハード制限
- ECSTaskMemory: ECSタスクに適用されるメモリユニットのハード制限
- ECSTaskCpuArchitecture: ECSタスクに適用されるCPUのアーキテクチャ
- ECSTaskOperatingSystemFamily: ECSタスクに適用されるOS
- ECSServiceDesiredCount: サービス内のインスタンス数
- ECSServiceSecurityGroupId: ECSサービスに関連付けるセキュリティグループ
- ECSServiceSubnetId: ECSサービスに関連付けるサブネット

※定義されているパラメータの候補は2023/12/01時点のものです。

# 設定手順
- AWSコンソールにログインして以下の設定を行う

1. ECRの設定  
    1. 「CloudFormation」のサービスを開く
    1. 「テンプレートファイルのアップロード」を選択し、ecr.ymlを選択、パラメータを入力する
    1. 作成したECRへ、コンテナイメージをアップロードする
1. ECS, ALBの設定
    1. 「CloudFormation」のサービスを開く
    1. 「テンプレートファイルのアップロード」を選択し、log-notification.ymlを選択、パラメータを入力する
