# 실습 1 - Java를 위한 GitHub Copilot App Modernization - Azure로 마이그레이션

AWS/RabbitMQ/Postgres에서 Java용 GitHub Copilot App Modernization을
사용해 샘플 Java 애플리케이션 "asset-manager"를 평가하고 Azure Blob
Storage, Azure Service Bus, 및 PostgreSQL용 Azure Database로
마이그레이션하는 과정을 안내하는 실습입니다.

개요

이 실습은 Java용 GitHub Copilot App Modernization을 사용하여 Java웹 +
워커 애플리케이션을 Azure로 평가하고 마이그레이션하는 방법을 시연합니다.
이 실습은 각 주요 단계마다 검증 지점이 있는 집중 작업으로 조직되어
있습니다.

Java용 GitHub Copilot App Modernization (Java용 App Modernization 또는
AppMod라고도 함)은 Visual Studio Code Extension 및 GitHub Copilot을
사용하여 앱 평가, 계획 및 코드 복원을 지원합나디. 반복적인 작업을
자동화하여 개발자의 신뢰를 높이고 Azure 마이그레이션과 지속적인 최적화를
가속화합니다.

**프로젝트 소개**

이 애플리케이션은 **Web**과 **Worker** 두 개의 하위 모듈로 구성됩니다.
두 경우 모두 스토리지 서비스와 메시지 큐 사용 가능을 포함하고 있습니다.

**원래 인프라**

이 프로젝트는 다음과 같은 원래 인프라를 사용합니다:

- 비밀번호 기반 인증 (접근 키/비밀 키)을 사용하여 이미지 저장을 위한 AWS
  S3 

- 비밀번호 기반 인증을 사용하여 메시지 큐잉을 위한 RabbitMQ 

- 비밀번호 기반 인증을 사용하여 메타데이터 저장을 위한 PostgreSQL
  데이터베이스

**원래 아키텍처**

![](./media/image1.png)

**마이그레이션 후 예산되는 인프라**

마이그레이션 후에는 다음 Azure 서비스를 사용할 것입니다

- 관리형 ID 인증을 사용하여 이미지 저장을 위한 Azure Blob Storage

- 관리형 ID 인증을 사용하여 메시지 큐일을 위한 Azure Service Bus

- 관리형 ID 인증을 사용하여 메타데이터 저자을 위한 PostgreSQL용 Azure
  Database

**마이그레이션된 아키텍처**

관리형 ID 기반 인증

![](./media/image2.png)

## 필수 구성 요소

이 실습을 진행하려면 Github Copilot이 활성화된 Github 계정이 필요합니다.
만약 없다면 여기에서 하나의 양식을 생서하세요 -
+++https://github.com/signup+++

## 작업 1: GitHub Copilot App Modernization 확장을 설치하기

이 작업에서는 이 실습 실생에 필요한 확장을 VS Code에 설치해야 합니다.

1.  VS Code를 여세요. 왼쪽 창에서 **Extensions**를 선택하세요.

    ![](./media/image3.png)

2.  +++GitHub Copilot app modernization+++를 검색하고 선택하고
    **Install**을 선택하세요.

    ![](./media/image4.png)
    
    ![](./media/image5.png)

GitHub Copilot App Modernization 확장 기능은 앱을 쉽게 현대화하는 데
도움이 됩니다.

## 작업 2: 기존 애플리케이션을 이해하기

기존 코드를 살펴보면서 PostgreSQL, AWS 및 RabbitMQ의 구성을 이해하세요.

## 작업 3: Java 애플리케이션을 평가하기

1.  VSCode를 열고 **Select Folder**를 클릭하세요.

    ![](./media/image6.png)

2.  **C:\Labfile**에서 **GitHub-Copilot-App-Modernization-for-java**
    폴더를 선택하고 **Select folder**를 클릭하세요.

    ![](./media/image7.png)

3.  폴더가 열려면 **Yes, I trust the authors** 옵션을 선택하세요.

    ![](./media/image8.png)

4.  VSCode 오른쪽 하단에 GitHub Copilot 아이콘을 볼 수 있습니다.

    ![](./media/image9.png)

5.  **Continue with GitHub** 옵션을 선택하세요.

    ![](./media/image10.png)

6.  **GitHub id** 및 **Authorize Visual Studio Code**를 사용하여
    로그인하세요.

7.  이제 GitHub Copilot이 활성화된 것을 볼 수 있습니다.

    ![](./media/image11.png)

8.  왼쪽 창에서 **extension** – **GitHub Copilot for App
    Modernization**을 선택하세요.

    ![](./media/image12.png)

