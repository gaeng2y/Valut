컴파일, 링크, 실행
# Swift 구조
## 컴파일 (Compile)

사람이 이해할 수 있는 문자를 사용하며 코드를 작성하고, 이 문자들을 컴퓨터 컴파일러를 통해서, 컴퓨터가 알아볼 수 있고 0과 1로 구성되어 있는 이진수 코드로 변환하는 과정을 의미

> codes => 1010...

## 링크 (Link)

컴파일 과정을 통해 얻은, 이진수로 구성되어 있는 컴파일 코드를 실행 가능한 파일(바이너리)로 변환하기 위하여, 프레임워크와 라이브러리 등을 접목시켜 변환하는 작업을 의미

> codes => 0101... + Framework (+ Libraries ... ) => Variables, Components ...

## 빌드 (Build)

Compile 과정과 Link 과정을 아우르는 과정을 빌드라고 한다.

# LLVM 컴파일러 구조

LLVM(Love Level Virtual Machine)은 컴파일러의 기반구조다. 프로그램을 컴파일 타임, 링크 타임, 런타임 상황에서 프로그램의 작성 언어에 상관없이 최적화를 쉽게 구현할 수 있도록 구성되어 있다.

![llvm-infra](https://user-images.githubusercontent.com/20410193/110127199-b10a1c00-7e08-11eb-99ee-30f85ce63246.png)