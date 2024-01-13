---
Exercise:
  title: '演習: Linux と Windows のコンテナー イメージをビルドし、Azure Container Registry に格納する'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# 演習 2 - アプリケーションを Azure Kubernetes Service (AKS) にデプロイする

## 目標

このガイド付きプロジェクトは、次の演習で構成されています。

+ 演習 1:Azure Container Registry (ACR) と Azure Kubernetes Service (AKS) をプロビジョニングする。
+ **演習 2: Linux と Windows のコンテナー イメージをビルドし、Azure Container Registry に格納する。**
+ 演習 3: コンテナー イメージを Azure Kubernetes Service にデプロイする。
+ 演習 4:デプロイを確認し、すべてのリソースのプロビジョニングを解除する。

この演習では、Linux と Windows のコンテナー イメージをビルドし、Azure Container Registry に格納します。

## 演習 2: Linux と Windows のコンテナー イメージをビルドし、ACR に格納する
この演習では、Linux と Windows ベースの Docker イメージをビルドし、このラボで前に作成した Azure コンテナー レジストリにプッシュします。


>**注**: この演習を完了するには、[Azure サブスクリプション](https://azure.microsoft.com/free/)が必要です。
> 指定されていないプロパティには、既定値を使用してください。


### タスク 1: Linux コンテナー イメージをビルドし、ACR に格納する
このタスクでは、ACR タスクを使用して Linux コンテナー イメージをビルドし、それを ACR に自動的にプッシュします。

1. Azure portal で、**[Cloud Shell]** アイコンを選択します。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。 
1. メッセージが表示されたら、 **[ストレージの作成]** を選択し、[Azure Cloud Shell] ペインが表示されるまで待ちます。 
1. [Cloud Shell] ペインの左上隅にあるドロップダウン メニューに、 **[Bash]** が表示されていることを確認します。
1. Cloud Shell 内の Bash セッションで、次のコマンドを実行して、Linux イメージの Dockerfile をホストするディレクトリを作成し、現在のディレクトリからそこに切り替えます。

   ```bash
   mkdir ~/image-l01
   cd ~/image-l01
   ```

1. Azure Cloud Shell の Bash セッションで、組み込みエディターを使用して server.js という名前のファイルを image-l01 ディレクトリに作成し、それに次の内容をコピーします。

   ```js
   const http = require('http')
   const port = 80
   const server = http.createServer((request, response) => {
     response.writeHead(200, {'Content-Type': 'text/plain'})
     response.write('Hello World from Node\n')
     response.end('Version: ' + process.env.NODE_VERSION + '\n')
   })
   server.listen(port)
   console.log(`Server running at http://localhost: ${port}`)
   ```

   > **注:** 完成した Node.js コードを実行すると、**Hello World from Node** というメッセージが表示されます。

1. Azure Cloud Shell の Bash セッションで、組み込みエディターを使用して package.json という名前のファイルを image-l01 ディレクトリに作成し、それに次の内容をコピーします。

   ```json
   {
     "name": "helloworld",
     "version": "1.0.0",
     "description": "Sample app for ACR Build",
     "main": "server.js",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1",
       "start": "node server.js"
     },
     "license": "MIT"
   }
   ```

1. ファイルへの変更を保存して閉じ、Bash プロンプトに戻ります。
1. Azure Cloud Shell の Bash セッションで、組み込みエディターを使用して Dockerfile という名前のファイルを image-l01 ディレクトリに作成し、それに次の内容をコピーします。

   ```Dockerfile
   FROM node:20.2-alpine
   COPY . /src
   RUN cd /src && npm install
   EXPOSE 80
   CMD ["node", "/src/server.js"]
   ```

1. ファイルへの変更を保存して閉じ、Bash プロンプトに戻ります。
1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して、この演習で前に作成した Azure コンテナー レジストリの名前を特定し、 **$ACRNAME** という名前の変数に格納します。

   ```azurecli
   ACR_RGNAME='acr-01-RG'
   ACR_NAME=$(az acr list --resource-group $ACR_RGNAME --query "[].name" --output tsv)
   ```

1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して、現在のディレクトリに格納されている Dockerfile に基づいて Docker イメージを作成し、 **$ACRNAME** 変数に名前が格納されている Azure コンテナー レジストリに自動的にプッシュします (末尾のピリオドを含めるようにしてください)。

   ```azurecli
   az acr build --registry $ACR_NAME --image hellofromnode:v1.0 .
   ```

   > **注:** ビルドの進行状況を追跡し、正常に完了したことを確認してください。 これにかかる時間は 1 分未満です。

### タスク 2: Windows コンテナー イメージをビルドし、ACR に格納する
このタスクでは、ACR タスクを使用して Windows コンテナー イメージをビルドし、それを ACR に自動的にプッシュします。

1. Cloud Shell 内の Bash セッションで、次のコマンドを実行して、Windows イメージの Dockerfile をホストするディレクトリを作成し、現在のディレクトリからそこに切り替えます。

   ```bash
   mkdir ~/image-w01
   cd ~/image-w01
   ```

1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して、Windows イメージのビルドに使用するファイルをホストするパブリック GitHub リポジトリを複製し、現在のディレクトリから複製されたリポジトリに切り替えます。

   ```git
   git clone https://github.com/Azure-Samples/dotnetcore-docs-hello-world.git
   cd ~/image-w01/dotnetcore-docs-hello-world
   ```

   > **注:** リポジトリには、**Hello World from .NET 7** というメッセージを表示する .NET 7 Web アプリのソース コードが含まれています。

1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して、現在のディレクトリに格納されている Dockerfile に基づいて Docker イメージを作成し、 **$ACRNAME** 変数に名前が格納されている Azure コンテナー レジストリに自動的にプッシュします (末尾のピリオドを含めるようにしてください)。

   ```azurecli
   az acr build --registry $ACR_NAME --image hellofromdotnet:v1.0 --platform windows --file Dockerfile.windows .
   ```

   > **注:** Windows イメージのビルドを定義する Dockerfile の名前は **Dockerfile.windows** です。

   > **注:** ビルドの進行状況を追跡し、正常に完了したことを確認してください。 これにかかる時間は 3 分未満です。

1. [Azure Cloud Shell] ペインを閉じます。
1. Azure portal で、 **[コンテナー レジストリ]** ページに移動し、両方のイメージをプッシュしたコンテナー レジストリを表すエントリを選択します。
1. [コンテナー レジストリ] ページの垂直のハブ メニューで **[リポジトリ]** を選択し、リポジトリの一覧に **hellofromnode** と **hellofromdotnet** が表示されることを確認します。
