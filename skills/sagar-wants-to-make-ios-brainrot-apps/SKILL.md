---
name: sagar-wants-to-make-ios-brainrot-apps
description: Expert Swift & iOS development skill — SwiftUI architecture, design systems, networking, state management, and production patterns for building native iOS apps with Claude Code.
triggers:
  - swift
  - ios
  - swiftui
  - xcode
  - uikit
  - cocoapods
  - spm
  - swift package
  - iphone app
  - ipad app
  - apple developer
  - core data
  - swiftdata
  - combine
  - async await swift
---

# Swift & iOS Development Skill

Expert-level Swift and iOS development assistant. Covers SwiftUI, UIKit interop, architecture, design systems, networking, state management, testing, and production deployment patterns.

## When to Use

- Building or modifying iOS/iPadOS/macOS apps
- Writing Swift code (views, models, networking, persistence)
- Designing UI with SwiftUI or UIKit
- Setting up Xcode projects, SPM packages, or CocoaPods
- Debugging iOS-specific issues (layout, navigation, lifecycle)
- App Store submission, provisioning, CI/CD for Apple platforms

---

## Project Structure (Recommended)

```
MyApp/
├── MyApp.xcodeproj/              # Or .xcworkspace if using CocoaPods
├── MyApp/
│   ├── App/
│   │   ├── MyAppApp.swift        # @main entry point
│   │   ├── AppDelegate.swift     # Only if UIKit lifecycle needed
│   │   └── Info.plist
│   ├── Features/                 # Feature modules
│   │   ├── Chat/
│   │   │   ├── Views/
│   │   │   │   ├── ChatView.swift
│   │   │   │   └── MessageBubble.swift
│   │   │   ├── ViewModels/
│   │   │   │   └── ChatViewModel.swift
│   │   │   └── Models/
│   │   │       └── Message.swift
│   │   ├── Settings/
│   │   │   ├── Views/
│   │   │   └── ViewModels/
│   │   └── Auth/
│   │       ├── Views/
│   │       └── Services/
│   ├── Core/
│   │   ├── Design/               # Design system
│   │   │   ├── Theme.swift
│   │   │   ├── Colors.swift
│   │   │   ├── Typography.swift
│   │   │   └── Components/       # Reusable UI components
│   │   │       ├── PrimaryButton.swift
│   │   │       ├── CardView.swift
│   │   │       └── LoadingView.swift
│   │   ├── Networking/
│   │   │   ├── APIClient.swift
│   │   │   ├── Endpoint.swift
│   │   │   └── NetworkError.swift
│   │   ├── Storage/
│   │   │   ├── UserDefaults+Extensions.swift
│   │   │   └── KeychainService.swift
│   │   ├── Extensions/
│   │   │   ├── View+Extensions.swift
│   │   │   ├── Color+Hex.swift
│   │   │   └── Date+Formatting.swift
│   │   └── Utilities/
│   │       ├── Logger.swift
│   │       └── Constants.swift
│   ├── Resources/
│   │   ├── Assets.xcassets/
│   │   ├── Localizable.xcstrings  # String catalogs (Xcode 15+)
│   │   └── Fonts/                 # Custom fonts (.ttf/.otf)
│   └── Preview Content/
│       └── Preview Assets.xcassets/
├── MyAppTests/
├── MyAppUITests/
└── Package.swift                  # If using SPM for modularization
```

### Feature-Based Organization

Group by feature, not by type. Each feature folder contains its own Views, ViewModels, and Models. Shared code lives in `Core/`.

---

## Architecture: MVVM + Services

### The Pattern

```
View ←→ ViewModel ←→ Service/Repository ←→ API/Database
```

**View** — Pure SwiftUI. No business logic. Observes ViewModel.
**ViewModel** — `@Observable` class (iOS 17+) or `ObservableObject`. Holds state, handles user actions.
**Service** — Stateless. Networking, persistence, business rules.
**Model** — Plain structs. `Codable`, `Identifiable`, `Hashable`.

### ViewModel Pattern (iOS 17+ with @Observable)

```swift
import SwiftUI

@Observable
final class ChatViewModel {
    var messages: [Message] = []
    var inputText = ""
    var isLoading = false
    var error: AppError?

    private let chatService: ChatService

    init(chatService: ChatService = .shared) {
        self.chatService = chatService
    }

    func send() async {
        guard !inputText.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty else { return }
        let text = inputText
        inputText = ""
        isLoading = true
        defer { isLoading = false }

        let userMessage = Message(role: .user, content: text)
        messages.append(userMessage)

        do {
            let response = try await chatService.sendMessage(text, history: messages)
            messages.append(response)
        } catch {
            self.error = AppError(error)
            messages.removeLast() // Remove optimistic user message on failure
        }
    }
}
```

