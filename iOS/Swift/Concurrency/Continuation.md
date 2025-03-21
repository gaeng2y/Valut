Swift concurrency에서는 **continuation**이라는 중요한 개념이 등장한다.
async한 메서드를 호출할 때 Swift concurrency에서는 스레드의 제어권을 포기하는 suspended라는 현상이 발생한다.

그러다 보니 다시 제어권을 돌려 받았을때 어디서부터 실행할지를 아는것이 필요한데 async(비동기)함수의 경우 **heap에 suspension point에서 실행**하는데 필요한 함수 컨텍스트들을 저장해서 제어권을 돌려받았을때 어디서실행할지를 알수있게 됩니다. 이것을 continuation이라고 부른다.

heap에다가 저장하는 이유는 어떠한 변수가 await의 suspension point이전에 정의되었는데 await이후에 사용되는 경우는 해당 변수가 사라지면안되고 저장이되어야합니다. stack에 저장하게 된다면 다른 await메서드에 의해 replace되는 경우에 변수자체가 사라지기때문에 이러한 변수(suspension point이전에 정의되고 suspension point 이후에 사용되는 변수)를 추적하기 위해서 heap의 async frame에 저장되게 됩니다

> continuation을 통해 일시정지된 함수의 상태를 추적해 어디서부터 재개할지 알 수 있습니다, 이후에 사용될 변수중에 이전에 정의된 변수의 상태가 어떠했는지를 heap의 async frame을 통해 알 수 있게 됩니다

기존 GCD에서는 thread가 block되면 task를 다른 thread로 보내는 full thread context switching이라는 꽤나 비용이큰 현상이 발생하는데 continuation을 활용하면 Full Thread Context Switching대신 function call정도의 비용으로 비동기 메서드를 호출하고 실행할 수 있게됩니다

GCD와 swift concurrency를 공부할때 기억해야할점은 swift Concurrency와 GCD는 상호 배타적이지 않다는겁니다. 또한 swift concurrency를 사용하면서 GCD도 여전히 필요한 경우도 있을 수도 있습니다