9. GitHub 채팅에서 Auto 옆 드롭다운을 선택해서 모델을 선택하세요.
    **Claude Sonnet 4.5**는 앱 현대화에 가장 적합합니다. 프리미멈 GitHub
    라이선스가 있다면 선택할 수 있고 그렇지 않으면 **Claude Haiku
    4.5**를 선택할 수 있습니다.

    ![](./media/image13.png)

10. GitHub Copilot App Modernization의 Quick Start 섹션에서 **Start
    Assessment**를 선택하세요.

    ![](./media/image14.png)

11. 평가 진행 상황을 확인하세요.

    ![](./media/image15.png)

12. 평가가 시작되어 점진적으로 진행되는 것을 볼 수 있습니다.

    ![](./media/image16.png)

    ![](./media/image17.png)

13. 이 과정은 약 5분 정도 걸립니다. 완료되면 **Assessment Report**는
    아래 스크린샷과 같이 표시됩니다.

    **Application Information** 섹션에는 신청서에 대한 기본 정보가 나열되어
    있습니다.
    
    ![](./media/image18.png)

14. **Issue Summary** 섹션은 이슈를 두 가지 범주로 나눕니다 **– Cloud
    Readiness** 및 **Java Upgrade**.

    ![](./media/image19.png)

15. 이슈에 대한 자세한 내용을 보려면 아래로 스크롤하세요. 이 경우
    **Cloud Readiness category**에는 **9 issues**가 없고 Java Upgrade
    카테고리에는 없는 문제가 있습니다.

    ![](./media/image20.png)

16. 각 섹션을 확장해 문제가 무엇인지, 그리고 어떻게 핼결할 수 있는지
    살펴보세요.

17. 첫 번째 문제는 **Database Migration (PostgreSQL)**에 관한 것이며
    **Solution**은 **Azure Database for PostgreSQL**로 마이그레이션하는
    것입니다

    ![](./media/image21.png)

18. PostgreSQL 데이터베이스 찾기 옵션을 더 확인하면 영향받는 파일 수와
    자세한 설명을 확인할 수 잇습니다.

    ![](./media/image22.png)

19. 다음은 Messaging Service Migration (Spring AMQP Rabbit MQ)입니다

    ![](./media/image23.png)

20. 세부 사항을 이해하려면 **Spring RabbitMQ usage found in code**를
    확장하세요.

    ![](./media/image24.png)

21. 영향을 받는 파일과 세부 정보를 확인하려면 **Spring AMQP dependency**
    파일을 확장하세요.

    ![](./media/image25.png)

22. **RabbitMQ connection string, username or password found in
    configuration file**을 확장하세요

    ![](./media/image26.png)

23. 다음 문제는 **Storage Migration (AWS S3)**으로 솔루션은 **Migrate
    from AWS S3 to Azure Blob Storage**입니다.

    ![](./media/image27.png)

24. 세부 사항을 보려면 **AWS S3 usage found**를 확장하세요.

    ![](./media/image28.png)
    
    ![](./media/image29.png)

25. 세부 사항을 보려면 **AWS S3 dependency usage found**를 획장하세요.

    ![](./media/image30.png)

## 작업 4: Azure Database for PostgreSQL Flexible Server로 마이그레이션하기

1.  **Database Migration**부터 시작합니다. Assessment Report의 Issues
    -\> Issue Category에 나열된 Database Migration(PostgreSQL) 이슈에
    대해 **Run Task**를 선택하세요.

![](./media/image31.png)

2.  작업이 시작되고 자세한 내용은 채팅창에 체워집니다.

> ![](./media/image32.png)

3.  작업이 진행되는 동안 GitHub Copilot은 먼저 **실행 계획을** 새우고
    적절한 **파일**에 **변경**을 시작합니다. 각 **행동**은 **채팅**창에
    설명되어 있습니다. **파일**에 **변경**이 있을 때 세부 정보가
    채워지고 사용자 변경 사항을 **유지**하거나 **페기**할지 확인을
    요청합니다. 또한, 획인이 필요한 곳에서는 계속을 위해 사용자 확인을
    요청합니다. **Continue**를 클릭해야만 진행됩니다.

![](./media/image33.png)

![](./media/image34.png)

4.  채팅에 나오는 파일 이름을 선택해서 여세요. 파일은 추가를 위한 녹색
    색상 코드, 삭제를 위한 빨간색 코드가 있습니다.

5.  변경 사항을 꼼꼼히 검토하고 이해한 후, 파일을 계속 보관해 변경
    사항을 **유지**하세요.

![](./media/image35.png)

![](./media/image36.png)

![](./media/image37.png)

