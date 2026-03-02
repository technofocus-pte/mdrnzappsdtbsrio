# 실습 2 - Azure의 에이전틱 앱으로 기존 ASP.NET 앱을 강화하기

이 실습은 기존 데이터 기반 ASP.NET Core CRUD 애플리케이션에 에이전트
기능을 추가하는 벙법을 시연합니다. Microsoft Agent Framework를 사용해
수행됩니다

만약 웹 애플리케이션에 쇼핑, 호텔 예약, 데이터 관리 같은 유용한 기능이
이미 있다면, 그 기능을 도구로 포장해 에이전트 기능을 웹 애플리케이션에
추가하는 것은 비교적 간단합니다.

## 작업 1: ASP.NET 앱을 로컬에서 실행하기

이 연습에서는 로컬 시스템에서 애플리케이션을 실행하게 됩니다.

1.  **C:** **extract**에서 **Labfiles zip** 파일을 선택하세요.

2.  VS Code를 여세요. **Open Folder**를 선택하세요.

![](./media/image1.png)

3.  **C:/Labfiles**에서
    **app-service-agentic-semantic-kernel-ai-foundry-agent-main** 포더를
    선택하세요.

![](./media/image2.png)

4.  팝업에서 **Yes, I trust the authors**를 선택하세요.

![](./media/image3.png)

5.  상단 메뉴에서 3개의 점을 선택하세요. VSCode 터미널을 열고 앱을
    실행하려면 **Terminal** -\> **New Terminal**을 선택하세요.

![](./media/image4.png)

6.  열리면 터미널에서 아래 명령을 실행하여 지정된 버전의 NuGet 패키지
    Microsoft.Agents.AI를 현재 프로젝트에 설치하세요.

+++dotnet add package Microsoft.Agents.AI --version
1.0.0-preview.251204.1+++

![](./media/image5.png)

7.  다음으로 +++dotnet build+++ 명령어를 실행하여 프로젝트를 구축하세요.

![](./media/image6.png)

8.  다음으로 +++dotnet build+++ 명령어를 실행하여 프로젝트를 구축하세요.

![](./media/image7.png)

9.  명령어를 실행하면 브라우저를 열고 +++<http://localhost:5280>+++를
    여세요

![](./media/image8.png)

10. 왼쪽에서 Microsoft Agent Framework 옵션을 선택하면 설정되지 않았다면
    메시지가 뜹니다. 이 실습에서 나중에 업데이트할 예정입니다.

![](./media/image9.png)

11. VS Code 터미널에서 Ctrl+C를 눌러 애플리케이션을 중지하세요.

## 에이전트 코드를 검토하기

에이전트는 제공자에서 서비스 (Program.cs)로 초기화되어 해당 Blazor
구성요소에 주입됩니다.

AgentFrameworkProvider는 *Services/AgentFrameworkProvider.cs*에서
초기화됩니다. 초기화 코드는 다음과 같은 역할을 합니다:

- Azure OpenAI using the AzureOpenAIClient에서 IChatClient를 생성합니다.

- (*Tools/TaskCrudTool.cs*내에서) CRUD애플리케이션의 기능을 캡술화하는
  TaskCrudTool 인스턴스를 빋습니다. 도구 방법의 설명 속성은 에이전트가
  이를 호출하는 방법을 결정하는 데 도움을 줍니다.

- AIFunctionFactory.Create()에 등록된 지침과 도구를 사용하여
  CreateAIAgent()를 사용하여 AI 에이전트를 생성합니다.

- 에이전트가 내비게이션 전반에 걸쳐 대화를 지속할 수 있도록 스레드를
  생성합니다.

// Create IChatClient

IChatClient chatClient = new AzureOpenAIClient(

new Uri(endpoint),

new DefaultAzureCredential())

.GetChatClient(deployment)

.AsIChatClient();

// Get TaskCrudTool instance from service provider

var taskCrudTool = sp.GetRequiredService\<TaskCrudTool\>();

// Create agent with tools

