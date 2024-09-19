세 메소드 모두 구독하는 메소드이다.
## subscribe
onNext, onError, onCompleted 가 모두 온다.
## bind
onNext만 온다.
즉, error나 completed가 발생하지 않는 스트림에서 구독할 때 사용하면 된다. (ui event 같은)
## drive
Driver를 구독할 때 사용한다.
bind와 비슷하지만 메인스케줄러, 그리고 share 같은 기능을 사용할 때 사용한다.
