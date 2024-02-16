# lambda_message-notification

## 概要
* CloudWatch Logsのサブスクリプションフィルターで検知したエラーログを、BacklogとSlackへ連携通知するLambdaを登録するCloudFormationテンプレート

## CloudFormation内のパラメーターの説明
* LambdaFunctionName：登録するLambdaのFunction名
* LabmdaRoleArn：Lambdaを実行するIAMロールのArn
* SlackWebhookUrl：通知先のSlack WebhookのURL
* SlackChannelName：通知先のSlackのチャンネル名
* BacklogMail：Backlogのインテグレーションで自動発行された課題登録用メールアドレス
* FromMail：Backlogユーザーのメールアドレス。このメールアドレスのユーザーが課題登録者として設定される
* BacklogTicketTitle：Backlogに自動登録する課題のタイトル

## Lambdaの機能説明
* CloudWatch Logsのサブスクリプションフィルターで検知したイベントを受け取り、検知したログメッセージの内容をSlackとBacklogへ送信する


## 設定手順
### Backlog側の設定
* ※実行にはBacklogプロジェクトの管理者権限が必要となります

1. Backlogプロジェクトのページを開く
1. 左メニューのプロジェクト設定＞インテグレーション＞「メールによる課題登録」＞「設定」　から設定を行う
1. 「課題登録用メールアドレスの追加」を押下し、各設定を行うと課題登録専用のメールアドレスが発行される

### Slack側の設定
1. https://api.slack.com/apps にアクセスして、「Create an APP(Create New App)」をクリックしてSlackアプリを作成を開始する
1. 「From Scratch」を選択
1. 「App Name」と対象のワークスペースを設定
1. 作成したSlackアプリの内容設定を行う
    1. 「Add features and functionaly」で「Incoming Webhooks」を選択する
    1. 「Activate Incoming Webhooks」をONにする
1. 通知botの設定を行う
    1. 左メニューの「App Home」 を選択
    1. 「App Display Name」の「Edit」を押下
    1. 「Display Name (Bot Name)」「Default username」に通知botに設定する名前を入力して、「Add」を実行
1. 左メニューの「Incoming Webhook」 を選択
    1. 「Add New Webhooks to Workspace」を押下
    1. アラート通知先のチャンネル名を選択して「許可する」を実行
1. 動作テストの実行
    1. 「Sample curl request to post to a channel:」のテキストをCopyする
    1. ※Windowsの場合は'（シングルクォートが）無効になるので、"（ダブルクォート）に書き換える
    * 例）`curl -X POST -H "Content-type: application/json" --data "{\"text\":\"Test Alert message!\"}" <WebhookのURL>`

### AWS：lambda_message-notificationの実行に必要な事前設定
* AWSコンソールにログインして以下の設定を行う

1. SES設定
    1. 「Amazon SES」のサービスを開く
    1. 「IDの作成」からBacklog課題登録用メールの設定を行う
    1. IDの詳細を以下の内容で設定
        1. IDタイプに「Eメールアドレス」を選択する
        1. 「Eメールアドレス」にBacklog側の設定手順で作成したBacklog課題登録用メールを入力する
        1. 「IDの作成」を押下
    1. 続いて再度「IDの作成」からBacklog登録ユーザーのメールの設定を行う
        1. IDタイプに「Eメールアドレス」を選択する
        1. 「Eメールアドレス」にBacklogプロジェクトに登録済みユーザーのメールアドレスを入力する（このユーザーが課題登録者として設定されます）
        1. 「IDの作成」を押下
    1. 「Amazon Web Services – Email Address Verification Request in region Asia Pacific (Tokyo)」というメールが送信されてくるので、リンク先をクリックして検証を完了させる
        * 宛先がBacklog課題登録用メールだと初回は検証確認メールの内容がBacklog課題に登録されるので、そのリンクから検証を完了させる
        * 検証が完了すると「IDステータス」が「検証済み」になる
1. IAMロール設定
    1. 「IAM」のサービスを開く
    1. Lambda実行ロールにSES使用のために以下のポリシーを追加する
        * AWS管理ポリシー：CloudWatchFullAccess
        * カスタムポリシーに以下を追加
            * Action： `"ses:SendEmail", "ses:SendRawEmail"`を追加
            * Resource：作成した2つのSESのarnを追加する（Backlog課題登録用メールとBacklog登録ユーザーのメール）

### AWS：Lambdaの登録
* AWSコンソール上で以下の設定を行う
1. 「CloudFormation」のサービスを開く
1. 「スタックの作成」を押下して「新しいリソース使用（標準）」を選択する
1. 「テンプレートの準備」に「テンプレートの準備完了」、「テンプレートの指定」に「テンプレートファイルのアップロード」を選択する
1. 「ファイルの選択」からlambda_message-notification.ymlをアップロードする
1. 「スタックの詳細を指定」でパラメータを入力して、CloudFormationスタックの作成まで行う

### AWS：CloudWatchの設定
1. ロググループにサブスクリプションフィルターを設定
    1. 「CloudWatch」のサービスを開く
    1. 左メニュー「ログ」＞ロググループ　でロググループの一覧を表示する
    1. 検知対象にしたいロググループ名のチェックボックスにチェックを入れる
    1. 「アクション」を押下して「Lambdaサブスクリプションフィルターを作成」を選択する
    1. 「Lambda関数」に作成したLambdaを選択する
    1. 「ログの形式」に「その他」を選択する
    1. 「サブスクリプションフィルターのパターン」にエラー発生対象として検知したい文言を設定する（例えば"ERROR"や"Exception"など）
    1. 「サブスクリプションフィルター名」に任意の名前を入力する
    1. 「ストリーミングを開始」を押下
