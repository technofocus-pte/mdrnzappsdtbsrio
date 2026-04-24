# ラボ2 - 既存のASP.NETアプリをAzure上のエージェントアプリに拡張する

このラボでは、既存のデータ駆動型 ASP.NET Core CRUD
アプリケーションにエージェント機能を追加する方法を説明します。この方法は、Microsoft
Agent Framework を使用します。

ウェブアプリケーションにショッピング、ホテル予約、データ管理などの便利な機能が既に備わっている場合、それらの機能をツールとしてラップすることで、エージェント機能をウェブアプリケーションに追加するのは比較的簡単です。このラボでは、まずシンプルなToDoリストアプリから始めます。最終的には、App
Serviceアプリでエージェントを使用してタスクを作成、更新、管理できるようになります。

## タスク 1: ASP.NET アプリケーションをローカルで実行する

この演習では、アプリケーションをローカルシステム上で実行します。

1.  **C**ドライブから、**Labfilesのzip**ファイルを解凍します。

2.  VS Code を開きます。 **\[Open Folder\]**を選択します。

    ![](./media/image1.png)

3.  **C:/Labfiles**から**app-service-agentic-semantic-kernel-ai-foundry-agent-main**フォルダーを選択します。

    ![](./media/image2.png)

4.  ポップアップ画面で**「Yes, I trust the
    authors」**を選択してください。

    ![](./media/image3.png)

5.  上部メニューの3つの点を選択します。 **「Terminal」** → **「New
    Terminal」**を選択してVS
    Codeのターミナルを開き、アプリを実行します。

    ![](./media/image4.png)

6.  開いたら、ターミナルで以下のコマンドを実行して、指定したバージョンの
    NuGet パッケージ Microsoft.Agents.AI
    を現在のプロジェクトにインストールします。

    +++dotnet add package Microsoft.Agents.AI --version
1.0.0-preview.251204.1+++

    ![](./media/image5.png)

7.  次に、コマンド +++dotnet build+++
    を実行してプロジェクトをビルドします。

    ![](./media/image6.png)

8.  次に、コマンド「+++dotnet
    run+++」を実行してプロジェクトを実行します。

    ![](./media/image7.png)

9.  コマンドが実行されたら、ブラウザを開いて +++http://localhost:5280+++にアクセスしてください。

    ![](./media/image8.png)

10. 左側のメニューから「Microsoft Agent
    Framework」を選択すると、「構成されていません」というメッセージが表示されます。この設定は、この実習の後半で更新します。

    ![](./media/image9.png)

11. アプリケーションを停止するには、 VS
    CodeのターミナルでCtrl+Cを押してください。

## エージェントコードを確認してください

エージェントはプロバイダー内のサービスとして初期化され（
Program.cs内）、それぞれのBlazorコンポーネントに注入されます。

AgentFrameworkProviderは*Services/
AgentFrameworkProvider.cs*で初期化されます。初期化コードでは以下の処理が行われます。

- AzureOpenAIClientを使用して、Azure OpenAI
  からIChatClientを作成します。

- CRUDアプリケーションの機能をカプセル化したTaskCrudToolインスタンスを取得します（
  *Tools/
  TaskCrudTool.cs内*）。ツールメソッドのDescription属性は、エージェントがメソッドの呼び出し方法を判断するのに役立ちます。

- CreateAIAgent () を使用して、 AIFunctionFactory.Create
  ()で登録された指示とツールを使用してAI エージェントを作成します。

- エージェントがナビゲーションをまたいで会話を継続するためのスレッドを作成します。

// Create IChatClient
IChatClient chatClient = new AzureOpenAIClient(
        new Uri(endpoint),
        new DefaultAzureCredential())
    .GetChatClient(deployment)
    .AsIChatClient();

// Get TaskCrudTool instance from service provider
var taskCrudTool = sp.GetRequiredService<TaskCrudTool>();

