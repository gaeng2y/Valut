그래서 한 가지 언급할 가치가 있는 것은 명령어 쿼리 분리라고 불리는 것입니다. 매우 잘 알려져 있고 대중적인 아이디어입니다.

그래서 우리가 물체에 대해 작동할 때마다, 우리는 모든 호출을 쿼리와 명령이라고 불리는 두 개의 다른 개념으로 분리합니다.

따라서 명령은 작업이나 변경을 요청할 때 보내는 것입니다.

예를 들어, 한 생물체의 공격 값을 2로 설정하려고 합니다.

수정할 필요가 있음을 명시하는 명령을 보내는 것입니다.

수정해야 할 것은 공격 값과 설정하려는 새 값입니다.

2와 같습니다.

또 다른 것은 쿼리와 쿼리는 반드시 아무것도 바꾸지 않고 정보를 요구하는 것입니다.

그래서 당신은 시스템이 당신에게 특정한 값을 주기를 요구할 뿐입니다. 당신의 공격 값을 저에게 주세요.

그래서 SQS라는 것이 있습니다. 즉, 명령과 쿼리를 보내는 별도의 방법이 있다는 것을 의미합니다.

예를 들어 특정 클래스의 필드에 직접 액세스하는 대신 메시지를 보내는 것입니다.

필드의 내용을 알려달라는 메시지를 보내거나, 필드를 이 값으로 설정하라는 명령을 보내거나.

또한 책임 체인 덕분에 다른 청취자가 이 명령을 전송할 수 있으며 실제 명령의 동작이나 쿼리를 무시할 수 있습니다.