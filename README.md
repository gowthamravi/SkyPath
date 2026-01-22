## AI Agent Workflow

When working on JIRA tickets or implementing new features, **ALWAYS** follow the comprehensive workflow documented in:
**[AGENT_WORKFLOW.md](AGENT_WORKFLOW.md)**

This workflow ensures:
- Proper MVVM architecture analysis before implementation
- Alignment with existing code patterns and project structure
- Comprehensive testing and quality standards
- Proper PR generation and documentation

## Swift/iOS Implementation References

When implementing code, refer to the guides in the `references/` folder for best practices:

| Guide | Use When |
|-------|----------|
| [async-await-basics.md](references/async-await-basics.md) | Implementing async operations |
| [actors.md](references/actors.md) | Managing shared mutable state |
| [sendable.md](references/sendable.md) | Passing data across concurrency boundaries |
| [tasks.md](references/tasks.md) | Creating and managing Swift Tasks |
| [async-sequences.md](references/async-sequences.md) | Working with AsyncSequence |
| [threading.md](references/threading.md) | Thread safety and main actor usage |
| [memory-management.md](references/memory-management.md) | Avoiding retain cycles and leaks |
| [core-data.md](references/core-data.md) | Core Data operations |
| [testing.md](references/testing.md) | Writing unit and integration tests |
| [performance.md](references/performance.md) | Optimizing app performance |
| [migration.md](references/migration.md) | Migrating legacy code patterns |
| [linting.md](references/linting.md) | Code style and linting rules |
| [glossary.md](references/glossary.md) | Terminology reference |

## Coding Standards

### Swift Style
- Use Swift 6 strict concurrency
- Prefer `@Observable` over `ObservableObject`
- Use `async/await` for all async operations
- Follow Apple's Swift API Design Guidelines
- Use `guard` for early exits
- Prefer value types (structs) over reference types (classes)

### SwiftUI Patterns
- Extract views when they exceed 100 lines
- Use `@State` for local view state only
- Use `@Environment` for dependency injection
- Prefer `NavigationStack` over deprecated `NavigationView`
- Use `@Bindable` for bindings to @Observable objects

### Navigation Pattern
```swift
// Use NavigationStack with type-safe routing
enum Route: Hashable {
    case detail(Item)
    case settings
}

NavigationStack(path: $router.path) {
    ContentView()
        .navigationDestination(for: Route.self) { route in
            // Handle routing
        }
}
```

## Project Overview

FlightSearch is a SwiftUI-based iOS application for searching one-way flights. The app uses the Booking.com RapidAPI for flight search functionality and displays available flights with departure details.

## Building and Running

### Build the project
```bash
xcodebuild -project FlightSearch.xcodeproj -scheme FlightSearch -configuration Debug build
```

### Run in simulator
```bash
xcodebuild -project FlightSearch.xcodeproj -scheme FlightSearch -destination 'platform=iOS Simulator,name=iPhone 15' build
```

### Clean build
```bash
xcodebuild -project FlightSearch.xcodeproj -scheme FlightSearch clean
```

Note: This project uses Swift Package Manager (SPM) for dependencies. Xcode will automatically resolve packages on first build.

## Architecture

### Application Flow

The app follows this navigation flow:
1. **GuestView** - Entry point with "Flightly" branding and splash screen
2. **MainView** - TabView container with three tabs (Home, Favorites, Profile)
3. **FlightSearchView** - Main search interface where users input travel details
4. **FlightsListView** - Results displayed in a modal sheet

### Key Components

**State Management**
- `FlightSearch` (ObservableObject) - Central state manager that holds:
  - `allStations`: Dictionary mapping airport codes to Station objects (loaded from local airport.json)
  - `origin`/`destination`: Selected Station objects
  - `fromDate`: Travel date
  - `passengersList`: Tuple of passenger counts (adult, teen, children, infants)
  - `flights`: Search results
  - Handles flight search API calls via `FlightSearchingService`

