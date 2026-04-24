# ラボ1 - Java向けGitHub Copilotアプリの近代化 - Azureへの移行

GitHub Copilot App Modernization for Java
を活用し、サンプルJavaアプリケーション「asset-manager」を
AWS/RabbitMQ/Postgres から Azure Blob Storage、Azure Service Bus、および
Azure Database for PostgreSQL へと評価・移行する手順を解説するラボです。

概要

本ハンズオンラボでは、GitHub Copilot App Modernization for Java
を使用して、Java Web + Worker アプリケーションを評価し、Azure
へ移行する手順を実演します。本ラボは、各主要ステップの後に検証ポイントを設けた、小規模かつ集中的なタスク構成となっています。

GitHub Copilot App Modernization for Java（通称：App Modernization for
Java、または AppMod）は、Visual Studio Code 拡張機能および GitHub
Copilot
を活用し、アプリケーションの評価、計画策定、およびコードの修正作業を支援します。本ツールは反復的なタスクを自動化することで、開発者の自信を高め、Azure
への移行作業やその後の継続的な最適化プロセスを加速させます。

**プロジェクトについて**

**Web**と**Worker**という2つのサブモジュールで構成されています。どちらのモジュールにも、ストレージサービスとメッセージキューを使用する機能が含まれています。

**元のインフラ**

このプロジェクトでは、以下の既存のインフラストラクチャを使用しています。

- 画像の保存にはAWS
  S3を使用し、パスワード認証（アクセスキー／シークレットキー）を利用します。

- パスワード認証を使用したメッセージキューイングのためのRabbitMQ

- メタデータ保存用のPostgreSQLデータベース、パスワード認証を使用

**オリジナルの建築**

![](./media/image1.png)

**移行後の想定されるインフラストラクチャ**

移行後、プロジェクトでは以下のAzureサービスを使用します。

- イメージ保存にはAzure Blob
  Storageを使用し、マネージドID認証を使用する。

- マネージドID認証を使用したメッセージキューイングのためのAzure Service
  Bus

- メタデータストレージには、マネージドID認証を使用するAzure Database for
  PostgreSQLを使用します。

**移行されたアーキテクチャ**

マネージドIDベース認証

![](./media/image2.png)

## 前提条件

## このラボを実施するには、GitHub Copilotが有効化されたGitHubアカウントが必要です。アカウントをお持ちでない場合は、こちら（+++https://github.com/signup+++）から作成してください。

## タスク 1: GitHub Copilot App Modernization 拡張機能をインストールする

このタスクでは、このラボの実行に必要な拡張機能をVS
Codeにインストールします。

1.  VS Codeを開きます。左側のペインから**「Extensions」**を選択します。

    ![](./media/image3.png)

2.  「+++GitHub Copilot app modernization+++」を検索して選択し、
    **「Install」**を選択します。

    ![](./media/image4.png)

    ![](./media/image5.png)

GitHub
Copilotのアプリ近代化拡張機能は、アプリを簡単に近代化するのに役立ちます。

## タスク2：既存のアプリケーションを理解する

既存のコードを読み解き、PostgreSQL、AWS、および
RabbitMQの構成を理解してください。

## タスク3：Javaアプリケーションを評価する

1.  **Cドライブから、LabfilesのZIP**ファイルを解凍します。

2.  VSCodeを開き、 **「Select Folder」**をクリックします。

    ![](./media/image6.png)

3.  **C:\Labfile**から**GitHub-Copilot-App-Modernization-for-java**フォルダーを選択し、
    **\[Select folder\]**をクリックします。

    ![](./media/image7.png)

4.  フォルダが開いたら、 **「Yes, I trust the
    authors**」を選択してください。

    ![](./media/image8.png)

5.  VS Codeの右下隅に、GitHub Copilotのアイコンが表示されます。

    ![](./media/image9.png)

6.  **「Continue with GitHub」**オプションを選択してください。

    ![](./media/image10.png)

7.  **GitHub ID**を使用してログインし、 **Visual Studio Code
    を承認**してください。

8.  これでGitHub Copilotが有効になっていることが確認できます。

    ![](./media/image11.png)

9.  左側のペインから、**拡張機能**「 **GitHub Copilot for App
    Modernization」**を選択します。

    ![](./media/image12.png)