// Create agent with tools
var agent = chatClient.CreateAIAgent(
    instructions: @"You are an agent that manages tasks using CRUD operations. 
        Use the provided functions to create, read, update, and delete tasks. 
        Always call the appropriate function for any task management request.
        Don't try to handle any requests that are not related to task management.
        When handling requests, if you're missing any information, don't make it up but prompt the user for it instead.",
    tools:
    [
        AIFunctionFactory.Create(taskCrudTool.CreateTaskAsync),
        AIFunctionFactory.Create(taskCrudTool.ReadTasksAsync),
        AIFunctionFactory.Create(taskCrudTool.UpdateTaskAsync),
        AIFunctionFactory.Create(taskCrudTool.DeleteTaskAsync)
    ]);

// Create thread for this scoped instance (persists across navigation)
var thread = agent.GetNewThread();

return (agent, thread);


ユーザーがメッセージを送信するたびに、Blazorコンポーネント（
*Components/Pages/
AgentFrameworkAgent.razor内*）は、ユーザー入力とエージェントスレッドを引数としてAgent.RunAsync
()を呼び出します。エージェントスレッドはチャット履歴を管理します。

var response = await this.Agent.RunAsync ( sentInput , this.agentThread
);

## タスク2：サンプルアプリケーションをデプロイする

このサンプルには、マネージドIDを使用してApp
Serviceアプリを作成し、サンプルアプリケーションをデプロイするAzure
Developer CLI（AZD）テンプレートが含まれています。

1.  ターミナルで、コマンド「+++ az login+++」を実行します。

    ![](./media/image10.png)

2.  Work or school accountを選択してください。

    ![](./media/image11.png)

3.  ログイン情報を使用してログインしてください

    - ユーザー名 - +++@ lab.CloudPortalCredential (User1).Username+++

    - パスワード - +++@ lab.CloudPortalCredential (User1).Password+++

    ![](./media/image12.png)

    ![](./media/image13.png)

4.  **「No, this app only」**を選択してください。

    ![](./media/image14.png)

5.  **Enter**キーを押して、サブスクリプションを承認します。

    ![](./media/image15.png)

6.  次のステップとして、コマンド「+++ azd auth
    login+++」を実行し、ログインしていることを確認してください。

    ![](./media/image16.png)

    ![](./media/image17.png)

7.  これが完了したら、+++ azd up+++ を実行してリソースをデプロイします。

    ![](./media/image18.png)

8.  エンビロンメント名を+++envt@lab.LabInstance.Id+++の形式で入力してください。

    ![](./media/image19.png)

    ![](./media/image20.png)

9.  サブスクリプションを選択します。表示されるサブスクリプションは1つのみですので、**Enter**キーを押して選択してください。

    ![](./media/image21.png)

10. 矢印キーを使って、Resource
    groupとして**「ResourceGroup1」**を選択してください。

    ![](./media/image22.png)

11. リソースが作成され、完了までには約10分かかります。

    ![](./media/image23.png)

12. 処理が完了すると、**エンドポイント(Endpoint)**がターミナルに表示されます。

    ![](./media/image24.png)

13. Azureからアプリケーションにアクセスするには、エンドポイントURLを開いてください。

    ![](./media/image25.png)

    ![](./media/image26.png)

    ![](./media/image27.png)

    ![](./media/image28.png)

## タスク3: Microsoft Foundry リソースの作成と構成

この演習では、Microsoft Foundry
リソースを作成し、使用するモデルをデプロイします。

1.  https://ai.azure.com
    にアクセスし、認証情報を使用してサインインしてください。一番上の**「New
    Foundry」**ラジオボタンが有効になっていることを確認してください。

    - ユーザー名 - +++@ lab.CloudPortalCredential (User1).Username+++

    - パスワード - +++@lab.CloudPortalCredential(User1).Password+++

2.  「Select a project to continue」ダイアログで、「**Create a new
    project」**を選択します。

    ![](./media/image29.png)

