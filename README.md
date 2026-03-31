# CS Study

## Foundations (기초 및 원리)

### 1. Swift Basics & Memory Layout
- [ ] [**Class vs Struct** (값 타입 vs 참조 타입의 실체 및 메모리 정렬)](./ClassVSStruct.md)
- [ ] **Optional** (nil, Forcing Unwrapping, Optional Binding: if let vs guard let 스타일 비교)
- [ ] **let vs var** (상수와 변수, 컴파일러의 최적화 방식)
- [ ] **Access Control** (open, public, internal, fileprivate, private 및 모듈 가시성)
- [ ] **Collections** (Array, Dictionary, Set의 차이와 Hashable/Equatable 원리)
- [ ] **Any vs AnyObject** (타입의 유연함과 Existential Container 구조)
- [ ] **Casting** (Upcasting vs Downcasting: as, as?, as!의 런타임 비용)

### 2. iOS Basics & UI System
- [ ] **App Life Cycle** (AppDelegate, SceneDelegate 상태 변화와 시스템 이벤트)
- [ ] **View Life Cycle** (viewDidLoad ~ deinit 순서와 각 단계별 적절한 로직 배치)
- [ ] **Bounds vs Frame** (좌표계 시스템의 차이와 View Rendering 원리)
- [ ] **UI Implementation** (Storyboard vs Code UI 장단점 및 IBOutlet/IBAction 연결 메커니즘)
- [ ] **Framework Comparison** (UIKit vs SwiftUI: 명령형 vs 선언형 프로그래밍의 철학 차이)

### 3. CS Basics (Hardware & Execution)
- [ ] **Stack vs Heap** (메모리 구조의 물리적 할당 방식과 할당/해제 오버헤드)
- [ ] **Process vs Thread** (OS 차원의 개념 차이와 멀티스레딩의 자원 공유)
- [ ] **Sync vs Async** (동기 vs 비동기 개념과 OS 스케줄러의 동작)

---

## Intermediate (실무 핵심 및 심화)

### 1. Swift Deep Dive (Memory & Syntax)
- [ ] **Memory Management (ARC)** (Reference Counting, Strong/Weak/Unowned의 실체)
- [ ] **Retain Cycle** (순환 참조 발생 원인과 Side Table을 통한 해결법)
- [ ] **Closure** (Capturing Values, @escaping, @autoclosure와 힙 할당 메커니즘)
- [ ] **Protocol Oriented Programming (POP)** (Protocol, Extension: 기능 확장 및 저장 프로퍼티 불가 이유)
- [ ] **Property Wrappers & Opaque Types** (some, any의 차이 및 State/Binding 내부 원리)
- [ ] **Generic** (유연한 함수/타입 만들기 및 Generic Specialization 최적화)
- [ ] **Data Handling** (Result Type 성공/실패 처리 및 Codable JSON Parsing 원리)

### 2. iOS Core (System & Logic)
- [ ] **Concurrency (GCD)** (Grand Central Dispatch, Serial/Concurrent Queue, QoS)
- [ ] **Auto Layout** (Hugging Priority vs Compression Resistance 및 Cassowary 알고리즘)
- [ ] **View Efficiency** (TableView/CollectionView Cell Reuse Mechanism 분석)
- [ ] **Design Patterns** (Singleton Pattern의 양날의 검, Delegate Pattern의 이벤트 처리)
- [ ] **Communication** (NotificationCenter vs KVO vs Delegate 통신 방식 비교)
- [ ] **Persistence** (UserDefaults vs CoreData vs Keychain 데이터 저장 및 보안 비교)

### 3. Network & CS (Protocol)
- [ ] **HTTP Protocol** (Methods: GET/POST/PUT 등, Status Codes 의미)
- [ ] **REST API** (RESTful의 정의와 Stateless 통신)
- [ ] **Transport Layer** (TCP vs UDP 전송 계층 차이 및 TLS Handshake)
- [ ] **Socket** (HTTP 통신과의 차이 및 실시간 양방향 통신의 원리)

---

## Advanced (심화 및 아키텍처)

### 1. Architecture & Design Patterns
- [ ] **Design Patterns** (MVC의 Massive VC 문제 해결, MVVM의 ViewModel 역할과 Data Binding)
- [ ] **Navigation & Logic** (Coordinator Pattern을 통한 화면 전환 분리, Clean Architecture 개념)
- [ ] **SOLID Principles** (객체지향 설계 5원칙의 Swift 실무 적용)
- [ ] **Modular Architecture** (Tuist를 활용한 모듈화 및 의존성 관리)

### 2. Advanced Swift & Optimization
- [ ] **Modern Concurrency** (async / await, Actor 모델의 데이터 레이스 방지)
- [ ] **Reactive Programming** (Combine / RxSwift 기초 및 스트림 설계)
- [ ] **Method Dispatch** (Static vs Dynamic Dispatch: VTable/Witness Table 분석)
- [ ] **Copy on Write (COW)** (값 타입의 성능 최적화 내부 동작 원리)
- [ ] **Hashable & Equatable** (내부 동작 원리와 컬렉션 검색 성능 최적화)
- [ ] **Run Loop** (메인 런루프의 역할과 메인 스레드 부하 관리)

### 3. Optimization & Testing
- [ ] **Debugging** (Memory Leak 분석 및 Instruments 도구 활용 실습)
- [ ] **Runtime Tools** (Thread/Address Sanitizer 등 런타임 디버깅 도구 활용)
- [ ] **Quality Assurance** (Unit Test: XCTest 기초, UI Test 개념 및 자동화)
- [ ] **CI/CD** (Test 파이프라인 구축 및 코드 커버리지 관리)

---

## Mastery (시스템 및 로우 레벨)

### 1. Compiler & Binary
- [ ] **Compilation Pipeline** (Source → SIL → LLVM IR → Machine Code 최적화 과정)
- [ ] **Binary Structure** (Mach-O 파일 구조 및 Static vs Dynamic Linking 분석)
- [ ] **Swift Stability*** (ABI Stability 및 Module Stability의 원리와 라이브러리 배포)

### 2. Graphics & OS Internals
- [ ] **Rendering Pipeline** (Render Loop: Commit-Render-Display 및 Core Animation 원리)
- [ ] **Off-screen Rendering** (성능 병목의 원인 분석 및 GPU 최적화)
- [ ] **OS Security** (App Sandbox, Code Signing 및 프로비저닝 메커니즘)

---
