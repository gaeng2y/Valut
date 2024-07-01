서드파티 라이브러리인 [CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket)이 최근 버전이 4년 전이기도하고 크래시가 발생하는 이슈가 있어 퍼스트 파티 프레임워크인 Swift Network를 이용하려고 한다.

주요 맹점은 UDP 통신 하던 부분에 대해 어떻게 처리할 것이냐 이다.

### UdpListener

```swift
import Foundation
import Network
import Combine

class UDPListener: ObservableObject {
    
    var listener: NWListener?
    var connection: NWConnection?
    var queue = DispatchQueue.global(qos: .userInitiated)
    /// New data will be place in this variable to be received by observers
    @Published private(set) public var messageReceived: Data?
    /// When there is an active listening NWConnection this will be `true`
    @Published private(set) public var isReady: Bool = false
    /// Default value `true`, this will become false if the UDPListener ceases listening for any reason
    @Published public var listening: Bool = true
    
    /// A convenience init using Int instead of NWEndpoint.Port
    convenience init(on port: Int) {
        self.init(on: NWEndpoint.Port(integerLiteral: NWEndpoint.Port.IntegerLiteralType(port)))
    }
    /// Use this init or the one that takes an Int to start the listener
    init(on port: NWEndpoint.Port) {
        let params = NWParameters.udp
        params.allowFastOpen = true
        self.listener = try? NWListener(using: params, on: port)
        self.listener?.stateUpdateHandler = { update in
            switch update {
            case .ready:
                self.isReady = true
                print("Listener connected to port \(port)")
            case .failed, .cancelled:
                // Announce we are no longer able to listen
                self.listening = false
                self.isReady = false
                print("Listener disconnected from port \(port)")
            default:
                print("Listener connecting to port \(port)...")
            }
        }
        self.listener?.newConnectionHandler = { connection in
            print("Listener receiving new message")
            self.createConnection(connection: connection)
        }
        self.listener?.start(queue: self.queue)
    }
    
    func createConnection(connection: NWConnection) {
        self.connection = connection
        self.connection?.stateUpdateHandler = { (newState) in
            switch (newState) {
            case .ready:
                print("Listener ready to receive message - \(connection)")
                self.receive()
            case .cancelled, .failed:
                print("Listener failed to receive message - \(connection)")
                // Cancel the listener, something went wrong
                self.listener?.cancel()
                // Announce we are no longer able to listen
                self.listening = false
            default:
                print("Listener waiting to receive message - \(connection)")
            }
        }
        self.connection?.start(queue: .global())
    }
    
    func receive() {
        self.connection?.receiveMessage { data, context, isComplete, error in
            if let unwrappedError = error {
                print("Error: NWError received in \(#function) - \(unwrappedError)")
                return
            }
            guard isComplete, let data = data else {
                print("Error: Received nil Data with context - \(String(describing: context))")
                return
            }
            self.messageReceived = data
            if self.listening {
                self.receive()
            }
        }
    }
    
    func cancel() {
        self.listening = false
        self.connection?.cancel()
    }
}
```

이러한 UDP를 연결하고 받는 클래스를 만든다.
