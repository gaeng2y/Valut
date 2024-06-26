우선 순위가 높은 태스크가 READY상태 (실행 가능)로 바뀌었지만 더 낮은 우선순위의 태스크가 CPU를 점유하고 있어서 실행되지 못하는 상태

운영 체제에서 중요한 개념 중 하나는 작업 스케줄링이다. 선착순, 라운드 로빈, 우선 순위 기반 스케줄링 등과 같은 몇 가지 스케줄링 방법이 있습니다. 각 스케줄링 방법에는 장단점이 있다. 짐작할 수 있듯이, 우선 순위 반전은 우선 순위 기반 스케줄링에 따라 옵니다. 기본적으로, 그것은 OS에서 우선 순위 기반 스케줄링을 사용할 때 가끔 발생하는 문제이다. 우선 순위 기반 스케줄링에서, 다른 작업에는 다른 우선 순위가 주어지므로 가능한 경우 우선 순위가 낮은 작업에 개입할 수 있습니다.

따라서, 우선 순위 기반 스케줄링에서, 낮은 우선 순위 작업(L)이 실행 중이고 우선 순위가 높은 작업(H)도 실행되어야 하는 경우, 우선 순위가 낮은 작업(L)은 우선 순위가 높은 작업(H)에 의해 선점될 것이다. 이제, 우선 순위가 낮고 우선 순위가 높은 작업 모두 각각의 작업을 수행하기 위해 공통 리소스(동일한 파일 또는 장치에 대한 액세스)를 공유해야 한다고 가정합니다. 이 경우, 자원 공유와 작업 동기화가 필요하기 때문에, 그러한 시나리오를 처리하기 위해 몇 가지 방법/기술을 사용할 수 있습니다. 우선 반전에 대한 우리의 주제를 위해, 뮤텍스와 말하는 동기화 방법을 언급합시다. 뮤텍스를 요약하자면, 작업은 중요한 섹션(CS)에 들어가기 전에 뮤텍스를 획득하고 중요한 섹션(CS)을 종료한 후 뮤텍스를 해제합니다. CS에서 실행되는 동안, 작업은 이 공통 자원에 접근한다. 이것에 대한 자세한 내용은 [여기에서](https://www.geeksforgeeks.org/g-fact-70/) 참조할 수 있습니다. 이제, L과 H 모두 공통의 중요 섹션(CS)을 공유한다고 말하세요. 즉, 이 CS에 동일한 뮤텍스가 필요합니다.

우선 순위 반전에 대한 논의에 따라, 몇 가지 시나리오를 살펴봅시다.  
1) L은 실행 중이지만 CS에 있지 않습니다. H는 실행해야 합니다. H는 L을 선점합니다. H는 실행을 시작합니다. H는 제어를 포기하거나 해제합니다. L은 재개하고 실행을 시작합니다.  
2) L은 CS에서 실행되고 있습니다; H는 실행해야 하지만 CS에서 실행되지 않습니다; H는 L을 선점합니다; H는 실행을 시작합니다; H는 제어를 포기합니다; L은 재개하고 실행을 시작합니다.  
3) L은 CS에서 실행되고 있다; H는 또한 CS에서 실행되어야 한다; H는 L이 CS에서 나올 때까지 기다린다; L은 CS에서 나온다; H는 CS에 들어가서 실행을 시작한다.

위의 시나리오는 우선 순위 반전의 문제를 보여주지 않는다는 점에 유의하십시오(시나리오 3도 아닙니다). 기본적으로, 우선 순위가 낮은 작업이 공유 CS에서 실행되지 않는 한, 우선 순위가 높은 작업은 그것을 선점할 수 있다. 하지만 L이 공유 CS에서 실행되고 H도 CS에서 실행해야 한다면, H는 L이 CS에서 나올 때까지 기다린다. 그 아이디어는 LS가 CS에 있는 동안 H가 오랫동안 기다리지 않도록 충분히 작아야 한다는 것이다. 그것이 CS 코드를 작성하는 데 신중한 고려가 필요한 이유이다. 위의 시나리오 중 어느 것에서도, 작업이 설계에 따라 실행되고 있기 때문에 우선 순위 반전(즉, 우선 순위 반전)이 발생하지 않았다.

이제 중간 우선순위의 또 다른 과제를 추가합시다. 이제 작업 우선 순위는 L < M < H 순서입니다. 우리의 예에서, M은 같은 중요한 섹션(CS)을 공유하지 않는다. 이 경우, 다음 작업 순서가 실행되면 '우선 순위 반전' 문제가 발생할 것이다.

4) L은 CS에서 실행되고 있습니다. H는 또한 CS에서 실행되어야 합니다. H는 L이 CS에서 나올 때까지 기다립니다. M은 L을 방해하고 실행을 시작합니다. M은 완료될 때까지 실행되고 제어를 포기합니다. L은 CS가 끝날 때까지 재개하고 실행을 시작합니다. H는 CS에 들어가 실행을 시작합니다.  
L도 H도 M과 CS를 공유하지 않는다는 점에 유의하세요.

여기서, 우리는 M의 실행이 L과 H의 실행을 지연시켰다는 것을 알 수 있다. 정확히 말하자면, H는 우선 순위가 높고 M과 CS를 공유하지 않지만, H는 M을 기다려야 했다. 이것은 CS를 공유하지 않았음에도 불구하고 M과 H의 우선 순위가 뒤집혔기 때문에 우선 순위 기반 일정이 예상대로 작동하지 않은 곳이다. 이 문제는 우선 반전이라고 불린다. 이게 바로 우선순위 반전이었어! 우선 순위 기반 스케줄링이 있는 시스템에서 우선 순위가 높은 작업은 이 문제에 직면할 수 있으며 예상치 못한 행동/결과를 초래할 수 있습니다. 범용 OS에서는 성능이 느려질 수 있다. RTOS에서, 그것은 더 심각한 결과를 초래할 수 있다. 가장 유명한 '우선 순위 반전' 문제는 화성 패스파인더에서 일어난 일이었다.

만약 우리에게 문제가 있다면, 이것에 대한 해결책이 있어야 한다. 우선 순위 반전의 경우, 우선 순위 상속 등과 같은 다양한 해결책이 있습니다.


다시 알아보기

LP가 리소스를 점유할 때 문제가 발생 HP가 기다린다면 