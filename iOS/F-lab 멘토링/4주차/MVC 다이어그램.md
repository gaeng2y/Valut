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

class DrinkView: UIView {
    
    let progressView = UIProgressView(progressViewStyle: .default)
    let counterLabel = UILabel()
    let literLabel = UILabel()
    let drinkButton = UIButton(type: .system)
    let resetButton = UIButton(type: .system)
    
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
    
    func updateUI(counter: Int, progress: CGFloat, liter: String) {
        progressView.progress = Float(progress)
        counterLabel.text = "\(counter)잔"
        literLabel.text = "(\(liter))"
    }
}

import UIKit

class DrinkViewController: UIViewController {
    
    private let model = WaterIntakeModel()
    private var drinkView: DrinkView!
    
    override func loadView() {
        drinkView = DrinkView()
        view = drinkView
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        drinkView.drinkButton.addTarget(self, action: #selector(didTapDrinkButton), for: .touchUpInside)
        drinkView.resetButton.addTarget(self, action: #selector(didTapResetButton), for: .touchUpInside)
        
        updateUI()
    }
    
    private func updateUI() {
        drinkView.updateUI(counter: model.counter, progress: model.progress, liter: model.getLiter())
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

```

```swift
import SwiftUI

struct SplashView: View {
    @Environment(\.colorScheme) var colorScheme
    @State private var navigated: Bool = false
    
    let moveToTimer = Timer.publish(every: 2, on: .main, in: .common)
        .autoconnect()
    
    var body: some View {
        ZStack {
            let fileName = colorScheme == .light ? "bubble_light" : "bubble_dark"
            LottieView(lottieFile: fileName)
                .ignoresSafeArea()
            
            FadeInOutView(text: "물리미", startTime: 1)
                .padding()
                .font(.title)
                .foregroundColor(.white)
        }
        .onReceive(moveToTimer) { _ in
            if UserDefaults.shared.integer(forKey: "totalCount") <= 0 {
                UserDefaults.shared.set(8, forKey: "totalCount")
            }
            
            navigated = true
        }
        .fullScreenCover(isPresented: $navigated) {
            MainTabView()
        }
    }
}

import Lottie
import SwiftUI

struct LottieView: UIViewRepresentable {
    var lottieFile: String
    var loopMode: LottieLoopMode = .playOnce
    var animationView = LottieAnimationView()
    
    func makeUIView(context: UIViewRepresentableContext<LottieView>) -> UIView {
        let view = UIView()
        
        self.animationView.animation = LottieAnimation.named(lottieFile)
        self.animationView.contentMode = .scaleAspectFill
        self.animationView.loopMode = loopMode
        
        self.animationView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(self.animationView)
        
        NSLayoutConstraint.activate([
            self.animationView.topAnchor.constraint(equalTo: view.topAnchor),
            self.animationView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            self.animationView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            self.animationView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
        ])
        
        return view
    }
    
    func updateUIView(_ uiView: UIView, context: UIViewRepresentableContext<LottieView>) {
        self.animationView.play()
    }
}

import SwiftUI

struct FadeInOutView: View {
    @State var characters: Array<String.Element>
    @State var opacity: Double = 0
    @State var baseTime: Double

    init(text: String, startTime: Double) {
        characters = Array(text)
        baseTime = startTime
    }

    var body: some View {
        HStack(spacing: 0) {
            ForEach(0 ..< characters.count) { num in
                Text(String(self.characters[num]))
                    .font(.largeTitle)
                    .fontWeight(.heavy)
                    .opacity(opacity)
                    .animation(.easeInOut.delay(Double(num) * 0.1),
                               value: opacity)
            }
        }
        .onAppear {
            DispatchQueue.main.asyncAfter(deadline: .now() + baseTime) {
                opacity = 1
            }
        }
        .onTapGesture {
            opacity = 0
            DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
                opacity = 1
            }
        }
    }
}

```

-> 

![](MVC-DrinkView.png)