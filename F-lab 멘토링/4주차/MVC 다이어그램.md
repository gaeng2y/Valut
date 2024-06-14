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

import UIKit

class DrinkViewController: UIViewController {
    
    private let model = WaterIntakeModel()
    
    private let progressView = UIProgressView(progressViewStyle: .default)
    private let counterLabel = UILabel()
    private let literLabel = UILabel()
    private let drinkButton = UIButton(type: .system)
    private let resetButton = UIButton(type: .system)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = UIColor(named: "BackgroundColor")
        
        setupUI()
        updateUI()
    }
    
    private func setupUI() {
        // Configure progress view
        progressView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(progressView)
        
        // Configure counter label
        counterLabel.translatesAutoresizingMaskIntoConstraints = false
        counterLabel.font = UIFont.systemFont(ofSize: 24)
        view.addSubview(counterLabel)
        
        // Configure liter label
        literLabel.translatesAutoresizingMaskIntoConstraints = false
        literLabel.font = UIFont.systemFont(ofSize: 16)
        view.addSubview(literLabel)
        
        // Configure drink button
        drinkButton.setTitle("마시기", for: .normal)
        drinkButton.backgroundColor = .systemTeal
        drinkButton.setTitleColor(.white, for: .normal)
        drinkButton.layer.cornerRadius = 10
        drinkButton.addTarget(self, action: #selector(didTapDrinkButton), for: .touchUpInside)
        drinkButton.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(drinkButton)
        
        // Configure reset button
        resetButton.setTitle("초기화", for: .normal)
        resetButton.backgroundColor = .gray
        resetButton.setTitleColor(.white, for: .normal)
        resetButton.layer.cornerRadius = 10
        resetButton.addTarget(self, action: #selector(didTapResetButton), for: .touchUpInside)
        resetButton.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(resetButton)
        
        // Setup constraints
        NSLayoutConstraint.activate([
            progressView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            progressView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            progressView.widthAnchor.constraint(equalTo: view.widthAnchor, multiplier: 0.8),
            progressView.heightAnchor.constraint(equalToConstant: 20),
            
            counterLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            counterLabel.topAnchor.constraint(equalTo: progressView.bottomAnchor, constant: 20),
            
            literLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            literLabel.topAnchor.constraint(equalTo: counterLabel.bottomAnchor, constant: 10),
            
            drinkButton.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 40),
            drinkButton.topAnchor.constraint(equalTo: literLabel.bottomAnchor, constant: 20),
            drinkButton.widthAnchor.constraint(equalToConstant: 100),
            drinkButton.heightAnchor.constraint(equalToConstant: 50),
            
            resetButton.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -40),
            resetButton.topAnchor.constraint(equalTo: literLabel.bottomAnchor, constant: 20),
            resetButton.widthAnchor.constraint(equalToConstant: 100),
            resetButton.heightAnchor.constraint(equalToConstant: 50)
        ])
    }
    
    private func updateUI() {
        progressView.progress = Float(model.progress)
        counterLabel.text = "\(model.counter)잔"
        literLabel.text = "(\(model.getLiter()))"
    }
    
    @objc private func didTapDrinkButton() {
        model.incrementCounter()
        updateUI()
        
        if model.counter >= 8 {
            let alert = UIAlertController(title: "", message: "지금은 8잔까지만 설정 가능합니다 ㅜㅜ", preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "OK", style: .default))
            present(alert, animated: true)
        }
    }
    
    @objc private func didTapResetButton() {
        model.resetCounter()
        updateUI()
    }
}

import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        window = UIWindow(frame: UIScreen.main.bounds)
        let viewController = DrinkViewController()
        window?.rootViewController = viewController
        window?.makeKeyAndVisible()
        return true
    }
}

```