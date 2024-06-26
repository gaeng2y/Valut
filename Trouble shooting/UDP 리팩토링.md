https://stackoverflow.com/questions/54640996/how-to-work-with-udp-sockets-in-ios-swift

### UdpListener
```swift
final class UdpListener: ObservableObject {
    var listener: NWListener?
    var connection: NWConnection?
    var queue = DispatchQueue.global(qos: .background)
    /// New data will be place in this variable to be received by observers
    @Published private(set) var messageReceived: Data?
    /// When there is an active listening NWConnection this will be `true`
    @Published private(set) var isReady: Bool = false
    /// Default value `true`, this will become false if the UDPListener ceases listening for any reason
    @Published var listening: Bool = true
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
                AppLogger.shared.socketLog("Listener connected to port \(port)")
            case .failed(let error):
                AppLogger.shared.socketLog("Listener failed with error: \(error)")
                self.stopListening()
            case .cancelled:
                AppLogger.shared.socketLog("Listener cancelled")
                self.stopListening()
            default:
                AppLogger.shared.socketLog("Listener connecting to port \(port)...")
            }
        }
        self.listener?.newConnectionHandler = { connection in
            AppLogger.shared.socketLog("Listener receiving new message")
            self.createConnection(connection: connection)
        }
        self.listener?.start(queue: self.queue)
    }
    
    func createConnection(connection: NWConnection) {
        self.connection = connection
        self.connection?.stateUpdateHandler = { newState in
            switch newState {
            case .ready:
                AppLogger.shared.socketLog("Listener ready to receive message - \(connection)")
                self.receive()
            case .cancelled, .failed:
                AppLogger.shared.socketLog("Listener failed to receive message - \(connection)")
                // Cancel the listener, something went wrong
                self.listener?.cancel()
                // Announce we are no longer able to listen
                self.listening = false
            default:
                AppLogger.shared.socketLog("Listener waiting to receive message - \(connection)")
            }
        }
        self.connection?.start(queue: .global())
    }
    
    func receive() {
        self.connection?.receiveMessage { data, context, isComplete, error in
            if let unwrappedError = error {
                AppLogger.shared.socketLog("Error: NWError received in \(#function) - \(unwrappedError)")
                return
            }
            guard isComplete, let data = data else {
                AppLogger.shared.socketLog("Error: Received nil Data with context - \(String(describing: context))")
                return
            }
            self.messageReceived = data
            if self.listening {
                self.receive()
            }
        }
    }
    
    func stopListening() {
        DispatchQueue.main.async {
            self.listening = false
            self.isReady = false
        }
        self.connection?.cancel()
        self.listener?.cancel()
    }
    
    func cancel() {
        self.listening = false
        self.connection?.cancel()
    }
}
```

구현 후 싱글톤에서 가져와서 쓰는데 메세지를 못받음...
