# docker-compose.yml

## docker-composeとは
- 独立した複数のコンテナの紐付け、構築、実行を自動化できる機能
- dockerCliよりもコマンドの数が少なくなる
- Dockerfileで作成したカスタムイメージを読み取って使うことができる

## docker-compose.ymlの書き方

例:
```
version: '3'
services: 
  redis-server:
    image: 'redis'
  node-app:
    build: .
    ports:
      - "4001:8081"
```
1. version: docker-composeのバージョン。バージョンによってdocker-compose.ymlの書き方が変わる

2. services: 再生するコンテナの種類を明示する。`redis-server`と`node-app`というコンテナを作る予定

3. image: コンテナを作成するのに使うイメージ名
4. build: コンテナを作成するのに使うDockerfileのパス。DockerHubからのインストールではなく、Dockerfileの成果物のイメージを使う場合はbuildを指定

5. ports: ポート番号の紐付け。```- "外部からアクセスするポート番号:コンテナ内のポート番号"```

## Node.js側の設定

例:
```
const express = require('express');
const redis = require('redis');

const app = express();
const client = redis.createClient({
  host: 'redis-server',
  port: 6379
});
```
1. requireでexpressフレームワーク、redisを導入している
2. redis.createClient({host,ports})でNode.jsとredisの紐付けを行う。hostはredisのホスト名,ポートはredisのポート番号(デフォルトは6379)
3. host名からdocker-compose.ymlに設定されたコンテナを探し、redisサーバーに接続される

## docker-compose　コマンド

- ```docker-compose up```: まだイメージが作成されていないならイメージを作成し、コンテナを作成する。一度作成したコンテナがあれば、それを起動する。
  - `Creating network ~`と表示され自動でコンテナ間を紐づけるネットワーク層が設立される
- ```docker-compose up --build```: docker-composeファイルを再読み込みしてイメージの変更を更新し、コンテナを起動する
- ```docker-compose up -d```: docker-composeファイルをバックグラウンドで起動。コンテナの出力結果がターミナルに表示されない
- ```docker-compose down```: docker-composeファイルで起動したコンテナを全て止め、削除する
- ```docker-compose ps```: docker-composeで作成したコンテナの稼働状況を確認できる。カレントディレクトリのdocker-compose.ymlを検索するため、違うディレクトリでコマンドを入力してもエラーになる

## dockerCliとの比較

1. `docker build . + docker run imageId`よりも`docker-compose up --build`の方が再起動しやすい
2. コンテナを複数シャットダウンする場合、`docker stop containerId`を何回もするより`docker-compose down`をした方が早い

## dockerコンテナのクラッシュ
コンテナがクラッシュするとシャットダウンされる。docker-composeで複数のコンテナが互いに依存する中で１つのコンテナがシャットダウンされるとサービスを提供できなくなる。

### リスタートポリシー
コンテナが終了した時の挙動を設定できる

|            |                                                |     | 
| ---------- | ---------------------------------------------- | --- | 
| "no"       | 一度シャットダウンされたら再起動しない         |     | 
| always     | シャットダウンされたら再起動する               |     | 
| on-failure | エラーが起きてシャットダウンされたら再起動する |     | 
| unless-stopped | 開発者が強制的に止めるまでシャットダウンされる |

例:
```
version: '3'
services: 
  redis-server:
    image: 'redis'
  node-app:
    restart: always
    build: .
    ports:
      - "4001:8081"
```

node-appサーバーがストップしたら常にリスタートするよう設定する