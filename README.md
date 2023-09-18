# aws-hands-on-ecs-1
* [Amazon ECS 入門ハンズオン](https://catalog.us-east-1.prod.workshops.aws/workshops/7ffc4ed9-d4b3-44dc-bade-676162b427cd/ja-JP)

# 手順
* Cloud9を使わずにLocal環境から実行する手順
* Local環境から実行するため、ハンズオンにはIAMのことは書かれていない
* {mm-dd}は日付を入れる

## 作業環境の作成
Dockerイメージを作成する
1. Local 環境のパブリック IP アドレスを調べる
2. Dockerfileの作成(既にリポジトリ内に作成済みのDockerfileを使用する)
3. コンテナイメージの作成(イメージ作成時に[hello-world-{mm-dd}-v{version}]タグを付与する)
4. コンテナイメージが新たに作成されたか確認する 
5. コンテナを動かす
 * コンテナをバックグラウンドで動かすために[デタッチモード]にする
 * ホスト側をport[8080]にする
 * プラットフォームは[linux/amd64]で動かす
 * コンテナに名前[h4b-local-run-{mm-dd}-v{version}]をつける
6. コンテナの起動を確認する
7. curlでリクエストを投げて[Hello World!]を確認する

## IAM ユーザーの作成と確認(初回のみ)
1.作成
```
aws configure --profile aws-hands-on-user
```

2.確認
```
cat ~/.aws/credentials
cat ~/.aws/config
```

3.CLI を使用して Amazon ECR プライベートレジストリに対して Docker を認証するコマンドを確認する
```
aws ecr get-login-password --profile {name}
```
[参考リンク](https://docs.aws.amazon.com/ja_jp/AmazonECR/latest/userguide/registry_auth.html)

## ECR にアップロード
作成したコンテナイメージを、コンテナレジストリのフルマネージドサービスである Amazon Elastic Container Registry (Amazon ECR) にアップロード(Push)します。
1. AWSマネージメントコンソールのECRで、リポジトリ名[h4b-ecs-helloworld-{mm-dd}-v{version}]を入力し作成する
2. 作成したコンテナイメージをリポジトリにアップロードするために、dockerコマンドでTag[0.0.1]を付与する(hint: `ホスト名/イメージ名[:タグ名]`)
3. Tagが付与されたか、コンテナイメージ一覧を確認する 
4. コンテナイメージをリポジトリにアップロード(push)するために、docker loginを行う(hint: `aws ecr ... --profile ... | docker login ... ホスト名`) 
5. イメージをアップロード(push)する
6. AWSマネージメントコンソールでECRの画面を開き、イメージの情報があることを確認する

## ECS の作成
### VPCの作成
* VPCを[VPC など]から名前タグ[h4b-ecs-{mm-dd}-v{version}]で、プライベートサブネットの数は[0]にする
* VPCを作成するを押下する
* セキュリティグループを選択し、インバウンドルールの編集を押下する
* ハンズオンで操作している端末のIPアドレスからのみアクセスできる制限を[HTTP]に入れる

### ECS クラスターの作成 
ECS クラスターは、コンテナを動かすための論理的なグループです。ECS を操作するために、まずは ECS クラスターを作成していきます。
* クラスター名[h4b-ecs-cluster-{mm-dd}-v{version}]で作成する 

### タスクの作成
* サイドバーのタスク定義から新しいタスク定義を作成する
* タスク定義ファミリー	h4b-ecs-task-definition-{mm-dd}-v{version}

コンテナ - 1 の作成
```
名前	apache-helloworld-{mm-dd}-v{version}
コンテナポート	80
イメージ URI	<要変更>.dkr.ecr.ap-northeast-1.amazonaws.com/h4b-ecs-helloworld-{mm-dd}-v{version}:0.0.1
プロトコル	TCP
ポート名 apache-helloworld-{mm-dd}-v{version}-80-tcp
ログ収集の使用	オフにする
```

### サービスの作成
サービスとは、ECS 上でコンテナを動かすときに利用する概念の一つです。
サービスを使用すると、 Amazon ECS クラスターで、指定した数のコンテナ群(タスク)を維持できます。
```
アプリケーションタイプ	サービス
ファミリー	h4b-ecs-task-definition-{mm-dd}-v{version}
リビジョン	1 (最新)
サービス名	h4b-ecs-service-{mm-dd}-v{version}
サービスタイプ	レプリカ
必要なタスク	2
ネットワーキング	h4b-ecs-vpc-{mm-dd}-v{version}
```

次に、ロードバランサーに関するパラメータを入れます。
```
ロードバランサーの種類	Application Load Balancer
Application Load Balancer	新しいロードバランサーの作成
ロードバランサー名	h4b-ecs-alb-{mm-dd}-v{version}
ポート	80
プロトコル	HTTP
ターゲットグループ名	h4b-ecs-targetgroup-{mm-dd}-v{version}
プロトコル	HTTP
```

### 起動失敗時

Amazon Elastic Container Service > クラスター > h4b-ecs-cluster > タスク > c3da67f8bd8648fd8c257387d2271988 > 設定で`Essential container in task exited`と出ていた。

```
❯ aws ecs list-tasks \
  --profile aws-hands-on-user \
  --cluster h4b-ecs-cluster \
  --desired-status STOPPED
```

```
❯ aws ecs describe-tasks \
  --profile aws-hands-on-user \
  --cluster h4b-ecs-cluster \
  --tasks arn:aws:ecs:ap-northeast-1:980591565011:task/h4b-ecs-cluster/faa2cb00dd3649399a27f90ef250b8ae > ~/Desktop/ecs-error.json
```
[参考情報](https://codenote.net/aws/5384.html)
  
結論、docker buildする際に--platformオプションでCPUアーキテクチャを指定するかDockerfileのFROMでCPUアーキテクチャを指定することでビルド時に指定のCPUアーキテクチャでイメージを作成できた。


### インターネットからアクセス
* ブラウザでHello World! と表示されれば、正しくアクセスが出来ています。


# 削除
[削除](https://catalog.us-east-1.prod.workshops.aws/workshops/7ffc4ed9-d4b3-44dc-bade-676162b427cd/ja-JP/7-delete)
