# Lab 2 - Enhance an existing ASP.NET app to an agentic app on Azure

This lab demonstrates how to add agentic capability to an existing
data-driven ASP.NET Core CRUD application. It does this using Microsoft
Agent Framework

If your web application already has useful features, like shopping,
hotel booking, or data management, it's relatively straightforward to
add agent functionality to your web application by wrapping those
functionalities as tools. In this lab, you start with a simple to-do
list app. By the end, you'll be able to create, update, and manage tasks
with an agent in an App Service app.

**Duration** - 50 minutes

## Task 1: Execute the ASP.NET app locally

In this exercise, you will execute the application in the local system.

1.  From **C:** **extract** the **Labfiles zip** file.

2.  Open VS Code. Select **Open Folder**.

    ![](./media/image1.png)

3.  Select **app-service-agentic-semantic-kernel-ai-foundry-agent-main**
    folder from **C:/Labfiles**.

    ![](./media/image2.png)

4.  Select **Yes, I trust the authors** in the pop up.

    ![](./media/image3.png)

5.  Select the 3 dots in the top menu. Select **Terminal** -\> **New
    Terminal** to open the VSCode terminal and execute the app.

    ![](./media/image4.png)

6.  Once opened, execute the below command in the terminal to install
    the NuGet package Microsoft.Agents.AI of the specified version into
    the current project.

    +++dotnet add package Microsoft.Agents.AI --version
1.0.0-preview.251204.1+++

    ![](./media/image5.png)

7.  Next, execute the command +++dotnet build+++ to build the project.

    ![](./media/image6.png)

8.  Next, execute the command +++dotnet run+++ to execute the project.

    ![](./media/image7.png)

9.  Once the command gets executed, open a browser and open
    +++http://localhost:5280+++

    ![](./media/image8.png)

10. Select the Microsoft Agent Framework option from the left and you
    can see that you get a message that it is not configured. We will
    update it later in this lab.

    ![](./media/image9.png)

11. Press Ctrl+C in the VS Code terminal in order to stop the
    application.

## Review the agent code

The agent is initialized as a service (in Program.cs) in a provider and
injected into the respective Blazor component.

The AgentFrameworkProvider is initialized
in *Services/AgentFrameworkProvider.cs*. The initialization code does
the following:

- Creates an IChatClient from Azure OpenAI using the AzureOpenAIClient.

- Gets the TaskCrudTool instance that encapsulates the functionality of
  the CRUD application (in *Tools/TaskCrudTool.cs*).
  The Description attributes on the tool methods help the agent
  determine how to call them.

- Creates an AI agent using CreateAIAgent() with instructions and tools
  registered via AIFunctionFactory.Create().

- Creates a thread for the agent to persist conversation across
  navigation.

```no-copy
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
```

Each time the user sends a message, the Blazor component
(in *Components/Pages/AgentFrameworkAgent.razor*)
calls Agent.RunAsync() with the user input and the agent thread. The
agent thread keeps track of the chat history.

var response = await this.Agent.RunAsync(sentInput, this.agentThread);

## Task 2: Deploy the sample application

The sample contains an Azure Developer CLI (AZD) template, which creates
an App Service app with managed identity and deploys your sample
application.

1.  In the terminal, execute the command +++az login+++

    ![](./media/image10.png)

2.  Select Work or school account.

    ![](./media/image11.png)

3.  Login using your login credentials

    - Username - +++@lab.CloudPortalCredential(User1).Username+++

    - Password - +++@lab.CloudPortalCredential(User1).Password+++

    ![](./media/image12.png)

    ![](./media/image13.png)

4.  Select **No, this app only**.

    ![](./media/image14.png)

5.  Select **Enter** to accept the subscription.

    ![](./media/image15.png)

6.  As the next step, execute the command, +++azd auth login+++ and
    ensure that you are logged in.

    ![](./media/image16.png)

    ![](./media/image17.png)

7.  Once this is done, execute +++azd up+++ to deploy the resources.

    ![](./media/image18.png)

8.  Enter the environment name as +++envt@lab.LabInstance.Id+++

    ![](./media/image19.png)

    ![](./media/image20.png)

9.  Select the subscription. Press **Enter** to do so since you will
    have only one subscription listed.

    ![](./media/image21.png)

