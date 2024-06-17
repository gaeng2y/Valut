### Model

```swift
import Foundation

struct WaterIntakeState {
    var counter: Int
    var progress: CGFloat
    var milliliter: Int
}

enum WaterIntakeAction {
    case increment
    case reset
}

class WaterIntakeModel {
    static let glassesOfTodayKey = "glassesOfToday"
    private let glassesOfToday: Int = 8

    func reduce(state: inout WaterIntakeState, action: WaterIntakeAction) {
        switch action {
        case .increment:
            if state.counter < glassesOfToday {
                state.counter += 1
            }
        case .reset:
            state.counter = 0
        }
        state.progress = CGFloat(state.counter) / CGFloat(glassesOfToday)
        state.milliliter = state.counter * 250
        saveCounter(counter: state.counter)
    }

    func initialState() -> WaterIntakeState {
        let counter = UserDefaults.standard.integer(forKey: WaterIntakeModel.glassesOfTodayKey)
        return WaterIntakeState(counter: counter,
                                progress: CGFloat(counter) / CGFloat(glassesOfToday),
                                milliliter: counter * 250)
    }

    private func saveCounter(counter: Int) {
        UserDefaults.standard.set(counter, forKey: WaterIntakeModel.glassesOfTodayKey)
    }

    func getLiter(milliliter: Int) -> String {
        guard milliliter >= 1000 else { return "\(milliliter)ML" }
        return "\(Float(milliliter) / 1000)L"
    }
}
```

### Intent

```swift
import Foundation
import Combine

class WaterIntakeIntent {
    private let model: WaterIntakeModel
    private(set) var state: WaterIntakeState {
        didSet {
            stateSubject.send(state)
        }
    }
    private let stateSubject = PassthroughSubject<WaterIntakeState, Never>()

    var statePublisher: AnyPublisher<WaterIntakeState, Never> {
        stateSubject.eraseToAnyPublisher()
    }

    init(model: WaterIntakeModel) {
        self.model = model
        self.state = model.initialState()
    }

    func send(action: WaterIntakeAction) {
        model.reduce(state: &state, action: action)
    }

    func getLiter() -> String {
        return model.getLiter(milliliter: state.milliliter)
    }
}
```

### View

```swift
import UIKit
import Combine

class DrinkView: UIView {
    
    let progressView = UIProgressView(progressViewStyle: .default)
    let counterLabel = UILabel()
    let literLabel = UILabel()
    let drinkButton = UIButton(type: .system)
    let resetButton = UIButton(type: .system)
    private var cancellables: Set<AnyCancellable> = []
    
    var intent: WaterIntakeIntent! {
        didSet {
            bindIntent()
        }
    }
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setupUI()
    }
    
    private func setupUI() {
        backgroundColor = UIColor(named: "BackgroundColor")
        
        // Configure progress view
        progressView.translatesAutoresizingMaskIntoConstraints = false
        addSubview(progressView)
        
        // Configure counter label
        counterLabel.translatesAutoresizingMaskIntoConstraints = false
        counterLabel.font = UIFont.systemFont(ofSize: 24)
        addSubview(counterLabel)
        
        // Configure liter label
        literLabel.translatesAutoresizingMaskIntoConstraints = false
        literLabel.font = UIFont.systemFont(ofSize: 16)
        addSubview(literLabel)
        
        // Configure drink button
        drinkButton.setTitle("마시기", for: .normal)
        drinkButton.backgroundColor = .systemTeal
        drinkButton.setTitleColor(.white, for: .normal)
        drinkButton.layer.cornerRadius = 10
        drinkButton.translatesAutoresizingMaskIntoConstraints = false
        addSubview(drinkButton)
        
        // Configure reset button
        resetButton.setTitle("초기화", for: .normal)
        resetButton.backgroundColor = .gray
        resetButton.setTitleColor(.white, for: .normal)
        resetButton.layer.cornerRadius = 10
        resetButton.translatesAutoresizingMaskIntoConstraints = false
        addSubview(resetButton)
        
        // Setup constraints
        NSLayoutConstraint.activate([
            progressView.centerXAnchor.constraint(equalTo: centerXAnchor),
            progressView.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor, constant: 20),
            progressView.widthAnchor.constraint(equalTo: widthAnchor, multiplier: 0.8),
            progressView.heightAnchor.constraint(equalToConstant: 20),
            
            counterLabel.centerXAnchor.constraint(equalTo: centerXAnchor),
            counterLabel.topAnchor.constraint(equalTo: progressView.bottomAnchor, constant: 20),
            
            literLabel.centerXAnchor.constraint(equalTo: centerXAnchor),
            literLabel.topAnchor.constraint(equalTo: counterLabel.bottomAnchor, constant: 10),
            
            drinkButton.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 40),
            drinkButton.topAnchor.constraint(equalTo: literLabel.bottomAnchor, constant: 20),
            drinkButton.widthAnchor.constraint(equalToConstant: 100),
            drinkButton.heightAnchor.constraint(equalToConstant: 50),
            
            resetButton.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -40),
            resetButton.topAnchor.constraint(equalTo: literLabel.bottomAnchor, constant: 20),
            resetButton.widthAnchor.constraint(equalToConstant: 100),
            resetButton.heightAnchor.constraint(equalToConstant: 50)
        ])
    }
    
    private func bindIntent() {
        intent.statePublisher
            .receive(on: RunLoop.main)
            .sink { [weak self] state in
                self?.counterLabel.text = "\(state.counter)잔"
                self?.progressView.progress = Float(state.progress)
                self?.literLabel.text = "(\(self?.intent.getLiter() ?? ""))"
            }
            .store(in: &cancellables)
    }
}
```

### ViewController

```swift
import UIKit

class DrinkViewController: UIViewController {
    
    private let model = WaterIntakeModel()
    private var intent: WaterIntakeIntent!
    private var drinkView: DrinkView!
    
    override func loadView() {
        drinkView = DrinkView()
        view = drinkView
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        intent = WaterIntakeIntent(model: model)
        drinkView.intent = intent
        
        drinkView.drinkButton.addTarget(self, action: #selector(didTapDrinkButton), for: .touchUpInside)
        drinkView.resetButton.addTarget(self, action: #selector(didTapResetButton), for: .touchUpInside)
    }
    
    @objc private func didTapDrinkButton() {
        intent.send(action: .increment)
        
        if intent.state.counter >= 8 {
            let alert = UIAlertController(title: "", message: "지금은 8잔까지만 설정 가능합니다 ㅜㅜ", preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "OK", style: .default))
            present(alert, animated: true)
        }
    }
    
    @objc private func didTapResetButton() {
        intent.send(action: .reset)
    }
}
```



![](MVI.png)