### ViewModel Pattern (iOS 16 and earlier with ObservableObject)

```swift
final class ChatViewModel: ObservableObject {
    @Published var messages: [Message] = []
    @Published var inputText = ""
    @Published var isLoading = false

    // Same logic, use @StateObject in view
}
```

### View Consuming ViewModel

```swift
struct ChatView: View {
    @State private var viewModel = ChatViewModel()

    var body: some View {
        VStack(spacing: 0) {
            ScrollViewReader { proxy in
                ScrollView {
                    LazyVStack(spacing: 12) {
                        ForEach(viewModel.messages) { message in
                            MessageBubble(message: message)
                                .id(message.id)
                        }
                    }
                    .padding()
                }
                .onChange(of: viewModel.messages.count) {
                    withAnimation {
                        proxy.scrollTo(viewModel.messages.last?.id, anchor: .bottom)
                    }
                }
            }

            InputBar(text: $viewModel.inputText, isLoading: viewModel.isLoading) {
                Task { await viewModel.send() }
            }
        }
    }
}
```

---

## Design System

Since you can't use Tailwind in native iOS, build a design system with Swift extensions and view modifiers.

### Colors

```swift
import SwiftUI

extension Color {
    // MARK: - Brand
    static let brand = Color("Brand", bundle: .main)           // Define in Assets.xcassets
    static let brandDark = Color("BrandDark", bundle: .main)

    // MARK: - Semantic
    static let surfacePrimary = Color(.systemBackground)
    static let surfaceSecondary = Color(.secondarySystemBackground)
    static let surfaceTertiary = Color(.tertiarySystemBackground)
    static let textPrimary = Color(.label)
    static let textSecondary = Color(.secondaryLabel)
    static let textMuted = Color(.tertiaryLabel)
    static let border = Color(.separator)
    static let borderLight = Color(.opaqueSeparator)

    // MARK: - Status
    static let success = Color.green
    static let warning = Color.orange
    static let destructive = Color.red

    // MARK: - Hex initializer
    init(hex: String) {
        let hex = hex.trimmingCharacters(in: .alphanumerics.inverted)
        var int: UInt64 = 0
        Scanner(string: hex).scanHexInt64(&int)
        let a, r, g, b: UInt64
        switch hex.count {
        case 6: (a, r, g, b) = (255, int >> 16, int >> 8 & 0xFF, int & 0xFF)
        case 8: (a, r, g, b) = (int >> 24, int >> 16 & 0xFF, int >> 8 & 0xFF, int & 0xFF)
        default: (a, r, g, b) = (255, 0, 0, 0)
        }
        self.init(
            .sRGB,
            red: Double(r) / 255,
            green: Double(g) / 255,
            blue: Double(b) / 255,
            opacity: Double(a) / 255
        )
    }
}
```

### Typography

```swift
import SwiftUI

enum AppFont {
    // System Dynamic Type (recommended — respects accessibility)
    static let largeTitle = Font.largeTitle.weight(.bold)
    static let title = Font.title2.weight(.semibold)
    static let headline = Font.headline
    static let body = Font.body
    static let callout = Font.callout
    static let caption = Font.caption
    static let captionSecondary = Font.caption2

    // Custom font (register in Info.plist under "Fonts provided by application")
    static func custom(_ size: CGFloat, weight: Font.Weight = .regular) -> Font {
        .system(size: size, weight: weight, design: .rounded)
    }
}

// Usage: Text("Hello").font(AppFont.title)
```

### Spacing & Layout

```swift
enum Spacing {
    static let xxs: CGFloat = 2
    static let xs: CGFloat = 4
    static let sm: CGFloat = 8
    static let md: CGFloat = 12
    static let lg: CGFloat = 16
    static let xl: CGFloat = 24
    static let xxl: CGFloat = 32
    static let xxxl: CGFloat = 48
}

enum CornerRadius {
    static let sm: CGFloat = 8
    static let md: CGFloat = 12
    static let lg: CGFloat = 16
    static let xl: CGFloat = 24
    static let full: CGFloat = 9999  // Capsule
}
```

### Reusable Components

