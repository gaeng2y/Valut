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
**LLVM**은 아키텍처별로 분리된 모듈식 **미들엔드-백엔드를 중점**으로 하고 있다.

1. 프론트엔드가 여러가지 프로그래밍 언어들을 중간 표현 코드로 번역하고
2. LLVM은 그 중간 표현 코드를 각각의 아키텍처에 맞게 최적화하여 실행이 가능한 형태로 바꾼다.

처음에는 [GCC](https://namu.wiki/w/GCC)를 프론트엔드로 하고 LLVM은 미들엔드-백엔드로 사용하는 LLVM-GCC 프로젝트(DragonEgg)가 있었으나, **LLVM의 자체 프론트엔드인 Clang이 등장**한 이후 **컴파일의 전 과정을 LLVM 툴체인으로 진행**할 수 있게 되었다.

# LLVM 프로젝트

## 1. LLVM Core

LLVM **미들엔드/백엔드**를 구성하는 라이브러리

## 2. Clang

C, C++, Objective-C 용 컴파일러. LLVM 프로젝트의 메인 프론트엔드로, 소스 코드를 LLVM IR(중간 표현, Intermediate Representation)로 컴파일하는 역할을 담당

## 2-1. Swift-Clang

Swift에 대응하는 프론트엔드 컴파일러. 애플이 기존의 LLVM을 포크해서 만든 **Swift-LLVM과 결합**되어 돌아간다. 기본적인 구조는 Clang과 유사하지만 스위프트 소스 코드와 LLVM IR 사이에 **SIL(Swift Intermediate Language)**이라는 또다른 중간 코드가 하나 더 있다.

즉, 프론트엔드에서 스위프트 코드는 SIL로 컴파일될 때 한 번, LLVM IR로 컴파일될 때 한번 해서 총 **두 번 최적화**되는 것이다.

![clang-llvm](https://camo.githubusercontent.com/a5f174499b3b151a17e61dfaffa75f2b71e71ec4858baf6c4c8726cd41007b0b/68747470733a2f2f626c6f672e6b616b616f63646e2e6e65742f646e2f5a62556d4b2f6274714f366638596c5a6f2f6b574d757a664b494a65584b4c6e774e4f6f6b346b312f696d672e6a7067)
## LLDB

LLVM 프론트엔드에 대응하는 디버거

# Swift의 빌드 단계 / 컴파일러 과정

Swift 소스 코드로 바이너리 코드를 생성할 때, 컴파일러는 다음의 과정을 거친다.

`Swift Code` -> `구문 분석 (AST 생성)` -> `의미 분석 (타입 정보를 포함한 AST 생성)` -> `모듈 임포트` -> `SIL 생성 (raw SIL)` -> `SIL 정규화 (canonical SIL)`, -> `SIL **최적화**` -> `LLVM IR 생성` (여기까지 LLVM의 프론트 엔드) -> `**최적화**` -> `Compiler backend` -> `Assembler` -> `링커`

- **Swift AST**(Abstract Syntax Tree)
	- Swift의 문법 분석(예약어 검사, 구현 등을 제외한 순수한 구문 분석)을 수행
	- `lib/AST` directory에 정의됨
	- 소스 파일에 있는 내용과 가장 가까운 representation
	- Swift 소스 코드, Swift 모듈 및 Clang 모듈로부터 생성됨(각각 `lib/Parse`, `lib/Serialization` , `lib/ClangImporter` 에서 생성)
	- 컴파일 초기에 resoultion, typechecking, high-level semantics functions (in `lib/Sema`) 으로 해석됨
- 