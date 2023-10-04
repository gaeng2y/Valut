[위키백과](https://ko.wikipedia.org/wiki/%EC%8B%B1%EA%B8%80%ED%84%B4_%ED%8C%A8%ED%84%B4)에 따르면

> 소프트웨어 디자인 패턴에서 싱글턴 패턴(Singleton pattern)을 따르는 클래스는, 생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나이고 최초 생성 이후에 호출된 생성자는 최초의 생성자가 생성한 객체를 리턴한다.
> 
> 이와 같은 디자인 유형을 싱글턴 패턴이라고 한다.
> 
> 주로 공통된 객체를 여러개 생성해서 사용하는 DBCP(DataBase Connection Pool)와 같은 상황에서 많이 사용된다.

싱글턴 패턴을 사용하게되면 **유일한 객체**를 만들 수 있다

싱글턴은 특정 클래스의 인스턴스가 오직 하나임을 보장하는 객체이다

유일한 객체를 만든다는 말은 다양한 객체들이 모두 같은 인스턴스와 소통할 수 있다는 말이다

싱글턴의 대안으로 인스턴스를 생성하여 각각의 객체들에게 의존성을 주입하는 방법을 생각해볼 수 있다

그러나 싱글턴 패턴을 사용하게 되면 하나보다 많은 인스턴스를 생성하는 것 자체를 방지할 수 있다

# [Singleton Pattern은 언제 사용함?](https://github.com/gaeng2y/Design-Pattern/blob/main/Yagom/Creational/Singleton/Singleton.md#singleton-pattern%EC%9D%80-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%A8)

주로 앱 전체에 걸쳐서 유일한 객체를 만들기 위해서 사용한다

**전역적으로 접근할 수 있는 유일한 객체**인 셈이다

여러 개의 객체가 같은 객체로부터 동일한 상태의 정보를 받아와야할 때 사용할 수 있다

예를 들면 어떤 사용자의 어떤 정보나 앱 전반에 걸쳐서 공유되어야하는 데이터들이 있다

우리가 자주 사용하던 UserDefaults / NotificationCenter / URLSession 등이 있다