6.  아래 스크린샷에서 Aws 설정이 제거되고 Azure Storage 블롭이 추가된
    것을 볼 수 있습니다.

![](./media/image38.png)

![](./media/image39.png)

7.  각 파일을 검토하고 **Keep**을 클릭하세요. 프롬프트되면
    **Continue**를 선택하세요.

![](./media/image40.png)

![](./media/image41.png)

![](./media/image42.png)

![](./media/image43.png)

![](./media/image44.png)

![](./media/image45.png)

![](./media/image46.png)

8.  **Todos** 섹션은 작성된 계획의 진행 상황과 몇 개가 완료되었는지, 몇
    개가 완료되었는지에 대한 상태를 보여줍니다.

![](./media/image47.png)

![](./media/image48.png)

![](./media/image49.png)

![](./media/image50.png)

![](./media/image51.png)

![](./media/image52.png)

![](./media/image53.png)

![](./media/image54.png)

![](./media/image55.png)

9.  첫 번째 작업이 완료되었고 **Migration compete** 메시지가 표시됩니다.

![](./media/image56.png)

10. 또한, 요약 파일은 프로젝트에 저장됩니다.

![](./media/image57.png)

## 작업 5: AWS S3에서 Azure Blob Storage로 마이그레이션하기

다음으로 AWS S3를 Azure Blob Storage로 마이그레이션하는 과정을 시작하게
됩니다.

1.  **Storage Migration (AWS S3)** 이슈에 대한 Run Task를 선택하세요.

![](./media/image58.png)

2.  GitHub Copilot은 \#appmod-run-task by kbId:
    s3-to-azure-blob-storage를 실행합니다.

![](./media/image59.png)

3.  GHCP는 MCP Server를 사용하여 run appmod-run-task,
    appmod-fetch-knowledgebase,appmod-search-file 등 다양한 작업을 계속
    실행할 것입니다. 각 단계에서 수동으로 **Continue**를 클릭하여 허용,
    확인, 진행 요청 시 진행하세요. Copilot Agent는 애플리케이션 현대화를
    촉진하기 위해 다양한 도구를 사용합니다. 각 도구 사용 여부는 Continue
    버튼을 클릭하여 확인해야 할 수도 있습니다.

4.  제안된 코드 변경사항을 검토한 후 Keep를 클릭하고 적용하세요.

![](./media/image60.png)

5.  이 작업이 완료되면 다음 작업으로 넘어갈 수 있습니다.

## 작업6: AMQP RabbitMQ에서 Azure Service Bus로 마이그레이션하기

> 애플리케이션 asset-manager는 메시지 큐잉을 위해 Spring AMQP와
> RabbitMQ를 사용합니다. 대신 Azure Service Bus로 넘어가겠습니다.

1.  이번 워크숍에서는 **Messaging Service Migration**에 대해
    살펴보겠습니다. **Migrate from AMQP RabbitMQ to Azure Service
    Bus**를 할 것입니다.

![](./media/image61.png)

6.  **Run Task**를 클릭하세요.

![](./media/image62.png)

7.  GitHub Copilot는 #appmod-run-task by kbId:
    amqp-rabbitmq-servicebus를 실행합니다.

8.  GHCP는 MCP Server를 사용하여
    appmod-run-task, appmod-fetch-knowledgebase,appmod-search-file 등
    다양한 작업을 계속 실행할 것입니다. 각 단계에서 수동으로
    **Continue**를 반복해서 클릭하여 허용, 확인 및 진행하세요. The
    Copilot Agent는 애플리케이션 현대화를 촉진하기 위해 다양한 도구를
    사용합니다. 각 도구 사용 여부는 Continue 버튼을 클릭하여 획인해야
    합니다.

9.  제안된 코드 변경사항을 검토한 후 **Keep**를 눌러 적용할 수 있습니다.

이 작업이 완료되면 코드의 모든 업데이트를 확인할 수 있습니다.

Azure에 배포할 준비가 된 완전한 마이그레이션 코드는 여기에서 확인할 수
있습니다 - **C:\Labfiles\\ java-migration-copilot-samples-expected**.
코드를 살펴보면 원본 코드와 달리 AWS S3, RabbitMQ, PostgreSQL
데이터베이스에 대한 모든 참조가 명확히 제거되고 Azure Blob Storage,
Azure Service Bus, PostgreSQL용 Azure Database로 대체된 것을 알 수
있습니다.

## 요약

- 자동 평가를 실행하고 Copilot이 생성한 마이그레이션을 데이터베이스,
  저장소, 매시장에 적용했습니다.

- 생성된 코드 변경 사항과 마이그레이션 요약을 검토하고 수락했스니다.

