클라이언트가 자신이 이용하지 않는 메서드에 의존하지 않아야 한다.

```swift
import Foundation

class Document { }

protocol Machine {
  func print(d: Document)
  func scan(d: Document)
  func fax(d: Document)
}

class MFP : Machine {
  func print(d: Document)
  //
}

enum NoRequiredFunctionality : Error {
  case doesNotFax
}

class OldFashionedPrinter: Machine {
  func print(d: Document) { }
  func fax(d: Document) throws {
    throw NoRequiredFunctionality.doesNotFax
  }
}
```

위 예제에서는 `Machine`에 `print(d:)` `scan(d:)` `fax(d:)` 이 모두 있기 때문에 `MFP`, `OldFashionedPrinter` 에서는 준수하는 메소드를 사용할 수 없기 때문에 ISP에 위배 되기 때문에

```swift
protocol Printer {
  func print(d: Document)
}

protocol Scanner {
  func scan(d: Document)
}

protocol Fax {
  func fax(d: Document)
}

class OrdinaryPrinter : Printer {
  func print(d: Document) { }
}

class Photocopier : Printer, Scanner {
  func print(d: Document) { }

  func scan(d: Document) {}
}

protocol MultiFunctionDevice : Printer, Scanner, Fax { }

class MultiFunctionMachine : MultiFunctionDevice {
  let printer: Printer
  let scanner: Scanner

  init(printer: Printer, scanner: Scanner) {
    self.printer = printer
    self.scanner = scanner
  }

  func print(d: Document) {
    printer.print(d) // Decorator
  }
}
```

위와 같이 출력하는 역할을 가진 프로토콜, 스캔하는 역할을 가진 프로토콜, 팩스를 보내는 역할을 하는 프로토콜로 나누면서 역할 인터페이스로 만든다.