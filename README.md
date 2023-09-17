# aws-hands-on-ecs-1
[Amazon ECS 入門ハンズオン](https://catalog.us-east-1.prod.workshops.aws/workshops/7ffc4ed9-d4b3-44dc-bade-676162b427cd/ja-JP)

# 手順
* Cloud9を使わずにLocal環境から実行する手順
* Local環境から実行するため、ハンズオンにはIAMのことは書かれていない

## 作業環境の作成
Dockerイメージを作成する
1. Local 環境のパブリック IP アドレスを調べる
2. Dockerfileの作成(既にリポジトリ内に作成済みのDockerfileを使用する)
3. コンテナイメージの作成(イメージ作成時に[hello-world]タグを付与する)
4. コンテナイメージが新たに作成されたか確認する 
5. コンテナを動かす(コンテナをバックグラウンドで動かす。ホスト側を8080portにする。コンテナに名前[h4b-local-run]をつける)
6. コンテナの起動を確認する
7. curlでリクエストを投げて[Hello World!]を確認する

## IAM ユーザーの作成と確認
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
aws ecr get-login-password {option}
```
[参考リンク](https://docs.aws.amazon.com/ja_jp/AmazonECR/latest/userguide/registry_auth.html)

## ECR にアップロード
作成したコンテナイメージを、コンテナレジストリのフルマネージドサービスである Amazon Elastic Container Registry (Amazon ECR) にアップロード(Push)します。
1. AWSマネージメントコンソールのECRで、リポジトリ名[h4b-ecs-helloworld]を入力し作成する
2. 作成したコンテナイメージをリポジトリにアップロードするために、dockerコマンドでTag[0.0.1]を付与する(hint: `ホスト名/イメージ名[:タグ名]`)
3. Tagが付与されたか、コンテナイメージ一覧を確認する 
4. コンテナイメージをリポジトリにアップロード(push)するために、docker loginを行う(hint: `aws ecr ... | docker login ... ホスト名`) 
5. イメージをアップロード(push)する
6. AWSマネージメントコンソールでECRの画面を開き、イメージの情報があることを確認する

## ECS の作成
### VPCの作成
* VPCを[VPC など]から名前タグ[h4b-ecs]で、プライベートサブネットの数は[0]で作成する
* セキュリティを選択し、インバウンドルールの編集を押下する
* ハンズオンで操作している端末のIPアドレスからのみアクセスできる制限を[HTTP]に入れる

### ECS クラスターの作成 
