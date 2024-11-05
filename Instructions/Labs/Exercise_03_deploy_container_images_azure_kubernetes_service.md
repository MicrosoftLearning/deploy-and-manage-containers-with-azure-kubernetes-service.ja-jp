---
Exercise:
  title: '演習: コンテナー イメージを Azure Kubernetes Service にデプロイする'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# 演習 3 - アプリケーションを Azure Kubernetes Service (AKS) にデプロイする

## 目標

このガイド付きプロジェクトは、次の演習で構成されています。

+ 演習 1: Azure Container Registry (ACR) と Azure Kubernetes Service (AKS) をプロビジョニングする。
+ 演習 2: Linux と Windows のコンテナー イメージをビルドし、Azure Container Registry に格納する。
+ **演習 3: コンテナー イメージを Azure Kubernetes Service にデプロイする。**
+ 演習 4:デプロイを確認し、すべてのリソースのプロビジョニングを解除する。

この演習では、コンテナー イメージを Azure Kubernetes Service にデプロイします。

## 演習 3: コンテナー イメージを AKS にデプロイする 
この演習では、この演習で前に作成した 2 つのコンテナー イメージを AKS クラスターにデプロイします。
>**注**: この演習を完了するには、[Azure サブスクリプション](https://azure.microsoft.com/free/)が必要です。
> 指定されていないプロパティには、既定値を使用してください。
> **注:** この演習に進む前に、AKS クラスターのプロビジョニングが正常に完了していることを確認してください。

### タスク 1: カスタム AKS 名前空間を作成する
このタスクでは、このラボで前に作成した AKS クラスターに 2 つの名前空間を作成します。

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