var agent = chatClient.CreateAIAgent(

instructions: @"You are an agent that manages tasks using CRUD
operations.

Use the provided functions to create, read, update, and delete tasks.

Always call the appropriate function for any task management request.

Don't try to handle any requests that are not related to task
management.

When handling requests, if you're missing any information, don't make it
up but prompt the user for it instead.",

tools:

\[

AIFunctionFactory.Create(taskCrudTool.CreateTaskAsync),

AIFunctionFactory.Create(taskCrudTool.ReadTasksAsync),

AIFunctionFactory.Create(taskCrudTool.UpdateTaskAsync),

AIFunctionFactory.Create(taskCrudTool.DeleteTaskAsync)

\]);

// Create thread for this scoped instance (persists across navigation)

var thread = agent.GetNewThread();

return (agent, thread);

사용자가 메시지를 보낼 때마다, Blazor 컴포넌트
(*Components/Pages/AgentFrameworkAgent.razor*)는 사용자 입력과 에이전트
스레드를 함께 Agent.RunAsync()를 호출합니다. 에이전트 스레드는 채팅
기록을 관리합니다.

var response = await this.Agent.RunAsync(sentInput, this.agentThread);

## 작업 2: 샘플 애플리케이션을 배포하기

샘플에는 Azure 개발자 CLI (AZD) 템플릿이 포하되어 있는데, 이 템플릿은
관리형 ID 를 가진 앱 서비스 앱을 생성하고 샘플 애플리케이션을
배포합니다.

1.  터미널에서 +++az login+++ 명령어를 실행하세요.

![](./media/image10.png)

2.  Work or school account를 선택하세요.

![](./media/image11.png)

3.  로그인 자격 증명으로 로그인하세요

- Username - +++@lab.CloudPortalCredential(User1).Username+++

- Password - +++@lab.CloudPortalCredential(User1).Password+++

![](./media/image12.png)

![](./media/image13.png)

4.  **No, this app only**를 선택하세요.

![](./media/image14.png)

5.  구독을 수락하려면 **Enter**를 선택하세요.

![](./media/image15.png)

6.  다음 단계에서 +++azd auth login+++ 명령어를 실행하고 로그되었는지
    확인하세요.

![](./media/image16.png)

![](./media/image17.png)

7.  완료되면 리소스를 배포하려면 +++azd up+++를 실행하세요.

![](./media/image18.png)

8.  환경 이름을 <+++envt@lab.LabInstance.Id>+++로 입력하세요.

![](./media/image19.png)

![](./media/image20.png)

9.  구독을 선택하세요. Press 구독이 하나만 목록에 있으니 **Enter**를
    누르세요.

![](./media/image21.png)

10. 화살표 키를 이동하고 리소스 그룹으로 **ResourceGroup1**을
    선택하세요.

![](./media/image22.png)

11. 리소스은 생성되고 완료하는 데 약 10분 정도 걸립니다.

![](./media/image23.png)

12. 완료 후 터미널에 **endpoint**가 표시됩니다.

![](./media/image24.png)

13. Azure에서 애플리케이션에 접근하려면 엔드포인트 URL을 열어보세요.

![](./media/image25.png)

![](./media/image26.png)

![](./media/image27.png)

![](./media/image28.png)

## 작업 3: Microsoft Foundry 리소스를 생성하고 구성하기

이 연습에서는 Microsoft Foundry 리소스를 생성하고 사용할 모델을 배포할
것입니다.

1.  +++https://ai.azure.com+++를 열고 자격 증명으로 로그인하세요.
    상단의 **New Foundry** 라디오 버튼이 활성화되어 있는지 획인하세요.

- 사용자 이름 - +++@lab.CloudPortalCredential(User1).Username+++

- 비밀번호 - <+++@lab.CloudPortalCredential(User1).Password>+++

2.  Select a project to continue 대화상자에서 **Create a new project**를
    선택하세요.

![](./media/image29.png)

