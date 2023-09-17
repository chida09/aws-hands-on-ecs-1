# aws-hands-on-ecs-1
[Amazon ECS 入門ハンズオン](https://catalog.us-east-1.prod.workshops.aws/workshops/7ffc4ed9-d4b3-44dc-bade-676162b427cd/ja-JP)

## 作業環境の作成
1. Local 環境のパブリック IP アドレスを調べる
2. Dockerfileの作成(既にリポジトリ内に作成済みのDockerfileを使用する)
3. コンテナイメージの作成(イメージ作成時に[hello-world]タグを付与する)
4. コンテナイメージが新たに作成されたか確認する 
5. コンテナを動かす(コンテナをバックグラウンドで動かす。ホスト側を8080portにする。コンテナに名前[h4b-local-run]をつける)
6. コンテナの起動を確認する
7. curlでリクエストを投げて[Hello World!]を確認する