10. Move your arrow key and select **ResourceGroup1** as the Resource
    group.

    ![](./media/image22.png)

11. The resources will get created and it will take around 10 minutes to
    complete.

    ![](./media/image23.png)

12. Once completed, the **endpoint** is listed in the terminal.

    ![](./media/image24.png)

13. Open the endpoint url to access the application from Azure.

    ![](./media/image25.png)

    ![](./media/image26.png)

    ![](./media/image27.png)

    ![](./media/image28.png)

## Task 3: Create and configure the Microsoft Foundry resource

In this exercise, you will create the Microsoft Foundry resource and
deploy a model to be used.

1.  Open +++https://ai.azure.com+++ and sign in using your credentials.
    Ensure that the top **New Foundry** radio button is set to active.

    - Username - +++@lab.CloudPortalCredential(User1).Username+++

    - Password - +++@lab.CloudPortalCredential(User1).Password+++

2.  In the Select a project to continue dialog, select **Create a new
    project**.

    ![](./media/image29.png)

3.  Enter the project name as <+++proj@lab.LabInstance.Id>+++, select
    the Resource group as **ResourceGroup1** and then select **Create**.

    ![](./media/image30.png)

    ![](./media/image31.png)

4.  Switch back to **Old Foundry**. From the Overview page, select Azure
    OpenAI, copy the endpoint value and save it in a notepad.

    ![](./media/image32.png)

5.  Select Models + endpoints from the left pane. Select **+ Deploy
    model** -> **Deploy base model**.

    ![](./media/image33.png)

6.  Select gpt-4.1 and click on **Confirm**.

    ![](./media/image34.png)

7.  Click on **Deploy**.

    ![](./media/image35.png)

8.  Copy and paste the model name to the notepad.

    ![](./media/image36.png)

## Task 4: Assign required permissions

In this exercise, you will assign the required permissions to the
Foundry resource.

1.  Open +++https://portal.azure.com+++ and login using your credentials
    if prompted. Select **Foundry** from the Home page.

    - Username - +++@lab.CloudPortalCredential(User1).Username+++

    - Password - +++@lab.CloudPortalCredential(User1).Password+++

    ![](./media/image37.png)

1.  Select the Foundry resource that you created in the last task.

    ![](./media/image38.png)

2.  Select **Access control (IAM)** from the left pane.

    ![](./media/image39.png)

3.  Select **+ Add** -\> **Add role assignment**.

    ![](./media/image40.png)

4.  Search for and select +++Cognitive Services OpenAI User+++.

    ![](./media/image41.png)

5.  Select **Next** to move ahead.

    ![](./media/image42.png)

6.  Select **Managed identity** and then select **Select members**.

    ![](./media/image43.png)

7.  Select **Managed identity** as **Foundry project** and then select
    the proj@lab.LabInstance.Id.

    ![](./media/image44.png)

8.  Select **Review + assign** in the next two screens.

    ![](./media/image45.png)

    ![](./media/image46.png)

## Task 5: Configure connection variables in your sample application

In this task, you will configure your sample application with the
connection variables.

1.  From VS Code, open *appsettings.json*. Using the values you copied
    earlier from the Foundry portal, configure the following variables.

    - AzureOpenAIEndpoint - Azure OpenAI endpoint (copied from the classic
    Foundry portal).

    - ModelDeployment - Model name in the deployment (copied from the model
    playground).

    ![](./media/image47.png)

2.  From the terminal execute +++dotnet run+++

    ![](./media/image48.png)

3.  Open a browser and navigate to +++localhost:5802+++

4.  Add few tasks in the app.

    ![](./media/image49.png)

    ![](./media/image50.png)

5.  Now, select **Microsoft Agent Framework** from the left pane and you
    can see that the chat opens up.

    ![](./media/image51.png)

6.  Enter Hi or any message like +++What are my current tasks+++ and get
    the response from the agent.

    ![](./media/image52.png)

    ![](./media/image53.png)

7.  Now, execute +++azd up+++ to deploy the app changes.

    ![](./media/image54.png)

    ![](./media/image55.png)

8.  Open the endpoint to view the app in the Azure.

    ![](./media/image56.png)

## Summary

In this lab, you have taken a simple .NET app and learnt to induce AI
into it.