3.  프로젝트 이름을 <+++proj@lab.LabInstance.Id>+++로 입력하고 Resource
    group을 as **ResourceGroup1**로 선택하고 **Create**를 선택하세요.

![](./media/image30.png)

![](./media/image31.png)

4.  다시 **Old Foundry**로 전환하세요. Overview 페이지에서 Azure
    OpenAI를 선택하고 엔드포인트 값을 복사하고 메모장에 저장하세요.

![](./media/image32.png)

5.  왼쪽 창에서 Models + endpoints를 선택하세요. **+ Deploy model** -\>
    **Deploy base model**을 선택하세요.

![](./media/image33.png)

6.  gpt-4.1을 선택하고 **Confirm**을 클릭하세요.

![](./media/image34.png)

7.  **Deploy**를 클릭하세요.

![](./media/image35.png)

8.  모델 이름을 메모장에 복사해서 붙여넣으세요.

![](./media/image36.png)

## 작업 4: 필요한 권한 할당하기

이번 작에서는 Foundry 리소스에 필요한 권한을 할당할 것입니다.

1.  +++https://portal.azure.com+++를 열고 프롬프트되면 자격 증명으로
    로그인하세요. Home 페이지에서 **Foundry**를 선택하세요.

    - 사용자 이름- +++@lab.CloudPortalCredential(User1).Username+++

    - 비밀번호 - +++@lab.CloudPortalCredential(User1).Password+++

![](./media/image37.png)

1.  이전 작업에서 생성한 Foundry 리소스를 선택하세요.

![](./media/image38.png)

2.  왼쪽 창에서 **Access control (IAM)**를 선택하세요.

![](./media/image39.png)

3.  **+ Add** -\> **Add role assignment**를 선택하세요.

![](./media/image40.png)

4.  +++Cognitive Services OpenAI User+++를 검색하고 선택하세요.

![](./media/image41.png)

5.  앞으로 나아가려면 **Next**를 선택하세요.

![](./media/image42.png)

6.  **Managed identity**를 선택하고 **Select members**를 선택하세요.

![](./media/image43.png)

7.  **Managed identity**를 **Foundry project**로 선택하고
    <proj@lab.LabInstance.Id>를 선택하세요.

![](./media/image44.png)

8.  다음 2개의 화면에서 **Review + assign**을 선택하세요.

![](./media/image45.png)

> ![](./media/image46.png)

## 작업 5: 샘플 애플리케이션에 연결 변수를 구성하기

이 작업에서는 샘플 애플리케이션을 연결 변수를 구성할 것입니다.

1.  VS Code에서 *appsettings.json*을 여세요. 앞서 Foundry 포털에서
    복사한 겂을 사용해 다음 변수들을 설정하세요.

- AzureOpenAIEndpoint - Azure OpenAI 엔드포인트 (고전적인 Foundry
  포털에서 복사함).

- ModelDeployment – 배포에 모델 이름 (모델 playground에서 복사함).

> ![](./media/image47.png)

2.  터미널에서 +++dotnet run+++를 실행하세요

![](./media/image48.png)

3.  브라우저를 열고 +++localhost:5802+++로 이동하세요

4.  앱에 몇 가지 작업을 추가하세요.

![](./media/image49.png)

> ![](./media/image50.png)

5.  이제 왼쪽 창에서 **Microsoft Agent Framework**를 선택하면 채팅
    박스가 열리는 것을 볼 수 있습니다.

![](./media/image51.png)

6.  Hi 또는 +++What are my current tasks+++라는 메시지를 입역하고
    에이전트에서 답변을 받으세요.

![](./media/image52.png)

![](./media/image53.png)

7.  이제, 앱 변화를 배포하려면 +++azd up+++를 실행하세요.

![](./media/image54.png)

![](./media/image55.png)

8.  Azure에서 앱을 보려면 엔드포인프를 여세요.

![](./media/image56.png)

## 요약

이 실습에서는 간단한 .NET 앱을 가져와서 AI를 도입하는 법을 배웠습니다.