```swift
// MARK: - Primary Button
struct PrimaryButton: View {
    let title: String
    let isLoading: Bool
    let action: () -> Void

    init(_ title: String, isLoading: Bool = false, action: @escaping () -> Void) {
        self.title = title
        self.isLoading = isLoading
        self.action = action
    }

    var body: some View {
        Button(action: action) {
            Group {
                if isLoading {
                    ProgressView()
                        .tint(.white)
                } else {
                    Text(title)
                        .font(.body.weight(.semibold))
                }
            }
            .frame(maxWidth: .infinity)
            .frame(height: 50)
            .background(Color.brand)
            .foregroundStyle(.white)
            .clipShape(RoundedRectangle(cornerRadius: CornerRadius.lg))
        }
        .disabled(isLoading)
    }
}

// MARK: - Card
struct CardView<Content: View>: View {
    let content: Content

    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }

    var body: some View {
        content
            .padding(Spacing.lg)
            .background(Color.surfaceSecondary)
            .clipShape(RoundedRectangle(cornerRadius: CornerRadius.md))
    }
}

// MARK: - Async Image with Placeholder
struct RemoteImage: View {
    let url: URL?

    var body: some View {
        AsyncImage(url: url) { phase in
            switch phase {
            case .success(let image):
                image.resizable().scaledToFill()
            case .failure:
                Image(systemName: "photo")
                    .foregroundStyle(.tertiary)
            default:
                ProgressView()
            }
        }
    }
}
```

### View Modifiers (Tailwind-like convenience)

```swift
extension View {
    func cardStyle() -> some View {
        self
            .padding(Spacing.lg)
            .background(Color.surfaceSecondary)
            .clipShape(RoundedRectangle(cornerRadius: CornerRadius.md))
    }

    func inputStyle() -> some View {
        self
            .padding(.horizontal, Spacing.lg)
            .padding(.vertical, Spacing.md)
            .background(Color.surfaceTertiary)
            .clipShape(RoundedRectangle(cornerRadius: CornerRadius.md))
            .overlay(
                RoundedRectangle(cornerRadius: CornerRadius.md)
                    .stroke(Color.border, lineWidth: 0.5)
            )
    }

    func shimmer(_ isActive: Bool = true) -> some View {
        self.redacted(reason: isActive ? .placeholder : [])
            .shimmering(active: isActive)
    }
}

// Custom shimmer modifier
struct ShimmerModifier: ViewModifier {
    let active: Bool
    @State private var phase: CGFloat = 0

    func body(content: Content) -> some View {
        if active {
            content
                .overlay(
                    LinearGradient(
                        colors: [.clear, .white.opacity(0.3), .clear],
                        startPoint: .leading,
                        endPoint: .trailing
                    )
                    .offset(x: phase)
                    .mask(content)
                )
                .onAppear {
                    withAnimation(.linear(duration: 1.5).repeatForever(autoreverses: false)) {
                        phase = 200
                    }
                }
        } else {
            content
        }
    }
}

extension View {
    func shimmering(active: Bool = true) -> some View {
        modifier(ShimmerModifier(active: active))
    }
}
```

---

## Navigation

### NavigationStack (iOS 16+, recommended)

```swift
@Observable
final class Router {
    var path = NavigationPath()

    func push<D: Hashable>(_ destination: D) {
        path.append(destination)
    }

    func pop() {
        path.removeLast()
    }

    func popToRoot() {
        path.removeLast(path.count)
    }
}

// App entry
struct MyAppApp: App {
    @State private var router = Router()

    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $router.path) {
                HomeView()
                    .navigationDestination(for: Route.self) { route in
                        switch route {
                        case .chat(let id): ChatView(conversationId: id)
                        case .settings: SettingsView()
                        case .profile(let userId): ProfileView(userId: userId)
                        }
                    }
            }
            .environment(router)
        }
    }
}

enum Route: Hashable {
    case chat(id: String)
    case settings
    case profile(userId: String)
}
```

### Tab-Based Navigation

```swift
struct ContentView: View {
    @State private var selectedTab = 0

    var body: some View {
        TabView(selection: $selectedTab) {
            Tab("Home", systemImage: "house.fill", value: 0) {
                HomeView()
            }
            Tab("Chat", systemImage: "bubble.left.and.bubble.right.fill", value: 1) {
                ChatView()
            }
            Tab("Settings", systemImage: "gearshape.fill", value: 2) {
                SettingsView()
            }
        }
    }
}
```

### Sheet / Modal Presentation

```swift
struct ParentView: View {
    @State private var showLogin = false
    @State private var selectedItem: Item?

    var body: some View {
        Button("Login") { showLogin = true }
            .sheet(isPresented: $showLogin) {
                LoginSheet()
                    .presentationDetents([.medium, .large])
                    .presentationDragIndicator(.visible)
            }
            .sheet(item: $selectedItem) { item in
                ItemDetailSheet(item: item)
            }
    }
}
```

---

## Networking

### Modern Async/Await API Client

