Operation은 이렇게 `수행할 작업이 캡슐화된 객체`라고 생각해볼 수 있을 것 같다.

수행해야할 코드 블록들을 객체화하면 어떤 장점이 있을까? 객체지향 프로그래밍의 장점과 거의 비슷하다.

1. 재사용에 용이하다.
2. 타입 간의 관계를 만들어줄 수 있다.
3. 다양한 프로퍼티를 활용해볼 수 있다.
4. 스케쥴링에 용이하다.

Operation을 사용하면 동시성 프로그래밍과 관련된 모든 작업들은 `Operation이라는 객체`로서 만들어지게 된다.

![oq](https://user-images.githubusercontent.com/73867548/153408968-b42522db-d1e1-43e4-b7a2-8e101cab45d7.png)
Order Bar라는 Operation Queue에 