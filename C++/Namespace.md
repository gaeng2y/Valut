```c++
int main() {
    std::cout << "Hello, World!!" << std::endl;
    return 0;
}
```

위 코드에서 std의 정체부를 알아보자. std는 C++ 표준 라이브러리의 모든 함수, 객체 등이 정의된 **이름 공간**이다.

네임스페이스란 어떤 정의된 객체에 대해 **어디 소속인지** 지정해주는 것과 동일하다.

같은 이름의 객체라도 소속된 네임 스페이스가 다르면 다른 객체이다.

```c++
std::cout
```

위의 경우 `std`라는 네임스페이스에 정의되어 있는 `cout`을 의미한다.

### 이름 공간 정의 방법
header1.h, header2.h 파일이 있다 가정해보자.

```c++
namespace header1 {
  int foo();
  void bar();
}

namespace header2 {
  int foo();
  void bar();
}
```

위 코드에서 `header1`에 있는 `foo`는 `header1`라는 이름 공간에 살고 있는 `foo`가 되고, `header2`에 있는 `foo`의 경우 `header2`라는 이름 공간에 살고 있는 `foo`가 된다.

```c++
#include "header1.h"

namespace header1 {
  int func() {
    foo(); // 알아서 header1::foo()가 실행된다.
    header2::foo();
  }
} // namespace header1
```

```c++
#include "header1.h"
#include "header2.h"

using header1::foo;
int main() {
  foo(); // header1에 있는 함수를 호출
}
```

```c++
#include "header1.h"
#include "header2.h"

using namespace header1;
int main() {
  foo(); // header1에 있는 함수를 호출
  bar(); // header1에 있는 함수를 호출
}
```