```swift
import Foundation

final class APIClient {
    static let shared = APIClient()

    private let session: URLSession
    private let baseURL: URL
    private let decoder: JSONDecoder

    init(
        baseURL: URL = URL(string: "https://api.example.com")!,
        session: URLSession = .shared
    ) {
        self.baseURL = baseURL
        self.session = session
        self.decoder = JSONDecoder()
        self.decoder.keyDecodingStrategy = .convertFromSnakeCase
        self.decoder.dateDecodingStrategy = .iso8601
    }

    // MARK: - Generic Request

    func request<T: Decodable>(
        _ endpoint: Endpoint,
        as type: T.Type = T.self
    ) async throws -> T {
        var request = URLRequest(url: baseURL.appending(path: endpoint.path))
        request.httpMethod = endpoint.method.rawValue
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")

        // Auth
        if let token = AuthManager.shared.accessToken {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }

        // Body
        if let body = endpoint.body {
            request.httpBody = try JSONEncoder().encode(body)
        }

        // Query parameters
        if let params = endpoint.queryItems {
            var components = URLComponents(url: request.url!, resolvingAgainstBaseURL: false)!
            components.queryItems = params
            request.url = components.url
        }

        let (data, response) = try await session.data(for: request)

        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }

        guard 200..<300 ~= httpResponse.statusCode else {
            throw NetworkError.httpError(statusCode: httpResponse.statusCode, data: data)
        }

        return try decoder.decode(T.self, from: data)
    }

    // MARK: - Streaming (SSE)

    func stream(_ endpoint: Endpoint) -> AsyncThrowingStream<String, Error> {
        AsyncThrowingStream { continuation in
            Task {
                var request = URLRequest(url: baseURL.appending(path: endpoint.path))
                request.httpMethod = "POST"
                request.setValue("application/json", forHTTPHeaderField: "Content-Type")
                request.setValue("text/event-stream", forHTTPHeaderField: "Accept")

                if let token = AuthManager.shared.accessToken {
                    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
                }

                if let body = endpoint.body {
                    request.httpBody = try? JSONEncoder().encode(body)
                }

                do {
                    let (bytes, response) = try await session.bytes(for: request)
                    guard let httpResponse = response as? HTTPURLResponse,
                          200..<300 ~= httpResponse.statusCode else {
                        continuation.finish(throwing: NetworkError.invalidResponse)
                        return
                    }

                    for try await line in bytes.lines {
                        if line.hasPrefix("data: ") {
                            let data = String(line.dropFirst(6))
                            if data == "[DONE]" { break }
                            continuation.yield(data)
                        }
                    }
                    continuation.finish()
                } catch {
                    continuation.finish(throwing: error)
                }
            }
        }
    }
}

// MARK: - Endpoint

struct Endpoint {
    let path: String
    let method: HTTPMethod
    let body: (any Encodable)?
    let queryItems: [URLQueryItem]?

    init(path: String, method: HTTPMethod = .get, body: (any Encodable)? = nil, queryItems: [URLQueryItem]? = nil) {
        self.path = path
        self.method = method
        self.body = body
        self.queryItems = queryItems
    }
}

enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
    case put = "PUT"
    case patch = "PATCH"
    case delete = "DELETE"
}

// MARK: - Typed Endpoints

extension Endpoint {
    static func sendMessage(_ text: String, conversationId: String?) -> Endpoint {
        Endpoint(path: "/chat/stream", method: .post, body: ChatRequest(
            message: text,
            conversationId: conversationId
        ))
    }

    static func getConversations(limit: Int = 50) -> Endpoint {
        Endpoint(path: "/conversations", queryItems: [
            URLQueryItem(name: "limit", value: "\(limit)")
        ])
    }

    static var me: Endpoint {
        Endpoint(path: "/users/me")
    }
}

// MARK: - Errors

enum NetworkError: LocalizedError {
    case invalidResponse
    case httpError(statusCode: Int, data: Data)
    case decodingError(Error)
    case noConnection

    var errorDescription: String? {
        switch self {
        case .invalidResponse: "Invalid server response"
        case .httpError(let code, _): "Server error (\(code))"
        case .decodingError: "Failed to parse response"
        case .noConnection: "No internet connection"
        }
    }
}
```

---

## State Management

### @Observable (iOS 17+) — Preferred

```swift
@Observable
final class AppState {
    var user: User?
    var isAuthenticated: Bool { user != nil }
    var conversations: [Conversation] = []
    var selectedConversation: Conversation?
}

// Inject via environment
struct MyApp: App {
    @State private var appState = AppState()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(appState)
        }
    }
}

// Consume
struct HomeView: View {
    @Environment(AppState.self) private var appState

    var body: some View {
        if let user = appState.user {
            Text("Hello, \(user.name)")
        }
    }
}
```

