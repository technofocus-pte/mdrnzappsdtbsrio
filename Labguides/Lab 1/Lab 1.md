# Lab 1 - GitHub Copilot App Modernization for Java - Migrate to Azure

A lab that walks through assessing and migrating the sample Java
application "asset-manager" from AWS/RabbitMQ/Postgres to Azure Blob
Storage, Azure Service Bus, and Azure Database for PostgreSQL using
GitHub Copilot App Modernization for Java.

**Overview**

This hands-on lab demonstrates assessing and migrating a Java web +
worker application to Azure using GitHub Copilot App Modernization for
Java. The lab is organized as small focused tasks with verification
points after each major step.

GitHub Copilot App Modernization for Java, also referred to as App
Modernization for Java or AppMod, assists with app assessment, planning
and code remediation using a Visual Studio Code Extension and GitHub
Copilot. It automates repetitive tasks, boosting developer confidence
and speeding up the Azure migration and ongoing optimization.

**About the Project**

This application consists of two sub-modules, **Web** and **Worker**.
Both of them contain functions of using storage service and message
queue.

**Original Infrastructure**

The project uses the following original infrastructure:

- AWS S3 for image storage, using password-based authentication (access
  key/secret key)

- RabbitMQ for message queuing, using password-based authentication

- PostgreSQL database for metadata storage, using password-based
  authentication

**Original Architecture**

![](./media/image1.png)

**Expected infrastructure after migration**

After migration, the project will use the following Azure services

- Azure Blob Storage for image storage, using managed identity
  authentication

- Azure Service Bus for message queuing, using managed identity
  authentication

- Azure Database for PostgreSQL for metadata storage, using managed
  identity authentication

**Migrated Architecture**

Managed identity based authentication

![](./media/image2.png)

## Prerequisites

In order to execute this lab, you will need a Github account with Github
Copilot enabled. If you do not have one, please create one form here -
+++https://github.com/signup+++

## Task 1: Install the GitHub Copilot App Modernization extension

In this task, you will install the extension in the VS Code that is
required for this lab execution.

1.  Open the VS Code. Select **Extensions** from the left pane.

    ![](./media/image3.png)

2.  Search for and select +++GitHub Copilot app modernization+++ and
    then select **Install**.

    ![](./media/image4.png)

    ![](./media/image5.png)

The GitHub Copilot App Modernization extension is what will help in
modernizing the apps easily.

## Task 2: Understand the existing application

Go through the existing code to understand the configuration of
PostgreSQL, AWS and RabbitMQ.

## Task 3: Assess Your Java Application

1.  From **C:** **extract** the **Labfiles zip** file.

2.  Open VSCode and click on **Select Folder**.

    ![](./media/image6.png)

3.  Select **GitHub-Copilot-App-Modernization-for-java** folder from
    **C:\Labfile** and click on **Select folder**.

    ![](./media/image7.png)

4.  Once the folder opens, select **Yes, I trust the authors** option.

    ![](./media/image8.png)

5.  At the right bottom of VSCode, you can see the GitHub Copilot icon.

    ![](./media/image9.png)

6.  Select the **Continue with GitHub** option.

    ![](./media/image10.png)

7.  Login using your **GitHub id** and **Authorize Visual Studio Code**.

8.  You can now see that the GitHub Copilot is enabled.

    ![](./media/image11.png)

9.  From the left pane, select the **extension** – **GitHub Copilot for
    App Modernization**.

    ![](./media/image12.png)

10. Select the drop down next to Auto in the GitHub chat to select the
    model. **Claude Sonnet 4.5** works best for App modernization. If
    you have a premium GitHub license, you can select that. Else, you
    can select **Claude Haiku 4.5**.

    ![](./media/image13.png)

11. From the Quick Start section of GitHub Copilot App Modernization,
    select **Start Assessment**.

    ![](./media/image14.png)

12. Check the progress of the assessment.

    ![](./media/image15.png)

13. You can see that the assessment starts and proceeds gradually.

    ![](./media/image16.png)

    ![](./media/image17.png)

14. This takes around 5 minutes to complete. Once done, the **Assessment
    Report** is displayed as in the screenshot below.

    The **Application Information** section lists the basic information
about the application.

    ![](./media/image18.png)

15. The **Issue Summary** section lists down the issues in 2 categories
    **– Cloud Readiness** and **Java Upgrade**.

    ![](./media/image19.png)

16. Scroll down to see the details on the issues. In this case, you have
    **9 issues** under the **Cloud Readiness category** and there are
    non under Java Upgrade category.

    ![](./media/image20.png)

17. Expand each section to see what the issue is and how it can be
    resolved.

18. First issue is on the **Database Migration (PostgreSQL)** and the
    **Solution** is to migrate to **Azure Database for PostgreSQL**

    ![](./media/image21.png)

19. When you further check the PostgreSQL database found option, you
    will see how many files are impacted and also a detailed
    explanation.

    ![](./media/image22.png)

20. The next one is Messaging Service Migration (Spring AMQP Rabbit MQ)

    ![](./media/image23.png)

21. Expand **Spring RabbitMQ usage found in code** to understand the
    details under that.

    ![](./media/image24.png)

22. Expand **Spring AMQP dependency** found to see the files affected
    and the details of them.

    ![](./media/image25.png)

