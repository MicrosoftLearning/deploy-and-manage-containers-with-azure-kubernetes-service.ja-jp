---
Guided Exercise:
  title: ガイド付き演習 AKS へのアプリケーションのデプロイ
---
# ガイド付き演習 - アプリケーションを Azure Kubernetes Service (AKS) にデプロイする

## 目標

このガイド付き演習は、次のアクティビティで構成されます。

+ 演習 1: Azure Container Registry (ACR) と Azure Kubernetes Service (AKS) をプロビジョニングする
+ 演習 2: Linux と Windows のコンテナー イメージをビルドし、ACR に格納する
+ 演習 3: コンテナー イメージを AKS にデプロイする 
+ 演習 4: デプロイを確認し、すべてのリソースのプロビジョニングを解除する

## 演習 1: Azure Container Registry (ACR) と Azure Kubernetes Service (AKS) をプロビジョニングする
この演習では、Azure コンテナー レジストリと AKS クラスターを作成します。


>**注**: この演習を完了するには、[Azure サブスクリプション](https://azure.microsoft.com/free/)が必要です。
> 指定されていないプロパティには、既定値を使用してください。


### タスク 1: Azure コンテナー レジストリを作成する
このタスクでは、Azure コンテナー レジストリを作成します

1. お使いのコンピューターから Web ブラウザー ウィンドウを開き、https://portal.azure.com で Azure portal に移動します。
1. メッセージが表示されたら、この演習で使用する Azure サブスクリプションの所有者ロールを持つユーザー アカウントを使用してログインします。 
1. Azure portal にサインインします。 
1. Azure portal の **[検索]** テキスト ボックスで、 **[コンテナー レジストリ]** を検索して選択します。
1. **[コンテナー レジストリ]** ページで、 **[+ 作成]** を選択し、次の設定を指定します。

    |設定|値|
    |---|---|
    |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
    |リソース グループ|新しいリソース グループの名前 **acr-01-RG**|
    |レジストリ名|5 から 50 文字の英数字で構成されるグローバルに一意の任意の有効な名前|
    |リージョン|Azure コンテナー レジストリと AKS クラスターを作成できる任意の Azure リージョン|
    |可用性ゾーン|**なし**|
    |SKU|**Basic**|

1. **[コンテナー レジストリ]** ページで、 **[確認 + 作成]** を選択し、 **[確認 + 作成]** タブで **[作成]** を選択します。

   > **注:** Azure コンテナー レジストリのプロビジョニングが完了するのを待たずに、次の演習に進みます。

### タスク 2: Azure 仮想ネットワークと AKS クラスターを作成する
このタスクでは、Azure 仮想ネットワークを作成し、Windows ノード プールを含む AKS クラスターをその仮想ネットワークにデプロイします。

> **注:** AKS クラスターをプロビジョニングするときに仮想ネットワークを作成できますが、これは一般的なシナリオではありません。 さらに重要なのは、AKS クラスターを既存の仮想ネットワークにデプロイするには追加の考慮事項が必要であり、それに習熟しておく必要があることです。

1. Azure portal の **[検索]** テキスト ボックスで、 **[仮想ネットワーク]** を検索して選択します。
1. **[仮想ネットワーク]** ページで **[+ 作成]** を選択し、 **[仮想ネットワークの作成]** ページの **[基本]** タブで、次の設定を指定します。

    |設定|値|
    |---|---|
    |サブスクリプション|最初の演習で選択した Azure サブスクリプションの名前|
    |リソース グループ|新しいリソース グループの名前 **aks-01-RG**|
    |仮想ネットワーク名|**vnet-01**|
    |リージョン|この演習の最初の演習で選択したのと同じ Azure リージョン|

1. **[仮想ネットワークの作成]** ページの **[基本]** タブで、 **[次へ]** を選択します。
1. **[仮想ネットワークの作成]** ページの **[セキュリティ]** タブで、既定の設定をそのまま使用し、 **[次へ]** を選択します。
1. **[仮想ネットワークの作成]** ページの **[IP アドレス]** タブで、IP アドレス空間が **10.0.0.0/16** に設定されていることを確認し、**既定**のサブネットを削除します。次に、 **[確認 + 作成]** を選択し、 **[確認 + 作成]** タブで **[作成]** を選択します。

   > **注:** 仮想ネットワークの作成には数秒しかかからないため、次の手順に直接進むことができます。

1. Azure portal の **[検索]** テキスト ボックスで、 **[Kubernetes サービス]** を検索して選択します。
1. **[Kubernetes サービス]** ページで **[+ 作成]** を選択し、ドロップダウン リストで **[Kubernetes クラスターの作成]** を選択して、 **[Kubernetes クラスターの作成]** ページの **[基本]** タブで、次の設定を指定します。

    |設定|値|
    |---|---|
    |サブスクリプション|最初の演習で選択した Azure サブスクリプションの名前|
    |リソース グループ|**aks-01-RG**|
    |クラスターのプリセット構成|**Dev/Test**|
    |Kubernetes クラスター名|**aks-01**|
    |リージョン|最初の演習で選択したのと同じ Azure リージョン|
    |可用性ゾーン|**なし**|
    |AKS 価格レベル|**Free**|
    |Kubernetes バージョン|既定値をそのまま使用します|
    |自動アップグレード|無効|
    |ノード サイズ|**Standard B4ms**|
    |スケーリング方法|**[手動]**|
    |ノード数|**2**|

   > **注:** ノード サイズとノード数の値に合わせて、vCPU クォータを増やすか、VM SKU を変更することが必要な場合があります。 vCPU クォータを増やす手順については、Microsoft Learn の記事「[VM ファミリの vCPU クォータの増加](https://learn.microsoft.com/en-us/azure/quotas/per-vm-quota-requests)」を参照してください。

1. **[Kubernetes クラスターの作成]** ページの **[基本]** タブで、 **[次へ: ノード プール >]** を選択します。
1. **[Kubernetes クラスターの作成]** ページの **[ノード プール]** タブで、 **[次へ: アクセス >]** を選択します。
1. **[Kubernetes クラスターの作成]** ページの **[アクセス]** タブで、 **[次へ: ネットワーク >]** を選択します。
1. **[Kubernetes クラスターの作成]** ページの **[ネットワーク]** タブで、 **[Azure CNI]** オプションを選択し、 **[仮想ネットワーク]** ドロップダウン リストで **[vnet-01]** を選択して、 **[クラスター サブネット]** テキストボックスの下にある **[Managed subnet configuration](マネージド サブネット構成)** を選択します。
1. **[vnet-01 \| サブネット]** ページで **[+ サブネット]** を選択します。
1. **[サブネットの追加]** ページで、次の設定を指定して、 **[保存]** を選択します。

    |設定|値|
    |---|---|
    |名前|**aks-subnet**|
    |サブネットのアドレス範囲|**10.0.0.0/20**|

1. **[vnet-01 \| サブネット]** ページに戻り、ページの左上部分の階層リンクの軌跡で、 **[Kubernetes クラスターの作成]** を選択します。 
1. **[Kubernetes クラスターの作成]** ページの **[ネットワーク]** タブに戻り、次の設定を指定します。

    |設定|Value|
    |---|---|
    |クラスター サブネット|**aks-subnet (10.0.0.0/20)**|
    |Kubernetes サービスのアドレス範囲|**172.16.0.0/22**|
    |Kubernetes DNS サービスの IP アドレス|**172.16.3.254**|
    |DNS 名プレフィックス|**aks-01-dns**|
    |プライベート クラスターの有効化|無効|
    |承認された IP 範囲の設定|無効|
    |ネットワーク ポリシー|**なし**|

1. **[Kubernetes クラスターの作成]** ページの **[ネットワーク]** タブで、 **[ノード プール]** タブを選択します。

   > **注:** クラスターに Windows ノード プールを追加します。 これには、ネットワーク構成を既定の **Kubenet** から **Azure CNI** に変更する必要がありました。 Kubenet ネットワーク構成では、Windows ノード プールはサポートされません。

1. **[Kubernetes クラスターの作成]** ページの **[ノード プール]** タブで、 **[+ ノード プールの追加]** を選択します。
1. **[ノード プールの追加]** ページで、次の設定を指定します。

    |設定|Value|
    |---|---|
    |ノード プール名|**w1pool**|
    |Mode|**User**|
    |OS の種類|**Windows**|
    |可用性ゾーン|**なし**|
    |Azure Spot インスタンスを有効にする|無効|
    |ノード サイズ|**Standard B4s_v2**|
    |スケーリング方法|**[手動]**|
    |ノード数|**2**|
    |ノードごとの最大ポッド数|**30**|
    |ノードごとにパブリック IP を有効にする|無効|

   > **注:** ノード サイズとノード数の値に合わせて、vCPU クォータを増やすか、VM SKU を変更することが必要な場合があります。 vCPU クォータを増やす手順については、Microsoft Learn の記事「[VM ファミリの vCPU クォータの増加](https://learn.microsoft.com/en-us/azure/quotas/per-vm-quota-requests)」を参照してください。

1. **[ノード プールの追加]** ページで、 **[追加]** を選択します。
1. **[Kubernetes クラスターの作成]** ページの **[ノード プール]** タブで、 **[統合]** タブを選択します。
1. **[Kubernetes クラスターの作成]** ページの **[統合]** タブの **[コンテナー レジストリ]** ドロップダウン リストで、前の演習で作成した Azure コンテナー レジストリを表すエントリを選択し、 **[Enable recommended alert rules](推奨されるアラート ルールを有効にする)** チェック ボックスをオフにします。次に、 **[Azure Policy]** オプションが無効になっていることを確認して、 **[確認 + 作成]** を選択します。
1. **[Kubernetes クラスターの作成]** ページの **[確認 + 作成]** タブで、 **[作成]** を選択します。

   > **注:** AKS クラスターのプロビジョニングが完了するのを待たずに、次の演習に進みます。 プロビジョニング プロセスには約 5 分かかる場合があります。

## 演習 2: Linux と Windows のコンテナー イメージをビルドし、ACR に格納する
この演習では、Linux ベースと Windows ベースの Docker イメージをビルドし、この演習で前に作成した Azure コンテナー レジストリにプッシュします。

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

## 演習 3: コンテナー イメージを AKS にデプロイする 
この演習では、この演習で前に作成した 2 つのコンテナー イメージを AKS クラスターにデプロイします。

> **注:** この演習に進む前に、AKS クラスターのプロビジョニングが正常に完了していることを確認してください。

### タスク 1: カスタム AKS 名前空間を作成する
このタスクでは、この演習で前に作成した AKS クラスターに 2 つの名前空間を作成します。

1. Azure portal の **[検索]** テキスト ボックスで、 **[Kubernetes サービス]** を検索して選択します。
1. **[Kubernetes サービス]** ページで、 **[aks-01]** を選択します。
1. **[aks-01]** ページの垂直のハブ メニューで、 **[名前空間]** を選択します。
1. **[aks-01 \| 名前空間]** ページで、 **[+ 作成]** を選択し、ドロップダウン メニューで **[名前空間]** を選択します。
1. **[名前空間を作成する]** ペインの **[名前]** テキストボックスに「**dev-node**」と入力し、 **[作成]** を選択します。
1. **[aks-01 \| 名前空間]** ページで、 **[+ 作成]** を選択し、ドロップダウン メニューで **[名前空間]** を選択します。
1. **[名前空間を作成する]** ペインの **[名前]** テキストボックスに「**dev-dotnet**」と入力し、 **[作成]** を選択します。

### タスク 2: Linux イメージをデプロイするための Kubernetes マニフェストを作成する
このタスクでは、Linux イメージを Linux ノード プールにデプロイするための Kubernetes マニフェストを作成します

1. Azure portal で、**[Cloud Shell]** アイコンを選択します。
1. [Cloud Shell] ペインの左上隅にあるドロップダウン メニューに、 **[Bash]** が表示されていることを確認します。
1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して、Linux イメージに基づいてポッドをプロビジョニングするためのデプロイ マニフェストをホストするディレクトリを作成し、現在のディレクトリからそこに切り替えます。

   ```bash
   mkdir ~/aks-l01
   cd ~/aks-l01
   ```

1. Azure Cloud Shell の Bash セッションで、組み込みエディターを使用して aks-deployment-l01.yaml という名前のファイルを aks-l01 ディレクトリに作成し、それに次の内容をコピーします。

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hellofromnode-deployment
     labels:
       environment: dev
       app: hellofromnode
   spec:
     replicas: 1
     template:
       metadata:
         name: hellofromnode
         labels:
           app: hellofromnode
       spec:
         nodeSelector:
           "kubernetes.io/os": linux
         containers:
         - name: hellofromnode
           image: ACR_NAME.azurecr.io/hellofromnode:v1.0
           resources:
             limits:
               cpu: 1
               memory: 800M
           ports:
             - containerPort: 80
     selector:
       matchLabels:
         app: hellofromnode
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: hellofromnode-service
   spec:
     type: LoadBalancer
     ports:
     - protocol: TCP
       port: 80
     selector:
       app: hellofromnode
   ```

   > **注:** デプロイは、Linux コンテナー イメージに基づいて AKS クラスター内の Linux ノード プールに対するポッドを作成します。 さらにマニフェストには、ポート 80 のパブリック IP アドレス経由で、デプロイ内のポッドへの負荷分散アクセスを提供するサービスが含まれています。

1. ファイルへの変更を保存して閉じ、Bash プロンプトに戻ります。
1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して、ファイル aks-deployment-l01.yaml の **ACR_NAME** プレースホルダーを置き換えます。

   ```azurecli
   ACR_RGNAME='acr-01-RG'
   ACR_NAME=$(az acr list --resource-group $ACR_RGNAME --query "[].name" --output tsv)
   sed -i "s/ACR_NAME/$ACR_NAME/" ./aks-deployment-l01.yaml
   ```

### タスク 3: Windows イメージをデプロイするための Kubernetes マニフェストを作成する
このタスクでは、Windows イメージを Windows ノード プールにデプロイするための Kubernetes マニフェストを作成します

1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して、Windows イメージに基づいてポッドをプロビジョニングするためのデプロイ マニフェストをホストするディレクトリを作成し、現在のディレクトリからそこに切り替えます。

   ```bash
   mkdir ~/aks-w01
   cd ~/aks-w01
   ```

1. Azure Cloud Shell の Bash セッションで、組み込みエディターを使用して aks-deployment-w01.yaml という名前のファイルを aks-w01 ディレクトリに作成し、それに次の内容をコピーします。

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hellofromdotnet-deployment
     labels:
       environment: dev
       app: hellofromdotnet
   spec:
     replicas: 1
     template:
       metadata:
         name: hellofromdotnet
         labels:
           app: hellofromdotnet
       spec:
         nodeSelector:
           "kubernetes.io/os": windows
         containers:
         - name: hellofromdotnet
           image: ACR_NAME.azurecr.io/hellofromdotnet:v1.0
           resources:
             limits:
               cpu: 1
               memory: 800M
           ports:
             - containerPort: 80
     selector:
       matchLabels:
         app: hellofromdotnet
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: hellofromdotnet-service
   spec:
     type: LoadBalancer
     ports:
     - protocol: TCP
       port: 80
     selector:
       app: hellofromdotnet
   ```

   > **注:** デプロイは、Windows コンテナー イメージに基づいて AKS クラスター内の Windows ノード プールに対するポッドを作成します。 さらにマニフェストには、ポート 80 のパブリック IP アドレス経由で、デプロイ内のポッドへの負荷分散アクセスを提供するサービスが含まれています。

1. ファイルへの変更を保存して閉じ、Bash プロンプトに戻ります。
1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して、ファイル aks-deployment-l01.yaml の **ACR_NAME** プレースホルダーを置き換えます。

   ```azurecli
   sed -i "s/ACR_NAME/$ACR_NAME/" ./aks-deployment-w01.yaml
   ```

### タスク 4: YAML マニフェスト ファイルを使用して AKS デプロイを実行する
このタスクでは、両方のコンテナー イメージを、ターゲット AKS クラスター内のそれぞれの名前空間とノード プールにデプロイします。

1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して AKS クラスターに接続します。

   ```azurecli
   AKSRG='aks-01-RG'
   AKSNAME='aks-01'
   az aks get-credentials --resource-group $AKSRG --name $AKSNAME
   ```

1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して、接続が成功したことを確認します。

   ```kubectl
   kubectl get nodes
   ```

   > **注:** コマンドの出力には、すべての AKS ノード (この例では 4 つ) の一覧が含まれている必要があります。

1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して、対応する YAML マニフェスト ファイルに定義された最初のデプロイを **dev-node** 名前空間に作成します。

   ```kubectl
   cd ~/aks-l01
   kubectl apply -f aks-deployment-l01.yaml -n=dev-node
   ```

   > **注:** デプロイが完了するのを待たずに、次の手順に進みます。 すべてのリソースのプロビジョニングには数分かかる場合があります。

1. Azure Cloud Shell の Bash セッションで、次のコマンドを実行して、対応する YAML マニフェスト ファイルに定義された 2 番めのデプロイを作成します。

   ```kubectl
   cd ~/aks-w01
   kubectl apply -f aks-deployment-w01.yaml -n=dev-dotnet
   ```

   > **注:** デプロイが完了するのを待たずに、次の手順に進みます。 すべてのリソースのプロビジョニングには数分かかる場合があります。

# 演習 4: デプロイを確認し、すべてのリソースのプロビジョニングを解除する
この演習では、デプロイの結果を確認し、すべてのリソースのプロビジョニングを解除します。

### タスク 1: AKS のデプロイとサービスを確認する
このタスクでは、デプロイとサービスのオブジェクトを含む、両方のデプロイの結果を確認します。

1. Azure Cloud Shell の Bash セッションから、次のコマンドを実行して両方のデプロイの状態を表示します。

   ```kubectl
   kubectl get deployments -n=dev-node
   kubectl get deployments -n=dev-dotnet
   ```

   > **注:** 次の手順に進む前に、両方のデプロイが準備完了状態で一覧表示されていることを確認します。 そのようになっていない場合は、もう 1 分間待ってから、前述の 2 つのコマンドを再実行し、デプロイの状態をもう一度チェックします。

1. Azure Cloud Shell の Bash セッションから、次のコマンドを実行して、マニフェスト ファイルに含まれていた 2 つのサービスの状態を表示します。

   ```kubectl
   kubectl get services -n=dev-node
   kubectl get services -n=dev-dotnet
   ```

1. 各サービスの一覧に **EXTERNAL-IP** 列の値が含まれていることを確認します。 
1. Web ブラウザーを使用して、前の手順で特定した IP アドレスに移動し、結果の Web ページにそれぞれ **Hello World from Node** と **Hello World from .Net 7** というメッセージが表示されることを確認します。

### タスク 2: すべてのリソースを削除する
このタスクでは、この演習でプロビジョニングされたすべてのリソースを削除します。

1. Azure Cloud Shell の Bash セッションから、次のコマンドを実行して、この演習でプロビジョニングされた 2 つのリソース グループ内のリソースの一覧を表示します。

   ```azurecli
   az resource list --resource-group 'acr-01-RG' --query "[].name" --output tsv
   az resource list --resource-group 'aks-01-RG' --query "[].name" --output tsv
   ```

   > **注:** これらが、削除するリソースであることを確認します。 そうであれば、次の手順に進みます。

1. Azure Cloud Shell の Bash セッションから、次のコマンドを実行して、この演習でプロビジョニングされたすべてのリソースを削除します。

   ```azurecli
   az group delete --name 'acr-01-RG' --no-wait --yes
   az group delete --name 'aks-01-RG' --no-wait --yes
   ```

   > **注:** コマンドは非同期的に実行されます (--nowait パラメーターによって強制されます)。そのため、両方のコマンドが Bash シェル プロンプトに直ちに戻っても、リソース グループとそのリソースが実際に削除されるまでに数分かかります。

1. Azure Cloud Shell ウィンドウを閉じます。
