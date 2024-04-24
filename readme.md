

# Windows で Azure Functions の開発

# 準備
docker ホストのインストール
func, azurite コマンドのインストール
anaconda のインストール
git のインストール


## Anaconda起動
Anaconda Navigator を管理者モードで起動して python 3.11 環境を作成
作成が終わったらｐｙｔｈｏｎ3.11環境をターミナルで起動します
起動したら適当なプロジェクトディレクトリを作成しプロジェクトディレクトリまで移動してください

## azurite, func コマンドのインストール

node.js のツールなので node.js をまずインストールします

```
conda install -c conda-forge nodejs
npm i -g azure-functions-core-tools@4 --unsafe-perm true

```

## git のインストール

```
conda install git
```

# Functionsプロジェクトの作成
以下のコマンドを入力しプロジェクトを作成します
project1 はディレクトリ名でコマンド入力すると作成されます

```
func init project1 --python
```

project1 というディレクトリが作成されます
project1 には以下のファイルが含まれます

```
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        2024/04/24     20:26                .vscode
-a----        2024/04/24     20:26            517 .gitignore
-a----        2024/04/24     20:26            104 function_app.py
-a----        2024/04/24     20:26            302 host.json
-a----        2024/04/24     20:26            206 local.settings.json
-a----        2024/04/24     20:26            207 requirements.txt
```

## venv の作成
project1 ディレクトリに移動します
以下のコマンドを入力して python の仮想環境 venv を作成します
project1 ディレクトリの中に venv ディレクトリが作成されます

```
python -m venv venv
```

次に以下のコマンドを実行して venv 環境を起動します

```
venv\Scripts\activate
```



## func new コマンドで使用できるテンプレート一覧を確認する

```
func templates list >tmp\out.txt
```

一部だけ抜粋します

```
Custom Templates:
  Azure Blob Storage trigger
  Azure Cosmos DB trigger
  Azure Event Grid trigger
  Azure Event Hub trigger
  HTTP trigger
  IoT Hub (Event Hub)
  Kafka trigger
  Azure Queue Storage trigger
  RabbitMQ trigger
  SendGrid
  Azure Service Bus Queue trigger
  Azure Service Bus Topic trigger
  SignalR negotiate HTTP trigger
  Timer trigger

Python Templates:
  Azure Blob Storage trigger
  Azure Cosmos DB trigger
  Durable Functions activity
  Durable Functions entity
  Durable Functions HTTP starter
  Durable Functions orchestrator
  Azure Event Grid trigger
  Azure Event Hub trigger
  HTTP trigger
  Kafka output
  Kafka trigger
  Azure Queue Storage trigger
  RabbitMQ trigger
  Azure Service Bus Queue trigger
  Azure Service Bus Topic trigger
  SQL Input Binding
  SQL Output Binding
  SQL Trigger
  Timer trigger
```

## httpトリガーの作成

```
func new --template "HTTP trigger" --name http_trigger
```

以下のように質問されるので ANONYMOUS を選択します

```
Use the up/down arrow keys to select a Auth Level:
FUNCTION
ANONYMOUS
ADMIN
```

作成に成功すると以下のメッセージが表示されます

```
The function "http_trigger" was created successfully from the "HTTP trigger" template.
```

function_app.py に以下のソースコードが追加されます

``` python3
@app.route(route="http_trigger", auth_level=func.AuthLevel.ANONYMOUS)
def http_trigger(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello, {name}. This HTTP triggered function executed successfully.")
    else:
        return func.HttpResponse(
             "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.",
             status_code=200
        )
```


## 定期実行タスクの作成

```
func new --template "Timer trigger" --name timer_trigger
```

以下のように質問されるので

```
Schedule: [0 */5 * * * *]
```

以下のように回答します、これは１分に１回実行される定期実行になります

```
Schedule: [0 */5 * * * *] 0 */1 * * * *
```


# とりあえず実行してみる


端末を２つ起動します

１つ目の端末では以下のコマンドを実行

```
azurite --location azurite
```

ストレージやキューのエミュレーターが起動します

２つ目の端末では以下のコマンドを入力します

```
func start
```

WEBサーバ、定期実行タスクなどを実行するFunctionsエミュレーターが起動します


# git の準備
今回のプロジェクト用に git リポジトリを作成します

認証の準備

SSHキーの作成

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

作成したキーを git サイトの SSHキーセクションに貼り付けます


gitに認証情報を設定する

```
git remote set-url origin git@github.com:username/repository-name.git
```

認証テスト

```
ssh -T git@github.com
```

ローカルで git の初期化を行う

```
git init
```

.gitignore ファイルに以下を追加します

```
**/tmp/
**/out/
/venv/
/azurite/
```

git にファイルを追加

```
git add .
```

git に状態を保存

```
git commit -m "Initial commit"
```

gitに保存先URLの指定

```
git remote add origin <リポジトリのURL>
```

git push してリモート先に保存する

```
git push -u origin master

```

# git に開発者個人用のブランチを作成してpushする

ローカルを最新にする

```
git checkout main
git pull
```

ブランチを作成して切り替える

```
git checkout -b develop-tracy

```


ブランチをpushする

```
git push -u origin develop-tracy
```





# Azure Portalの設定

## Functionsの作成
Azure portal で適当なグループを作成
グループ内にFunctionsを作成する
作成時点でリソースの状態により github 連携が選べたり選べなかったりするが、作成後にデプロイセンターで設定できるので、ここで設定できなくても問題ない
作成後に デプロイセンター にて ｇｉｔｈｕｂ を割り当てる


### chatGPTによる解説
Azure Portal にログイン:
Azure Portal にアクセスし、ログインします。
Function App を選択:
対象の Azure Functions アプリケーションを選択します。
デプロイメント センターへのアクセス:
左側のナビゲーションメニューから「デプロイメントセンター」を選択します。
デプロイメントソースの設定:
GitHub または使用している Git リポジトリをデプロイメントソースとして設定し、develop-hoge ブランチを選択してデプロイメントのトリガーとします。
ビルドプロバイダとデプロイメントトリガーの設定:
必要に応じてビルドプロバイダを設定し（例えば Kudu、GitHub Actions 等）、プッシュトリガーを有効にします。



# SQLServerの準備


## pythonライブラリのインストール