- `Router` (Singleton ObservableObject) - Manages NavigationPath for programmatic navigation

**Networking Architecture**
- Uses custom `ServiceHandler` package (external dependency)
- `FlightSearchingService` implements `FlightSearching` protocol
- Two API endpoints defined in `Requests.swift`:
  - Station lists (from S3 bucket) - currently not used, loads from local JSON instead
  - Flight search (Booking.com RapidAPI)
- Base URLs defined in `BaseURL.swift` using custom `#URL` macro from SwiftMacros package
- API key stored in `Header.headers` in Requests.swift (RapidAPI credentials)

**Data Models**
- `Station` - Airport/station information (code, name, country)
- `FlightSearchResponse` - API response wrapper containing flights array
- `Flights` - Flight object with bounds (flight segments)
- `Bounds`/`Segments` - Nested structure for flight details (origin, destination, times, baggage, carrier)

### View Structure

**FlightSearchView** components:
- `StationView` - Airport selection (origin/destination)
- `DateView` - Date picker for travel date
- `PassengerView` - Stepper controls for passenger counts
- `LetsGoView` - Search button that triggers API call and shows results sheet

**Extensions**
The app uses numerous SwiftUI view modifiers and extensions in the Extensions/ folder:
- Custom button styles
- Date formatting extensions
- Loading indicators (ActivityIndicatorView package)
- Custom bordered view modifiers
- Navigation path helpers

## Dependencies (Swift Package Manager)

- **ServiceHandler** (1.0.1) - Custom networking layer from github.com/gowthamravi/ServiceHandler-main
- **ActivityIndicatorView** (1.1.1) - Loading spinner UI from github.com/exyte/ActivityIndicatorView
- **SwiftMacros** (main branch) - Custom macro support from github.com/bernndr/swift-macros
- **CodingKeysGenerator** (0.1.4) - Code generation for Codable keys
- **swift-syntax** (509.1.1) - Apple's Swift syntax library

## Developer Configuration

### API Keys and Credentials
API keys and credentials are stored in `FlightSearch/Config.swift` (gitignored).

**Setup**:
1. Copy `Config.swift.template` to `Config.swift`
2. Add your personal API keys and tokens
3. Never commit Config.swift to git

### Project References in Config
The Config.swift file also stores project-specific references:
- **Jira**: Project key and related task IDs for tracking development tasks
- **Figma**: Design file URLs for quick access to specifications
- **GitHub**: Repository information for git operations

### Configuration Structure
```swift
struct APIConfig {
    // API Credentials
    static let rapidAPIKey: String
    static let jiraAPIToken: String
    static let figmaAccessToken: String
    static let githubAccessToken: String

    // Project References
    static let jiraProjectKey: String
    static let jiraTaskReferences: [String: String]
    static let figmaFileURLs: [String: String]
    static let githubRepoURL: String
}
```

See `API_SETUP.md` for detailed setup instructions.

## Important Notes

### API Configuration
- RapidAPI credentials are now in `Config.swift` (gitignored) via `APIConfig` struct
- Requests.swift uses `APIConfig.rapidAPIKey` and `APIConfig.rapidAPIHost`
- The app uses Booking.com's flight search API via RapidAPI
- Base URLs use a custom `#URL` macro from the SwiftMacros package

### Data Loading
- Airport data is loaded from a local `airport.json` file in the bundle at app start
- The remote station list API endpoint exists but is commented out in favor of local data
- Station dictionary is built in `FlightSearch.start()` method

### Navigation Pattern
- Uses `@EnvironmentObject` to pass `FlightSearch` and `Router` down the view hierarchy
- `Router.shared` singleton manages programmatic navigation with NavigationPath
- Results are presented as modal sheets, not pushed navigation

### UI Patterns
- Custom font "Avenir-HeavyOblique" used for branding
- Custom button style `FlightlyButton()` defined in Extensions
- Loading indicator shown during flight search
- Empty state handled by `NoFlightView` when no results