23. Expand **RabbitMQ connection string, username or password found in
    configuration file**

    ![](./media/image26.png)

24. The next issue is the **Storage Migration (AWS S3)** to which the
    solution is **Migrate from AWS S3 to Azure Blob Storage**.

    ![](./media/image27.png)

25. Expand **AWS S3 usage found** to view the details.

    ![](./media/image28.png)

    ![](./media/image29.png)

26. Expand **AWS S3 dependency usage found** to view the details.

    ![](./media/image30.png)

## Task 4: Migrate to Azure Database for PostgreSQL Flexible Server

1.  We will start with the **Database Migration**. Select **Run Task**
    against the Database Migration(PostgreSQL) issue that is listed
    under Issues -\> Issue Category in the Assessment Report.

    ![](./media/image31.png)

2.  The task will start and the details will get populated in the chat
    window.

    ![](./media/image32.png)

3.  As the task continues, the GitHub Copilot first **plans** the
    **execution** and then **starts** making **changes** to the
    appropriate **files**. Each **action** is described in the **chat**
    window. As it makes **changes** to the **files**, the details are
    populated and asks for **confirmation** from the user to **keep**
    the changes or **discard** them. Also, at places where confirmation
    is required, it asks for the user confirmation to continue. Only if
    you click on **Continue**, it will proceed.

    ![](./media/image33.png)

    ![](./media/image34.png)

4.  Select the file name that gets listed in the chat, to open it. The
    file has got green color code for addition and red for deletion.

5.  Ensure to go through the changes, understand and then continue to
    **keep** the file to have the changes.

    ![](./media/image35.png)

    ![](./media/image36.png)

    ![](./media/image37.png)

6.  In the below screenshot, you can see that the Aws configuration are
    removed and the Azure Storage blob is added.

    ![](./media/image38.png)

    ![](./media/image39.png)

7.  Review each file and click **Keep**. Then select **Continue** if
    prompted.

    ![](./media/image40.png)

    ![](./media/image41.png)

    ![](./media/image42.png)

    ![](./media/image43.png)

    ![](./media/image44.png)

    ![](./media/image45.png)

    ![](./media/image46.png)

8.  The **Todos** section shows the progress on the created plan and the
    status on how many are finished and how many are completed.

    ![](./media/image47.png)

    ![](./media/image48.png)

    ![](./media/image49.png)

    ![](./media/image50.png)

    ![](./media/image51.png)

    ![](./media/image52.png)

    ![](./media/image53.png)

    ![](./media/image54.png)

    ![](./media/image55.png)

9.  The first task is complete and it shows a **Migration compete**
    message.

    ![](./media/image56.png)

10. Also , the Summary file is saved in the project.

    ![](./media/image57.png)

## Task 5: Migrate from AWS S3 to Azure Blob Storage

Next, you will start the migration process of AWS S3 to Azure Blob
Storage.

1.  Select Run Task against the issue **Storage Migration (AWS S3).**

    ![](./media/image58.png)

2.  GitHub Copilot runs \#appmod-run-task by kbId:
    s3-to-azure-blob-storage.

    ![](./media/image59.png)

3.  GHCP will continue to run appmod-run-task,
    appmod-fetch-knowledgebase,appmod-search-file and other tasks using
    the MCP Server. During each step, please manually click **Continue**
    repeatedly to allow, confirm and proceed if prompted. The Copilot
    Agent uses various tools to facilitate application modernization.
    Each tool's usage might require confirmation by clicking the
    Continue button.

4.  Review the proposed code changes and click Keep to apply them.

    ![](./media/image60.png)

5.  Once this is completed, you can move to the next task.

## Task 6: Migrate from AMQP RabbitMQ to Azure Service Bus

The Application asset-manager uses Spring AMQP with RabbitMQ for message queuing. Let's move to Azure Service Bus instead.

1.  For this part of the workshop, we will take a look at
    the **Messaging Service Migration**. We will **Migrate from AMQP
    RabbitMQ to Azure Service Bus**.

    ![](./media/image61.png)

6.  Click **Run Task**.

    ![](./media/image62.png)

7.  GitHub Copilot runs #appmod-run-task by kbId:
    amqp-rabbitmq-servicebus

8.  GHCP will continue to
    run appmod-run-task, appmod-fetch-knowledgebase,appmod-search-file and
    other tasks using the MCP Server. During each step, please manually
    click **Continue** repeatedly to allow, confirm and proceed. The
    Copilot Agent uses various tools to facilitate application
    modernization. Each tool's usage requires confirmation by clicking
    the Continue button.

9.  Review the proposed code changes and click **Keep** to apply them.

Once these are completed, you should be able to see all the updates in
your code.

The complete migrated code ready to be deployed to Azure is available
here - **C:\Labfiles\java-migration-copilot-samples-expected**. Go
through the code and as against the original code, you can see that all
the references to AWS S3, RabbitMQ and PostgreSQL database are clearly
removed and replaced with Azure Blob Storage, Azure Service Bus and
Azure Database for PostgreSQL correspondingly.

## Summary

- You ran an automated assessment and applied Copilot-generated
  migrations for database, storage, and messaging.

- You reviewed and accepted the generated code changes and migration
  summary.