### Combine (iOS 13+, legacy but common)

```swift
final class AuthManager: ObservableObject {
    static let shared = AuthManager()

    @Published var currentUser: User?
    @Published var accessToken: String?

    var isAuthenticated: Bool { currentUser != nil }

    func signIn(email: String, password: String) async throws {
        // Supabase auth, Firebase, custom backend, etc.
    }

    func signOut() {
        currentUser = nil
        accessToken = nil
    }
}
```

---

## Data Persistence

### SwiftData (iOS 17+, recommended)

```swift
import SwiftData

@Model
final class Conversation {
    var id: String
    var title: String
    var createdAt: Date
    var updatedAt: Date
    @Relationship(deleteRule: .cascade) var messages: [ChatMessage]

    init(id: String = UUID().uuidString, title: String) {
        self.id = id
        self.title = title
        self.createdAt = .now
        self.updatedAt = .now
        self.messages = []
    }
}

@Model
final class ChatMessage {
    var id: String
    var role: String   // "user" | "assistant"
    var content: String
    var timestamp: Date
    var conversation: Conversation?

    init(role: String, content: String) {
        self.id = UUID().uuidString
        self.role = role
        self.content = content
        self.timestamp = .now
    }
}

// Setup in App
@main
struct MyAppApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: [Conversation.self, ChatMessage.self])
    }
}

// Query in View
struct ConversationListView: View {
    @Query(sort: \Conversation.updatedAt, order: .reverse) var conversations: [Conversation]
    @Environment(\.modelContext) private var context

    var body: some View {
        List(conversations) { conversation in
            Text(conversation.title)
        }
        .onDelete { indexSet in
            for index in indexSet {
                context.delete(conversations[index])
            }
        }
    }
}
```

### UserDefaults (Simple preferences)

```swift
extension UserDefaults {
    var hasCompletedOnboarding: Bool {
        get { bool(forKey: "hasCompletedOnboarding") }
        set { set(newValue, forKey: "hasCompletedOnboarding") }
    }

    var selectedModel: String {
        get { string(forKey: "selectedModel") ?? "gpt-4" }
        set { set(newValue, forKey: "selectedModel") }
    }
}

// SwiftUI binding via @AppStorage
struct SettingsView: View {
    @AppStorage("selectedModel") private var selectedModel = "gpt-4"
    @AppStorage("hapticFeedback") private var hapticFeedback = true

    var body: some View {
        Form {
            Picker("Model", selection: $selectedModel) {
                Text("GPT-4").tag("gpt-4")
                Text("Claude").tag("claude-sonnet")
            }
            Toggle("Haptic Feedback", isOn: $hapticFeedback)
        }
    }
}
```

### Keychain (Secrets — tokens, API keys)

```swift
import Security

enum KeychainService {
    static func save(_ data: Data, for key: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecValueData as String: data
        ]
        SecItemDelete(query as CFDictionary)
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else {
            throw KeychainError.saveFailed(status)
        }
    }

    static func load(for key: String) throws -> Data? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        guard status == errSecSuccess else {
            if status == errSecItemNotFound { return nil }
            throw KeychainError.loadFailed(status)
        }
        return result as? Data
    }

    static func delete(for key: String) {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key
        ]
        SecItemDelete(query as CFDictionary)
    }

    enum KeychainError: Error {
        case saveFailed(OSStatus)
        case loadFailed(OSStatus)
    }
}
```

---

## Authentication (Supabase Example)

```swift
import Supabase

final class AuthManager: ObservableObject {
    static let shared = AuthManager()

    let client = SupabaseClient(
        supabaseURL: URL(string: ProcessInfo.processInfo.environment["SUPABASE_URL"] ?? "")!,
        supabaseKey: ProcessInfo.processInfo.environment["SUPABASE_ANON_KEY"] ?? ""
    )

    @Published var session: Session?

    var isAuthenticated: Bool { session != nil }

    init() {
        Task { await observeAuthChanges() }
    }

    func observeAuthChanges() async {
        for await (event, session) in client.auth.authStateChanges {
            await MainActor.run {
                self.session = session
            }
        }
    }

    func signInWithGoogle() async throws {
        try await client.auth.signInWithOAuth(.google) { url in
            // Open URL in ASWebAuthenticationSession
            await UIApplication.shared.open(url)
        }
    }

    func signInWithEmail(_ email: String, password: String) async throws {
        try await client.auth.signIn(email: email, password: password)
    }

    func signUp(email: String, password: String) async throws {
        try await client.auth.signUp(email: email, password: password)
    }

    func signOut() async throws {
        try await client.auth.signOut()
    }
}
```

---

## Common UI Patterns

### Chat Interface

