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

2.  Open VS Code. Select **Open Folder**.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image1.png)

3.  Select **app-service-agentic-semantic-kernel-ai-foundry-agent-main**
    folder from **C:/Labfiles**.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image2.png)

4.  Select **Yes, I trust the authors** in the pop up.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image3.png)

5.  Select the 3 dots in the top menu. Select **Terminal** -\> **New
    Terminal** to open the VSCode terminal and execute the app.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image4.png)

6.  Once opened, execute the below command in the terminal to install
    the NuGet package Microsoft.Agents.AI of the specified version into
    the current project.

    +++dotnet add package Microsoft.Agents.AI --version 1.0.0-preview.251204.1+++

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image5.png)

7.  Next, execute the command +++dotnet build+++ to build the project.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image6.png)

8.  Next, execute the command +++dotnet run+++ to execute the project.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image7.png)

9.  Once the command gets executed, open a browser and open
    +++http://localhost:5280+++

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image8.png)

10. Select the Microsoft Agent Framework option from the left and you
    can see that you get a message that it is not configured. We will
    update it later in this lab.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image9.png)

11. Press Ctrl+C in the VS Code terminal in order to stop the
    application.

## Review the agent code

The agent is initialized as a service (in Program.cs) in a provider and
injected into the respective Blazor component.

The AgentFrameworkProvider is initialized
in *Services/AgentFrameworkProvider.cs*. The initialization code does
the following:

- Creates an IChatClient from Azure OpenAI using the AzureOpenAIClient.

- Gets the TaskCrudTool instance that encapsulates the functionality of
  the CRUD application (in *Tools/TaskCrudTool.cs*).
  The Description attributes on the tool methods help the agent
  determine how to call them.

- Creates an AI agent using CreateAIAgent() with instructions and tools
  registered via AIFunctionFactory.Create().

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
(in *Components/Pages/AgentFrameworkAgent.razor*)
calls Agent.RunAsync() with the user input and the agent thread. The
agent thread keeps track of the chat history.

var response = await this.Agent.RunAsync(sentInput, this.agentThread);

## Task 2: Deploy the sample application

The sample contains an Azure Developer CLI (AZD) template, which creates
an App Service app with managed identity and deploys your sample
application.

1.  In the terminal, execute the command +++az login+++

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image10.png)

2.  Select Work or school account.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image11.png)

3.  Login using your login credentials

    - Username - +++@lab.CloudPortalCredential(User1).Username+++

    - TAP - +++@lab.CloudPortalCredential(User1).AccessToken+++

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image12.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image13.png)

4.  Select **No, this app only**.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image14.png)

5.  Select **Enter** to accept the subscription.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image15.png)

6.  As the next step, execute the command, +++azd auth login+++ and
    ensure that you are logged in.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image16.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image17.png)

7.  Once this is done, execute +++azd up+++ to deploy the resources.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image18.png)

8.  Enter the environment name as +++envt@lab.LabInstance.Id+++

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image19.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image20.png)

9.  Select the subscription. Press **Enter** to do so since you will
    have only one subscription listed.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image21.png)

10. Move your arrow key and select **ResourceGroup1** as the Resource
    group.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image22.png)

11. The resources will get created and it will take around 10 minutes to
    complete.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image23.png)

12. Once completed, the **endpoint** is listed in the terminal.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image24.png)

13. Open the endpoint url to access the application from Azure.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image25.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image26.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image27.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image28.png)

## Task 3: Create and configure the Microsoft Foundry resource

In this exercise, you will create the Microsoft Foundry resource and
deploy a model to be used.

1.  Open +++https://ai.azure.com+++ and sign in using your credentials.
    Ensure that the top **New Foundry** radio button is set to active.

    - Username - +++@lab.CloudPortalCredential(User1).Username+++

    - TAP - +++@lab.CloudPortalCredential(User1).AccessToken+++

2.  In the Select a project to continue dialog, select **Create a new
    project**.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image29.png)

3.  Enter the project name as +++proj@lab.LabInstance.Id+++, select
    the Resource group as **ResourceGroup1** and then select **Create**.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image30.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image31.png)

4.  Switch back to **Old Foundry**. From the Overview page, select Azure
    OpenAI, copy the endpoint value and save it in a notepad.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image32.png)

5.  Select Models + endpoints from the left pane. Select **+ Deploy
    model** -> **Deploy base model**.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image33.png)

6.  Select gpt-4.1 and click on **Confirm**.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image34.png)

7.  Click on **Deploy**.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image35.png)

8.  Copy and paste the model name to the notepad.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image36.png)

## Task 4: Assign required permissions

In this exercise, you will assign the required permissions to the
Foundry resource.

1.  Open +++https://portal.azure.com+++ and login using your credentials
    if prompted. Select **Foundry** from the Home page.

    - Username - +++@lab.CloudPortalCredential(User1).Username+++

    - TAP - +++@lab.CloudPortalCredential(User1).AccessToken+++

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image37.png)

1.  Select the Foundry resource that you created in the last task.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image38.png)

2.  Select **Access control (IAM)** from the left pane.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image39.png)

3.  Select **+ Add** -\> **Add role assignment**.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image40.png)

4.  Search for and select +++Cognitive Services OpenAI User+++.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image41.png)

5.  Select **Next** to move ahead.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image42.png)

6.  Select **Managed identity** and then select **Select members**.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image43.png)

7.  Select **Managed identity** as **Foundry project** and then select
    the proj@lab.LabInstance.Id.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image44.png)

8.  Select **Review + assign** in the next two screens.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image45.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image46.png)

## Task 5: Configure connection variables in your sample application

In this task, you will configure your sample application with the
connection variables.

1.  From VS Code, open *appsettings.json*. Using the values you copied
    earlier from the Foundry portal, configure the following variables.

    - AzureOpenAIEndpoint - Azure OpenAI endpoint (copied from the classic
    Foundry portal).

    - ModelDeployment - Model name in the deployment (copied from the model
    playground).

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image47.png)

2.  From the terminal execute +++dotnet run+++

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image48.png)

3.  Open a browser and navigate to +++localhost:5820+++

	>[!Alert] It can take up to **10 minutes** for the localhost address to finish building. 

4.  Add few tasks in the app.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image49.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image50.png)

5.  Now, select **Microsoft Agent Framework** from the left pane and you
    can see that the chat opens up.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image51.png)

6.  Enter Hi or any message like +++What are my current tasks+++ and get
    the response from the agent.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image52.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image53.png)

7.  Now, execute +++azd up+++ to deploy the app changes.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image54.png)

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image55.png)

8.  Open the endpoint to view the app in the Azure.

    ![](https://raw.githubusercontent.com/technofocus-pte/mdrnzappsdtbsrio/refs/heads/main/Labguides/Lab%202/media/image56.png)

## Summary

In this lab, you have taken a simple .NET app and learnt to induce AI
into it.
