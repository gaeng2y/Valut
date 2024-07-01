### Model

 ```swift
 import Foundation

class WaterIntakeModel {
    static let glassesOfTodayKey = "glassesOfToday"
    private let glassesOfToday: Int = 8
    private(set) var counter: Int {
        didSet {
            progress = CGFloat(counter) / CGFloat(glassesOfToday)
            milliliter = counter * 250
            saveCounter()
        }
    }
    private(set) var progress: CGFloat
    private(set) var milliliter: Int
    
    init() {
        self.counter = UserDefaults.standard.integer(forKey: WaterIntakeModel.glassesOfTodayKey)
        self.progress = CGFloat(counter) / CGFloat(glassesOfToday)
        self.milliliter = counter * 250
    }
    
    func incrementCounter() {
        if counter < glassesOfToday {
            counter += 1
        }
    }
    
    func resetCounter() {
        counter = 0
    }
    
    private func saveCounter() {
        UserDefaults.standard.set(counter, forKey: WaterIntakeModel.glassesOfTodayKey)
    }
    
    func getLiter() -> String {
        guard milliliter >= 1000 else { return "\(milliliter)ML" }
        return "\(Float(milliliter) / 1000)L"
    }
}
```
### ViewModel

```swift
import Foundation
import Combine

class DrinkViewModel: ObservableObject {
    @Published private(set) var counter: Int
    @Published private(set) var progress: CGFloat
    @Published private(set) var milliliter: Int
    private let model: WaterIntakeModel
    
    init(model: WaterIntakeModel) {
        self.model = model
        self.counter = model.counter
        self.progress = model.progress
        self.milliliter = model.milliliter
    }
    
    func incrementCounter() {
        model.incrementCounter()
        updateProperties()
    }
    
    func resetCounter() {
        model.resetCounter()
        updateProperties()
    }
    
    func getLiter() -> String {
        return model.getLiter()
    }
    
    private func updateProperties() {
        counter = model.counter
        progress = model.progress
        milliliter = model.milliliter
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
    
    var viewModel: DrinkViewModel! {
        didSet {
            bindViewModel()
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
    
    private func bindViewModel() {
        viewModel.$counter
            .receive(on: RunLoop.main)
            .sink { [weak self] counter in
                self?.counterLabel.text = "\(counter)잔"
            }
            .store(in: &cancellables)
        
        viewModel.$progress
            .receive(on: RunLoop.main)
            .sink { [weak self] progress in
                self?.progressView.progress = Float(progress)
            }
            .store(in: &cancellables)
        
        viewModel.$milliliter
            .receive(on: RunLoop.main)
            .sink { [weak self] _ in
                self?.literLabel.text = "(\(self?.viewModel.getLiter() ?? ""))"
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
    private var viewModel: DrinkViewModel!
    private var drinkView: DrinkView!
    
    override func loadView() {
        drinkView = DrinkView()
        view = drinkView
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        viewModel = DrinkViewModel(model: model)
        drinkView.viewModel = viewModel
        
        drinkView.drinkButton.addTarget(self, action: #selector(didTapDrinkButton), for: .touchUpInside)
        drinkView.resetButton.addTarget(self, action: #selector(didTapResetButton), for: .touchUpInside)
    }
    
    @objc private func didTapDrinkButton() {
        viewModel.incrementCounter()
        
        if viewModel.counter >= 8 {
            let alert = UIAlertController(title: "", message: "지금은 8잔까지만 설정 가능합니다 ㅜㅜ", preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "OK", style: .default))
            present(alert, animated: true)
        }
    }
    
    @objc private func didTapResetButton() {
        viewModel.resetCounter()
    }
}

```

![](MVVM%201.jpg)