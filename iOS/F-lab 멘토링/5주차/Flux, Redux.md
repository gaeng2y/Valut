Flux, Redux 는 MVC 의 복잡성 증가로 인해 탄생한 것이다. 두 패턴에는 일정한 패턴이 있다.

- 사용자 Action 을 미리 정의하는 일급 객체가 존재한다.
- 사용자 Action 에 대한 응답을 Store 하는 일급 객체가 존재한다.
- Action -> Response 를 정의하고 관리하는 객체가 따로 존재한다.
- 모든 객체들이 각각의 상태를 직접 참조하는 대신 메시지를 매개체로 하여 소통한다.

## Flux

![flux](https://velog.velcdn.com/images/sanghwi_back/post/ee47ef9c-f2a8-45c2-bdda-77d77da2f823/image.png)

1. **Action (액션)**:
   - 사용자 입력이나 시스템 이벤트를 나타내는 객체.
   - 예시:
     ```swift
 enum Action {
	case addTodoItem(text: String)
	case removeTodoItem(index: Int)
 }
     ```

2. **Dispatcher (디스패처)**:
   - 액션을 받아서 스토어에 전달하는 중앙 허브.
   - 예시:
```swift
class Dispatcher {
	 static let shared = Dispatcher()
	 private init() {}
	 
	 func dispatch(action: Action) {
		 Store.shared.handleAction(action: action)
	 }
 }
```

3. **Store (스토어)**:
   - 애플리케이션의 상태를 관리하고, 상태 변경을 처리.
   - 예시:
     ```swift
class Store: ObservableObject {
	 static let shared = Store()
	 @Published private(set) var todoItems: [String] = []
	 
	 func handleAction(action: Action) {
		 switch action {
		 case .addTodoItem(let text):
			 todoItems.append(text)
		 case .removeTodoItem(let index):
			 todoItems.remove(at: index)
		 }
	 }
}
     ```

4. **View (뷰)**:
   - 상태를 바탕으로 UI를 업데이트하고, 사용자 입력을 받아 액션을 생성.
   - 예시:
```swift
struct ContentView: View {
	 @ObservedObject var store = Store.shared
	 @State private var newItemText: String = ""
	 
	 var body: some View {
		 VStack {
			 TextField("New Item", text: $newItemText)
			 Button(action: {
				 Dispatcher.shared.dispatch(action: .addTodoItem(text: newItemText))
				 newItemText = ""
			 }) {
				 Text("Add Item")
			 }
			 
			 List {
				 ForEach(store.todoItems.indices, id: \.self) { index in
					 Text(store.todoItems[index])
						 .onTapGesture {
							 Dispatcher.shared.dispatch(action: .removeTodoItem(index: index))
						 }
				 }
			 }
		 }
	 }
}
```

## Redux
Redux 는 쉽게 말해 Flux + Functional Programming이다.

1. **Actions (액션)**:
   - 상태 변화를 설명하는 객체.
   - 예시:
     ```swift
 enum Action {
	 case addTodoItem(text: String)
	 case removeTodoItem(index: Int)
 }
     ```

2. **Reducers (리듀서)**:
   - 액션을 처리하여 상태를 변경하는 순수 함수.
   - 예시:
     ```swift
 func todoReducer(state: [String], action: Action) -> [String] {
	 var newState = state
	 switch action {
	 case .addTodoItem(let text):
		 newState.append(text)
	 case .removeTodoItem(let index):
		 newState.remove(at: index)
	 }
	 return newState
 }
     ```

3. **Store (스토어)**:
   - 애플리케이션의 상태를 저장하고 관리.
   - 상태를 읽고, 액션을 디스패치하며, 리스너를 등록/해제할 수 있다.
   - 예시:
     ```swift
 class Store: ObservableObject {
	 @Published private(set) var state: [String] = []
	 private let reducer: (Action) -> Void
	 
	 init(reducer: @escaping (Action) -> Void) {
		 self.reducer = reducer
	 }
	 
	 func dispatch(action: Action) {
		 state = todoReducer(state: state, action: action)
	 }
 }
     ```

4. **Middleware (미들웨어)** (옵션):
   - 액션이 리듀서에 도달하기 전에 가로채서 추가적인 로직을 실행할 수 있다.
   - 예시:
     ```swift
 func loggerMiddleware(action: Action) {
	 print("Dispatching action: \(action)")
 }
     ```

5. **View (뷰)**:
   - 상태를 바탕으로 UI를 업데이트하고, 사용자 입력을 받아 액션을 생성.
   - 예시:
     ```swift
 struct ContentView: View {
	 @ObservedObject var store: Store
	 
	 @State private var newItemText: String = ""
	 
	 var body: some View {
		 VStack {
			 TextField("New Item", text: $newItemText)
			 Button(action: {
				 loggerMiddleware(action: .addTodoItem(text: newItemText))
				 store.dispatch(action: .addTodoItem(text: newItemText))
				 newItemText = ""
			 }) {
				 Text("Add Item")
			 }
			 
			 List {
				 ForEach(store.state.indices, id: \.self) { index in
					 Text(store.state[index])
						 .onTapGesture {
							 loggerMiddleware(action: .removeTodoItem(index: index))
							 store.dispatch(action: .removeTodoItem(index: index))
						 }
				 }
			 }
		 }
	 }
 }
     ```