10. GitHubチャットの「Auto」の横にあるドロップダウンを選択して、モデルを選択してください。アプリの近代化には、
    **Claude Sonnet
    4.5**が最適です。GitHubのプレミアムライセンスをお持ちの場合は、それを選択できます。そうでない場合は、
    **Claude Haiku 4.5**を選択してください。

    ![](./media/image13.png)

11. GitHub Copilot App Modernization のクイックスタートセクションから、
    **\[Start Assessment\]**を選択します。

    ![](./media/image14.png)

12. 評価の進捗状況を確認してください。

    ![](./media/image15.png)

13. 評価は段階的に開始され、進行していくことがわかります。

    ![](./media/image16.png)

    ![](./media/image17.png)

14. 完了までには約5分かかります。完了すると、以下のスクリーンショットのように**Assessment
    Report**が表示されます。

「**Application
Information」**セクションには、申請に関する基本情報が記載されています。

    ![](./media/image18.png)

15. 「**Issue Summary**」セクションでは、課題を「**Cloud
    Readiness**と「**Java
    Upgrade**」の2つのカテゴリに分類して記載しています。

    ![](./media/image19.png)

16. 下にスクロールして、問題の詳細を確認してください。この場合、**Cloud
    Readiness category**
    **9件の問題**がありますが、Javaアップグレードカテゴリには問題がありません。

    ![](./media/image20.png)

17. 各セクションを展開して、問題点とその解決策を確認してください。

18. 最初の問題は**Database
    Migration（PostgreSQL）**に関するもので、**解決策(Solution)**は**Azure
    Database for PostgreSQL**に移行することです**。**

    ![](./media/image21.png)

19. PostgreSQLデータベースの検出オプションをさらに確認すると、影響を受けたファイルの数と詳細な説明が表示されます。

    ![](./media/image22.png)

20. 次はMessaging Service Migration（Spring AMQP Rabbit MQ）です。

    ![](./media/image23.png)

21. **Spring RabbitMQ usage found
    in**を展開して、その詳細を理解してください。

    ![](./media/image24.png)

22. **Spring AMQP
    dependency**を展開すると、影響を受けるファイルとその詳細が表示されます。

        ![](./media/image25.png)

23. **RabbitMQ connection string, username or password found in
    configuration file**を展開します。

    ![](./media/image26.png)

24. 次の課題は**Storage Migration（AWS
    S3）**であり、その解決策は**Migrate from AWS S3 to Azure Blob
    Storage**です。

    ![](./media/image27.png)

25. **AWS S3 usage found**を展開すると、詳細が表示されます。

    ![](./media/image28.png)

    ![](./media/image29.png)

26. **AWS S3 dependency usage found**を展開すると、詳細が表示されます。

    ![](./media/image30.png)

## タスク4：Azure Database for PostgreSQL Flexible Serverへの 移行

1.  **Database Migration**から始めます。Assessment
    Reportの「Issues」→「Issue Category」に表示されている「Database
    Migration（ PostgreSQL）」課題に対して、 **「Run
    Task」**を選択してください。

    ![](./media/image31.png)

2.  タスクが開始され、詳細がチャットウィンドウに表示されます。

    ![](./media/image32.png)

3.  タスクが進行するにつれて、GitHub Copilot
    はまず実行**計画を立て、**次に適切な**ファイルへの変更**を**開始します**。各**アクションは**チャット**ウィンドウ**に表示されます。**ファイル**に**変更**が加えられると、詳細が表示され、変更を**保持するか破棄するかを**ユーザーに**確認するよう求められます。**また、確認が必要な箇所では、続行するかどうかをユーザーに確認するよう求められます**。
    「Continue」**をクリックした場合のみ、処理が続行されます。

    ![](./media/image33.png)

    ![](./media/image34.png)

4.  チャットに表示されるファイル名を選択して開いてください。追加の場合は緑色、削除の場合は赤色で色分けされています。

5.  変更内容をよく確認し、理解した上で、変更内容を反映させるためにファイルを**保存(keep)**し続けてください。

    ![](./media/image35.png)

    ![](./media/image36.png)

    ![](./media/image37.png)