```swift
struct ChatView: View {
    @State private var viewModel = ChatViewModel()
    @FocusState private var isInputFocused: Bool

    var body: some View {
        VStack(spacing: 0) {
            // Messages
            ScrollViewReader { proxy in
                ScrollView {
                    LazyVStack(spacing: Spacing.sm) {
                        ForEach(viewModel.messages) { msg in
                            MessageBubble(message: msg)
                                .id(msg.id)
                        }

                        if viewModel.isLoading {
                            TypingIndicator()
                                .id("typing")
                        }
                    }
                    .padding()
                }
                .scrollDismissesKeyboard(.interactively)
                .onChange(of: viewModel.messages.count) {
                    withAnimation(.easeOut(duration: 0.2)) {
                        proxy.scrollTo(viewModel.isLoading ? "typing" : viewModel.messages.last?.id, anchor: .bottom)
                    }
                }
            }

            Divider()

            // Input
            HStack(alignment: .bottom, spacing: Spacing.sm) {
                TextField("Message", text: $viewModel.inputText, axis: .vertical)
                    .lineLimit(1...5)
                    .textFieldStyle(.plain)
                    .padding(.horizontal, Spacing.md)
                    .padding(.vertical, Spacing.sm)
                    .background(Color.surfaceTertiary)
                    .clipShape(RoundedRectangle(cornerRadius: CornerRadius.lg))
                    .focused($isInputFocused)

                Button {
                    Task { await viewModel.send() }
                } label: {
                    Image(systemName: "arrow.up.circle.fill")
                        .font(.system(size: 32))
                        .foregroundStyle(viewModel.inputText.isEmpty ? .tertiary : Color.brand)
                }
                .disabled(viewModel.inputText.isEmpty || viewModel.isLoading)
            }
            .padding(.horizontal, Spacing.md)
            .padding(.vertical, Spacing.sm)
        }
    }
}

struct MessageBubble: View {
    let message: Message
    private var isUser: Bool { message.role == .user }

    var body: some View {
        HStack {
            if isUser { Spacer(minLength: 60) }

            Text(message.content)
                .font(AppFont.body)
                .foregroundStyle(isUser ? .white : Color.textPrimary)
                .padding(.horizontal, Spacing.lg)
                .padding(.vertical, Spacing.md)
                .background(isUser ? Color.brand : Color.surfaceSecondary)
                .clipShape(RoundedRectangle(cornerRadius: CornerRadius.lg))

            if !isUser { Spacer(minLength: 60) }
        }
    }
}

struct TypingIndicator: View {
    @State private var phase = 0.0

    var body: some View {
        HStack(spacing: 4) {
            ForEach(0..<3, id: \.self) { i in
                Circle()
                    .fill(Color.textMuted)
                    .frame(width: 8, height: 8)
                    .offset(y: sin(phase + Double(i) * .pi / 3) * 4)
            }
        }
        .frame(maxWidth: .infinity, alignment: .leading)
        .padding()
        .onAppear {
            withAnimation(.easeInOut(duration: 0.6).repeatForever(autoreverses: false)) {
                phase = .pi * 2
            }
        }
    }
}
```

### Pull to Refresh + Infinite Scroll

```swift
struct ConversationListView: View {
    @State private var viewModel = ConversationListViewModel()

    var body: some View {
        List {
            ForEach(viewModel.conversations) { conversation in
                ConversationRow(conversation: conversation)
                    .task {
                        if conversation == viewModel.conversations.last {
                            await viewModel.loadMore()
                        }
                    }
            }

            if viewModel.isLoadingMore {
                ProgressView()
                    .frame(maxWidth: .infinity)
                    .listRowSeparator(.hidden)
            }
        }
        .refreshable {
            await viewModel.refresh()
        }
    }
}
```

### Empty State

```swift
struct EmptyState: View {
    let icon: String
    let title: String
    let message: String
    var action: (() -> Void)?
    var actionTitle: String?

    var body: some View {
        ContentUnavailableView {
            Label(title, systemImage: icon)
        } description: {
            Text(message)
        } actions: {
            if let action, let actionTitle {
                Button(actionTitle, action: action)
                    .buttonStyle(.borderedProminent)
            }
        }
    }
}
```

### Search

```swift
struct SearchableListView: View {
    @State private var searchText = ""
    @State private var items: [Item] = []

    var filteredItems: [Item] {
        if searchText.isEmpty { return items }
        return items.filter { $0.title.localizedCaseInsensitiveContains(searchText) }
    }

    var body: some View {
        List(filteredItems) { item in
            Text(item.title)
        }
        .searchable(text: $searchText, prompt: "Search items")
    }
}
```

