---
Exercise:
  title: '演習: デプロイを確認し、すべてのリソースのプロビジョニングを解除する'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# 演習 4 - アプリケーションを Azure Kubernetes Service (AKS) にデプロイする

## 目標

このガイド付きプロジェクトは、次の演習で構成されています。

+ 演習 1: Azure Container Registry (ACR) と Azure Kubernetes Service (AKS) をプロビジョニングする
+ 演習 2: Linux と Windows のコンテナー イメージをビルドし、ACR に格納する
+ 演習 3: コンテナー イメージを AKS にデプロイする
+ **演習 4: デプロイを確認し、すべてのリソースのプロビジョニングを解除する**

この演習では、前の演習でのサービスのデプロイを確認し、すべてのリソースのプロビジョニングを解除します。

# 演習 4: デプロイを確認し、すべてのリソースのプロビジョニングを解除する
この演習では、デプロイの結果を確認し、すべてのリソースのプロビジョニングを解除します。

>**注**: この演習を完了するには、[Azure サブスクリプション](https://azure.microsoft.com/free/)が必要です。
> 指定されていないプロパティには、既定値を使用してください。

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
このタスクでは、このラボでプロビジョニングされたすべてのリソースを削除します。

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