6.  下のスクリーンショットで、AWSの設定が削除され、Azure Storage
    Blobが追加されていることが確認できます。

    ![](./media/image38.png)

    ![](./media/image39.png)

7.  各ファイルを確認し、
    **「Keep」**をクリックしてください。プロンプトが表示されたら、
    **「Continue」**を選択してください。

    ![](./media/image40.png)

    ![](./media/image41.png)

    ![](./media/image42.png)

    ![](./media/image43.png)

    ![](./media/image44.png)

    ![](./media/image45.png)

    ![](./media/image46.png)

8.  「**Todos」**セクションには、作成したプランの進捗状況と、完了済みおよび終了済みの項目数が表示されます。

    ![](./media/image47.png)

    ![](./media/image48.png)

    ![](./media/image49.png)

    ![](./media/image50.png)

    ![](./media/image51.png)

    ![](./media/image52.png)

    ![](./media/image53.png)

    ![](./media/image54.png)

    ![](./media/image55.png)

9.  最初のタスクが完了し、**Migration
    compete**メッセージが表示されました。

    ![](./media/image56.png)

10. また、概要(Summary)ファイルはプロジェクトに保存されます。

    ![](./media/image57.png)

## タスク5： AWS S3からAzure Blob Storageへの移行

次に、AWS S3からAzure Blob Storageへの移行プロセスを開始します。

1.  **「Storage Migration（AWS S3）」**の課題に対して「Run
    Task」を選択してください。

    ![](./media/image58.png)

2. GitHub Copilot は、kbId「s3-to-azure-blob-storage」を指定して
#appmod-run-task を実行します。

    ![](./media/image59.png)

3.  GHCPは引き続き、MCPサーバーを使用して appmod-run-task、appmod-fetch-knowledgebase、appmod-search-file、およびその他のタスクを実行します。各ステップの実行中、画面にプロンプトが表示された場合は、手動で「Continue」を繰り返しクリックし、許可・確認を行って処理を進め  てください。Copilot Agentは、アプリケーションのモダナイゼーションを支援するために様々なツールを使用します。各ツールの使用にあたっては、「 Continue」ボタンをクリックして確認を行う必要がある場合があります。

4.  提案されたコード変更内容を確認し、「Keep」をクリックして適用してください。

    ![](./media/image60.png)

5.  これが完了したら、次のタスクに進むことができます。

## タスク6：AMQP RabbitMQからAzure Service Busへの移行

    アプリケーション資産管理ツールは、メッセージキューイングにSpring AMQPとRabbitMQを使用しています。代わりにAzure Service Busに移行しましょう。

1.  このワークショップでは、**Messaging Service
    Migration**について見ていきます。**Migrate from AMQP RabbitMQ to
    Azure Service Bus**を行います。

    ![](./media/image61.png)

2.「Run Task」をクリックします。

    ![](./media/image62.png)

3.  GitHub Copilot は、kbId「amqp-rabbitmq-servicebus」を指定して
    #appmod-run-task を実行します。

4.  GHCP は、MCP Server
    を使用して、appmod-run-task、appmod-fetch-knowledgebase、appmod-search-file、およびその他のタスクの実行を継続します。各ステップの進行中、処理の許可、確認、および続行を行うため、手動で「**Continue**」をクリックし続けてください。Copilot
    Agent
    は、アプリケーションのモダナイゼーションを支援するために様々なツールを使用します。各ツールの使用には、「Continue」ボタンをクリックすることによる確認が必要です。

5.  提案されたコード変更を確認し、
    **「Keep」**をクリックしてください。 それらを適用する。

    これらの作業が完了すると、コード内のすべての更新内容を確認できるようになります。

    Azureにデプロイする準備が整った移行済みのコード全体は、こちらから入手できます -
    **C:\Labfiles\\ java-migration-copilot-samples-expected**
    を参照してください。コードを確認すると、元のコードと比較して、AWS
    S3、RabbitMQ、PostgreSQL
    データベースへのすべての参照が明確に削除され、それぞれ Azure Blob
    Storage、Azure Service Bus、Azure Database for PostgreSQL
    に置き換えられていることがわかります。

## まとめ

- 自動評価を実行し、データベース、ストレージ、メッセージングに関してCopilotが生成した移行を適用しました。

- 生成されたコード変更と移行概要を確認し、承認しました。