---

## Package Management

### Swift Package Manager (preferred)

In Xcode: File > Add Package Dependencies, or add to `Package.swift`:

```swift
// Package.swift (for modular app or library)
// swift-tools-version: 6.0
import PackageDescription

let package = Package(
    name: "MyApp",
    platforms: [.iOS(.v17)],
    dependencies: [
        .package(url: "https://github.com/supabase/supabase-swift", from: "2.0.0"),
        .package(url: "https://github.com/onevcat/Kingfisher", from: "8.0.0"),
        .package(url: "https://github.com/pointfreeco/swift-composable-architecture", from: "1.0.0"),
    ],
    targets: [
        .target(name: "MyApp", dependencies: [
            .product(name: "Supabase", package: "supabase-swift"),
            .product(name: "Kingfisher", package: "Kingfisher"),
        ]),
    ]
)
```

### Common iOS Packages

| Package | Purpose | URL |
|---------|---------|-----|
| Supabase | BaaS (auth, DB, storage) | `supabase/supabase-swift` |
| Kingfisher | Image loading + cache | `onevcat/Kingfisher` |
| Alamofire | Networking (if URLSession isn't enough) | `Alamofire/Alamofire` |
| SwiftLint | Code style enforcement | `realm/SwiftLint` |
| Lottie | Animations | `airbnb/lottie-ios` |
| TCA | Composable Architecture | `pointfreeco/swift-composable-architecture` |
| KeychainAccess | Keychain wrapper | `kishikawakatsumi/KeychainAccess` |
| RevenueCat | In-app purchases | `RevenueCat/purchases-ios-spm` |
| PostHog | Analytics | `PostHog/posthog-ios` |
| Sentry | Crash reporting | `getsentry/sentry-cocoa` |

---

## Testing

### Unit Tests

```swift
import XCTest
@testable import MyApp

final class ChatViewModelTests: XCTestCase {
    var sut: ChatViewModel!
    var mockService: MockChatService!

    override func setUp() {
        mockService = MockChatService()
        sut = ChatViewModel(chatService: mockService)
    }

    func testSendMessage_appendsUserMessage() async {
        sut.inputText = "Hello"
        mockService.response = Message(role: .assistant, content: "Hi!")

        await sut.send()

        XCTAssertEqual(sut.messages.count, 2)
        XCTAssertEqual(sut.messages[0].role, .user)
        XCTAssertEqual(sut.messages[0].content, "Hello")
        XCTAssertEqual(sut.messages[1].role, .assistant)
    }

    func testSendMessage_clearsInput() async {
        sut.inputText = "Hello"
        mockService.response = Message(role: .assistant, content: "Hi!")

        await sut.send()

        XCTAssertTrue(sut.inputText.isEmpty)
    }

    func testSendMessage_emptyInput_doesNothing() async {
        sut.inputText = "   "
        await sut.send()
        XCTAssertTrue(sut.messages.isEmpty)
    }
}
```

### SwiftUI Preview

```swift
#Preview {
    ChatView()
        .environment(AppState())
}

#Preview("Dark Mode") {
    ChatView()
        .environment(AppState())
        .preferredColorScheme(.dark)
}

#Preview("Message Bubble") {
    VStack {
        MessageBubble(message: .init(role: .user, content: "Hello!"))
        MessageBubble(message: .init(role: .assistant, content: "Hi there! How can I help?"))
    }
    .padding()
}
```

---

## App Store & Distribution

### Info.plist Keys (Common)

```xml
<!-- Camera access -->
<key>NSCameraUsageDescription</key>
<string>Take photos to share in chat</string>

<!-- Photo library -->
<key>NSPhotoLibraryUsageDescription</key>
<string>Select photos to share in chat</string>

<!-- Location -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>Find nearby services</string>

<!-- Microphone -->
<key>NSMicrophoneUsageDescription</key>
<string>Record voice messages</string>

<!-- Face ID -->
<key>NSFaceIDUsageDescription</key>
<string>Unlock the app with Face ID</string>
```

### App Icons

Place in `Assets.xcassets/AppIcon.appiconset/`. Xcode 15+ only needs a single 1024x1024 image.

### Launch Screen

Use `Info.plist` storyboard or SwiftUI-based launch screen:
```xml
<key>UILaunchScreen</key>
<dict>
    <key>UIColorName</key>
    <string>LaunchBackground</string>
    <key>UIImageName</key>
    <string>LaunchLogo</string>
</dict>
```

---

## Xcode Tips for Claude Code

When working with Xcode projects from the terminal/Claude Code:

```bash
# Build from CLI
xcodebuild -project MyApp.xcodeproj -scheme MyApp -sdk iphonesimulator build

# Run tests
xcodebuild test -project MyApp.xcodeproj -scheme MyApp -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16'

# Clean
xcodebuild clean -project MyApp.xcodeproj -scheme MyApp

# List schemes
xcodebuild -list -project MyApp.xcodeproj

# Open in Xcode
open MyApp.xcodeproj

# Generate xcodeproj from Package.swift (for SPM-based projects)
swift package generate-xcodeproj

# Resolve packages
swift package resolve

# Format Swift code (if SwiftFormat installed)
swift-format format --in-place --recursive Sources/
```

### Working with .pbxproj

The `.pbxproj` file is Xcode's project file. When adding new Swift files from Claude Code:

1. **Create the `.swift` file** in the correct directory
2. **Tell the user to add it to the Xcode project** (drag into Xcode navigator, or use `xcodegen` / `tuist` for automated project generation)
3. Or use **SPM-based project structure** where files are auto-discovered

### Recommended: Use Tuist or XcodeGen

For CLI-friendly project management:

```bash
# Tuist (recommended)
brew install tuist
tuist init --platform ios
tuist generate          # Generates .xcodeproj from Project.swift

# XcodeGen
brew install xcodegen
xcodegen generate       # Generates .xcodeproj from project.yml
```

---

## Swift 6 Concurrency & Sendable

```swift
// Mark types as Sendable when crossing concurrency boundaries
struct Message: Codable, Identifiable, Sendable {
    let id: String
    let role: Role
    let content: String
    let timestamp: Date

    enum Role: String, Codable, Sendable {
        case user, assistant, system
    }
}

// Use @MainActor for UI-bound classes
@MainActor
@Observable
final class ChatViewModel {
    var messages: [Message] = []
    // ...
}

// Structured concurrency
func loadData() async {
    async let user = fetchUser()
    async let conversations = fetchConversations()

    let (u, c) = await (try? user, try? conversations)
    self.user = u
    self.conversations = c ?? []
}
```

---

## Quick Reference

| Web/Tailwind | SwiftUI Equivalent |
|---|---|
| `flex` | `HStack` / `VStack` / `ZStack` |
| `flex-col` | `VStack` |
| `flex-row` | `HStack` |
| `grid` | `LazyVGrid` / `LazyHGrid` |
| `gap-4` | `.spacing(16)` on Stack or `.padding()` |
| `p-4` | `.padding(16)` or `.padding()` |
| `px-4` | `.padding(.horizontal, 16)` |
| `rounded-xl` | `.clipShape(RoundedRectangle(cornerRadius: 12))` |
| `rounded-full` | `.clipShape(Capsule())` or `.clipShape(Circle())` |
| `bg-gray-100` | `.background(Color(.systemGray6))` |
| `text-sm` | `.font(.caption)` |
| `text-lg` | `.font(.title3)` |
| `font-bold` | `.fontWeight(.bold)` |
| `text-center` | `.multilineTextAlignment(.center)` |
| `opacity-50` | `.opacity(0.5)` |
| `shadow-md` | `.shadow(radius: 4)` |
| `hidden` | `.hidden()` or conditional `if` |
| `overflow-scroll` | `ScrollView` |
| `absolute` / `relative` | `ZStack` with `.offset()` or `GeometryReader` |
| `transition` | `.animation(.easeInOut, value: state)` |
| `hover:` | `.onHover { }` (macOS) / no direct equivalent on iOS |
| `dark:` | `@Environment(\.colorScheme)` or adaptive `Color(.label)` |
| `max-w-lg` | `.frame(maxWidth: 512)` |
| `w-full` | `.frame(maxWidth: .infinity)` |
| `border` | `.overlay(RoundedRectangle(...).stroke(...))` |
| `divide-y` | Use `List` with built-in dividers, or manual `Divider()` |
| `truncate` | `.lineLimit(1)` |
| `sr-only` | `.accessibilityHidden(true)` on visual, `.accessibilityLabel()` on interactive |

---

## Minimum Deployment Targets

| Feature | Minimum iOS |
|---------|------------|
| SwiftUI | 13.0 |
| Combine | 13.0 |
| async/await | 15.0 |
| NavigationStack | 16.0 |
| Charts (Swift Charts) | 16.0 |
| @Observable | 17.0 |
| SwiftData | 17.0 |
| #Preview macro | 17.0 |
| String Catalogs | 17.0 |
| TipKit | 17.0 |
| Interactive Widgets | 17.0 |
| Custom container views | 18.0 |
| Liquid Glass | 26.0 |

**Recommended minimum: iOS 17** — gives you @Observable, SwiftData, #Preview, and modern APIs while covering 90%+ of active devices.