3.  プロジェクト名を<+++proj@lab.LabInstance.Id> +++ と入力し、Resource
    groupを**ResourceGroup1**として選択してから、
    **\[Create\]**を選択します。

    ![](./media/image30.png)

    ![](./media/image31.png)

4.  **Old Foundry**に戻ります。OverviewページからAzure
    OpenAIを選択し、エンドポイントの値をコピーしてメモ帳に保存します。

    ![](./media/image32.png)

5.  左側のペインから「Models + endpoints」を選択します。 **「+ Deploy
    model** -\> **Deploy base model」**を選択します。

    ![](./media/image33.png)

6.  gpt-4.1を選択し、**「Confirm」**をクリックします。

    ![](./media/image34.png)

7.  **「Deploy」**をクリックしてください。

    ![](./media/image35.png)

8.  モデル名をコピーしてメモ帳に貼り付けてください。

    ![](./media/image36.png)

## タスク4：必要な権限を割り当てる

この演習では、Foundryリソースに必要な権限を割り当てます。

1.  https://portal.azure.com
    にアクセスし、必要に応じて認証情報を使用してログインしてください。ホーム画面から**Foundry**を選択してください。

    - ユーザー名 - +++@ lab.CloudPortalCredential (User1).Username+++

    - パスワード - +++@ lab.CloudPortalCredential (User1).Password+++

    ![](./media/image37.png)

1.  前回のタスクで作成したFoundryリソースを選択してください。

    ![](./media/image38.png)

2.  左側のペインから**「Access control（IAM）」**を選択します。

    ![](./media/image39.png)

3.  **「+ Add** -\> **Add role assignment」**を選択します。

    ![](./media/image40.png)

4.  「+++Cognitive Services OpenAI User+++」を検索して選択します。

    ![](./media/image41.png)

5.  **「Next」**を選択して次に進んでください。

    ![](./media/image42.png)

6.  **「Managed identity」**を選択し、次に**「Select
    members」**を選択します。

    ![](./media/image43.png)

7.  **Foundry project**として**Managed
    identity**を選択し、次にproj@lab.LabInstance.Idを選択します。

    ![](./media/image44.png)

8.  次の2つの画面で**「Review + assign」**を選択してください。

    ![](./media/image45.png)

    ![](./media/image46.png)

## タスク5：サンプルアプリケーションで接続変数を設定する

このタスクでは、接続変数を使用してサンプルアプリケーションを設定します。

1.  VS Code から*appsettings.jsonを開きます*。Foundry
    ポータルから先ほどコピーした値を使用して、以下の変数を設定します。

    - AzureOpenAIEndpoint - Azure
    OpenAIエンドポイント（従来のFoundryポータルからコピー）。

    - ModelDeployment -
    デプロイメント内のモデル名（モデルプレイグラウンドからコピー）。

    ![](./media/image47.png)

2.  ターミナルから+++dotnet run+++を実行します。

    ![](./media/image48.png)

3.  ブラウザを開き、+++localhost:5802+++ にアクセスしてください。

4.  アプリにタスクをいくつか追加してください。

    ![](./media/image49.png)

    ![](./media/image50.png)

5.  **「Microsoft Agent Framework」**を選択すると、チャットが開きます。

    ![](./media/image51.png)

6.  「Hi」または「+++ What are my current
    tasks+++」のようなメッセージを入力して、エージェントからの応答を受け取ります。

    ![](./media/image52.png)

    ![](./media/image53.png)

7.  それでは、+++ azd up+++
    を実行してアプリの変更をデプロイしてください。

    ![](./media/image54.png)

    ![](./media/image55.png)

8.  エンドポイントを開いて、Azure でアプリを表示します。

    ![](./media/image56.png)

## まとめ

この実習では、シンプルな.NETアプリケーションを取り上げ、そこにAIを組み込む方法を学びました。
