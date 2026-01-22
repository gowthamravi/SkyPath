# AI Agent Workflow: Flightly MVVM Task Implementation

## Phase 1: Discovery & Planning

### Step 1: Ticket Analysis and MVVM Context Discovery

**📚 Reference:** Review [core-data.md](references/core-data.md) if task involves data persistence.

1. **Analyze Ticket Requirements**:
* Extract core requirements from the JIRA ticket ID provided in the `/start-ticket` command.
* Identify if the task requires a new **View**, modifications to an existing **ViewModel**, or updates to the **Model/Service** layer.


2. **Map the MVVM Chain**:
* **View**: Identify the SwiftUI Views or UIKit ViewControllers that need to observe changes.
* **ViewModel**: Search for the corresponding ViewModel. Identify the `@Published` properties (SwiftUI) or Combine/Observable patterns currently in use.
* **Model/Service**: Locate the Data Models and API/Repository Services that provide data to the ViewModel.


3. **Codebase Exploration**:
* Search for existing Repository or Service patterns to ensure the new implementation follows the project's dependency injection style.
* Check for existing Protocols used for mocking Services in unit tests.
* Ensure the proposed changes adhere to **iOS 16.0+** standards (e.g., using `NavigationStack` instead of `NavigationView`).


4. **Initial Findings & Alignment**:
* Summarize the impact: *"I will be adding/modifying the [Name]ViewModel to handle [X] and updating [Name]View to reflect these changes."*
* **Ask**: *"I've mapped out the MVVM components for ticket (((ticket_id))). I plan to interface with the [ExistingService] to fetch the required data. Does this architectural approach match your project's implementation of MVVM?"*
* Look for existing code patterns in these modules when planning the implementation.



### Step 2: Fetch Figma Designs and Assets

If the ticket includes a Figma link:

**IMPORTANT: Do NOT ask permission for any Figma operations. Proceed automatically.**

#### 2.1 Download Full Screen Image First
* Export the entire screen/frame as PNG at 2x scale
* Save to `/tmp/figma_design.png` for reference
* **This is your source of truth** - always compare against this image

#### 2.2 Extract COMPLETE Specs BEFORE Writing Any Code

**CRITICAL: Do NOT write a single line of code until you have extracted ALL of the following:**

```
For EVERY component in the design, extract:
├── Position (x, y coordinates)
├── Size (width, height)
├── Colors
│   ├── Fill color (hex)
│   ├── Stroke/Border color (hex)
│   ├── Stroke width (px)
│   └── Text color (hex)
├── Corner radius
├── Padding (internal spacing)
├── Margin (spacing from other elements)
├── Font (size, weight)
└── Asset type (icon vs image vs vector)
```

**Create a specs document like this BEFORE coding:**

```markdown
## Figma Specs Extraction

### Screen Dimensions
- Width: 360px, Height: 800px

### Component: Header
- Background: #0085FF
- Height: 200px
- Horizontal padding: 20px

### Component: Profile Image
- Size: 36x36
- Border: 1px white stroke
- Type: IMAGE (not icon) - download from Figma

### Component: Search Card
- Background: #FFFFFF
- Corner radius: 10px
- Internal padding: 20px
- Margin from edges: 18px

### Component: From Field
- Height: 52px
- Border: 1px #EFEFEF
- Corner radius: 10px
- Left padding: 18px
- Icon: DepartureIcon (download from Figma)

### Component: Swap Button
- Size: 40x40
- Fill: #0085FF
- Position: RIGHT side of From/To, INSIDE the fields
- Vertical: Centered between From and To
- Horizontal: 32px from right edge of fields (INSIDE, not outside)

### Component: Return Date Field
- Border: 1px #0085FF (BLUE, different from other fields)

[... continue for ALL components]
```

#### 2.3 Identify ALL Assets to Download

**List ALL icons and images BEFORE downloading:**

| Asset Name | Type | Figma Node ID | Notes |
|------------|------|---------------|-------|
| ProfileImage | IMAGE | 1603:XXXX | Actual photo, not icon |
| NotificationIcon | ICON | 1603:XXXX | Bell icon |
| DepartureIcon | ICON | 1603:XXXX | Plane departure |
| ArrivalIcon | ICON | 1603:XXXX | Plane arrival |
| SwapArrowsIcon | ICON | 1603:XXXX | Two arrows |
| TravellersIcon | ICON | 1603:XXXX | Person icon |
| OfferBanner | IMAGE | 1603:XXXX | Promotional banner |
| AddIcon | ICON | 1603:XXXX | Plus icon |

#### 2.4 Download All Assets

* **ALWAYS use PNG format** - never PDF or SVG
* Download at **all 3 scales** (@1x, @2x, @3x) directly from Figma API
* Use `format=png&scale=1`, `scale=2`, `scale=3` parameters
* **Do NOT ask permission** - proceed automatically
* **Distinguish icons from images** - icons are vectors, images are photos/banners

#### 2.5 Create Asset Imagesets

* Create `.imageset` folders in `Assets.xcassets`
* Add all 3 PNG files per asset
* Generate proper `Contents.json` for each imageset
* Naming: `{AssetName}.imageset/` with `{AssetName}.png`, `{AssetName}@2x.png`, `{AssetName}@3x.png`

#### 2.6 Verify Hidden Elements

* Check `visible` property in Figma - some elements may be hidden (visible=false)
* Don't implement hidden elements unless specified

Example Figma API calls:
```bash
# Download full screen for reference
curl -H "X-Figma-Token: $TOKEN" \
  "https://api.figma.com/v1/images/{FILE_ID}?ids={SCREEN_NODE_ID}&format=png&scale=2"

# Get node structure with ALL dimensions, colors, positions
curl -H "X-Figma-Token: $TOKEN" \
  "https://api.figma.com/v1/files/{FILE_ID}/nodes?ids={NODE_ID}"

# Get image URLs at each scale
curl -H "X-Figma-Token: $TOKEN" \
  "https://api.figma.com/v1/images/{FILE_ID}?ids={NODE_IDS}&format=png&scale=1"
curl -H "X-Figma-Token: $TOKEN" \
  "https://api.figma.com/v1/images/{FILE_ID}?ids={NODE_IDS}&format=png&scale=2"
curl -H "X-Figma-Token: $TOKEN" \
  "https://api.figma.com/v1/images/{FILE_ID}?ids={NODE_IDS}&format=png&scale=3"
```

### Step 3: Create Implementation Plan

**📚 Reference:** Review [glossary.md](references/glossary.md) for terminology and [migration.md](references/migration.md) for legacy pattern considerations.

1. Based on the ticket analysis and any Figma details provided, create a detailed implementation plan using the **Todowrite** tool.
2. Break down the task into specific, actionable steps including:
* Files that need to be created or modified.
* Code changes required.
* Unit tests to be written.
* Any potential challenges or considerations.


3. Present the plan to the user and ask: *"I've created an implementation plan above. Does this approach look good to you, or would you like to adjust anything?"*
4. **Wait** for user approval before proceeding.

---

## Phase 2: Execution

### Step 4: Implementation

**📚 Implementation References - Consult these guides while coding:**

| Topic | Guide | When to Use |
|-------|-------|-------------|
| **Concurrency** | [async-await-basics.md](references/async-await-basics.md) | Any async operations, API calls |
| **Actors** | [actors.md](references/actors.md) | Shared mutable state management |
| **Sendable** | [sendable.md](references/sendable.md) | Passing data across concurrency boundaries |
| **Tasks** | [tasks.md](references/tasks.md) | Creating/managing async tasks |
| **Threading** | [threading.md](references/threading.md) | Main actor, thread safety |
| **Memory** | [memory-management.md](references/memory-management.md) | Closures, retain cycles, weak/unowned |
| **Performance** | [performance.md](references/performance.md) | Optimization, profiling |
| **Code Style** | [linting.md](references/linting.md) | Code conventions, formatting |

1. Once the plan is approved, walk through each todo item systematically.
2. **Write Code following project guidelines**:
* Use **Swift 6.2** language features.
* Support **iOS 16.0+** baseline.
* Follow **SOLID** principles.
* Never force-unwrap optionals.
* Follow existing code style and conventions.
* **DO NOT add any commentary comments in the code**—write clean, self-documenting code.


3. **IMPORTANT - New Swift File Creation**:
* When creating any new Swift files, you **MUST** add them to the Xcode project so they appear in the Xcode IDE.
* **For Tuist projects**: Add the file to the appropriate `Project.swift` target definition.
* **For standard Xcode projects**: Use `xcodebuild` or `.pbxproj` manipulation.
* Verify the file is properly referenced and visible in Xcode.


4. **Write Comprehensive Unit Tests**:
* Analyze existing test patterns in the codebase.
* Aim for a minimum of **80% coverage**.
* Use appropriate mocks and test doubles.
* Any new test files must also be added to the Xcode project.
* **DO NOT add commentary comments in test code.**


5. Mark each todo as completed as you finish it.

### Step 5: Verify Build and Tests

**📚 Reference:** Review [performance.md](references/performance.md) if build reveals performance warnings.

1. Build the project.
2. Run the unit tests.
3. Fix any compilation errors or test failures.
4. Only mark implementation as complete when the build succeeds and all tests pass.

### Step 6: Verify UI Against Figma (CRITICAL)

**Do NOT skip this step. Verify EVERY component against Figma.**

1. **Component-by-Component Verification**:
   * Open the Figma design image (`/tmp/figma_design.png`)
   * Compare EACH UI component against the Figma design
   * Check the following for every element:

   | Check | What to Verify |
   |-------|---------------|
   | **Position** | Is the element in the correct location? (left/right/center) |
   | **Size** | Does it match Figma dimensions? (width x height in pixels) |
   | **Colors** | Are fill colors, stroke colors, text colors correct? (hex values) |
   | **Icons** | Is the correct icon being used? Does it look like Figma? |
   | **Spacing** | Are margins and paddings correct? |
   | **Layout** | Is the layout structure correct? (HStack vs VStack vs ZStack) |
   | **Typography** | Font size, weight, color matching? |

2. **Common Mistakes to Avoid**:
   * Using system icons (SF Symbols) instead of Figma assets
   * Adding backgrounds/circles to icons when Figma shows none
   * Wrong layout container (HStack instead of ZStack for overlays)
   * Hardcoded colors that don't match Figma hex values
   * Elements in wrong positions (left vs right)
   * Missing or extra UI elements

3. **Create Verification Checklist**:
   ```
   [ ] Header - positions, icons, sizes correct
   [ ] Form fields - widths, borders, colors correct
   [ ] Buttons - sizes, colors, icons (or no icons) correct
   [ ] Icons - correct assets, no unwanted backgrounds
   [ ] Spacing - matches Figma layout
   [ ] Text - colors, sizes match Figma
   ```

4. **Fix All Discrepancies**:
   * If anything doesn't match, fix it BEFORE marking complete
   * Re-download assets if icons look wrong
   * Adjust dimensions to match Figma exactly

5. **Final Build Verification**:
   * Build again after fixes
   * Ensure no regressions

---

## Phase 3: Manual Testing & User Approval

### Step 7: Implementation Summary & Manual Testing Request

1. Provide a summary of what was implemented.
2. List all files created or modified.
3. **Ask the user**: *"Implementation complete! Please perform your manual testing to verify:*
   - *UI matches the Figma design*
   - *All functionality works as expected*
   - *No regressions in existing features*

   *Once you've verified everything works correctly, let me know and I'll proceed to write unit tests."*

4. **WAIT for user approval** before proceeding to unit tests.

---

## Phase 4: Unit Test Implementation

**📚 Reference:** Review [testing.md](references/testing.md) for comprehensive testing patterns and best practices.

### Step 8: Test Target Detection & Setup

**IMPORTANT: Check for existing test infrastructure BEFORE creating anything.**

#### 8.1 Detect Existing Test Target

```bash
# Check if test target exists
xcodebuild -list -project {PROJECT_NAME}.xcodeproj | grep -i "test"
```

**If test target EXISTS:**
- Use the existing test target name
- Check existing test patterns and conventions
- Add new tests following existing structure
- DO NOT recreate or modify test target configuration

**If test target does NOT exist:**
- Create new test target using the Ruby script approach
- Configure for `swift-testing` framework
- Set up proper build settings

#### 8.2 Detect Existing Mocks

```bash
# Search for existing mock files
find . -name "Mock*.swift" -o -name "*Mock.swift"
```

**If mocks exist:**
- Reuse existing mock implementations
- Extend mocks only if new methods are needed
- Follow existing mock patterns

**If mocks don't exist:**
- Create new mock directory: `{TestTarget}/Mocks/`
- Create mocks for all protocols used by the ViewModel

### Step 9: Analyze What to Test

1. **Identify the ViewModel(s) modified/created** in this ticket
2. **List all testable functionality:**
   - `@Published` properties and their initial states
   - Public methods and their behavior
   - State transitions (loading, error, success)
   - Validation logic
   - Data transformations

3. **Create test plan:**
   ```markdown
   ## Test Plan for [TicketID]

   ### ViewModel: [Name]ViewModel

   #### Initial State Tests
   - [ ] Property X is nil/empty on init
   - [ ] Property Y has default value Z

   #### Method Tests: methodName()
   - [ ] Success case - expected behavior
   - [ ] Failure case - error handling
   - [ ] Edge case - boundary conditions

   #### State Management Tests
   - [ ] Loading state transitions
   - [ ] Error message handling
   ```

### Step 10: Implement Unit Tests

**📚 References for Testing Async Code:**
- [testing.md](references/testing.md) - Test patterns, mocking, assertions
- [async-sequences.md](references/async-sequences.md) - Testing AsyncSequence
- [tasks.md](references/tasks.md) - Testing Task-based code

#### 10.1 Test File Structure

```
{ProjectName}Tests/
├── Mocks/
│   ├── Mock{Repository}Repository.swift
│   └── Mock{Service}Service.swift
├── TestData.swift
├── {Feature}ViewModelTests.swift
└── {Model}Tests.swift
```

#### 10.2 Test Implementation Guidelines

**Use `swift-testing` framework:**
```swift
import Testing
@testable import {ProjectName}

@Suite("{FeatureName} Tests")
@MainActor
struct {Feature}ViewModelTests {

    @Test("Description of what is being tested")
    func testMethodName() async {
        // Arrange
        let mockRepo = Mock{Repository}()
        let viewModel = {Feature}ViewModel(repository: mockRepo)

        // Act
        await viewModel.someMethod()

        // Assert
        #expect(viewModel.property == expectedValue)
    }
}
```

**Test naming convention:**
- Use descriptive test names that explain the scenario
- Group related tests in the same `@Suite`
- Use `@MainActor` for ViewModel tests

#### 10.3 Add Tests to Xcode Project

**If test target already exists:**
```bash
# Simply create the test files in the existing test directory
# Xcode will automatically pick them up
```

**If test target was newly created:**
```ruby
# Use xcodeproj gem to add files to target
test_target.source_build_phase.add_file_reference(file_ref)
```

### Step 11: Run and Verify Tests

1. **Build tests:**
   ```bash
   xcodebuild -project {Project}.xcodeproj -scheme {Scheme} build-for-testing \
     -destination 'platform=iOS Simulator,name=iPhone 16'
   ```

2. **Run tests:**
   ```bash
   xcodebuild test -project {Project}.xcodeproj -scheme {Scheme} \
     -destination 'platform=iOS Simulator,name=iPhone 16'
   ```

3. **Verify all tests pass** before proceeding

4. **If tests fail:**
   - Fix the failing tests
   - Re-run until all pass
   - Do NOT proceed with commit until all tests pass

### Step 12: Create Separate Test Commit

**CRITICAL: Tests must be in a SEPARATE commit from the implementation.**

```bash
# Stage only test-related files
git add {ProjectName}Tests/

# Create dedicated test commit
git commit -m "$(cat <<'EOF'
[{TICKET_ID}] Add unit tests for {Feature}

Tests added:
- {ViewModel}Tests: {X} tests covering {functionality}
- Mock implementations for {Repository/Service}

Test coverage: {X} tests, all passing

🤖 Generated with Cursor

Co-Authored-By: Cursor <noreply@cursor.com>
EOF
)"

# Push the test commit
git push
```

---

## Phase 5: Completion & PR Generation

### Step 13: Final Summary

1. Provide a complete summary including:
   - Implementation changes (from Phase 2)
   - Test coverage (from Phase 4)
   - Total commits in PR

2. **Show commit history:**
   ```
   Commits in this PR:
   1. [{TICKET_ID}] Implement {feature} - UI and functionality changes
   2. [{TICKET_ID}] Add unit tests for {feature} - {X} tests
   ```

### Step 14: Generate PR Description

1. **Ask the user**: *"Implementation and tests are complete. Would you like me to generate the PR description?"*

2. If user confirms, create PR with format:
```markdown
## Summary
- Implemented {feature} per Figma design
- Added {X} unit tests for {ViewModel}

## Changes
### Implementation (Commit 1)
- {List of implementation changes}

### Tests (Commit 2)
- {List of test files added}
- {X} tests covering {functionality}

## Test Plan
- [x] Unit tests pass ({X} tests)
- [ ] Manual UI verification
- [ ] Functionality testing

## Screenshots
{User to provide}

🤖 Generated with Cursor
```

### Step 15: Create Pull Request

1. **Ask**: *"Here's the PR description. Would you like me to create the pull request now?"*
2. If user confirms: Execute `gh pr create` with the description.

### Step 16: Add QA Test Plan to JIRA Ticket

**CRITICAL: After PR creation, ALWAYS add a QA test plan comment to the JIRA ticket.**

1. **Generate QA Test Plan** based on the implementation:
   - Pre-requisites (branch name, build instructions)
   - Visual verification checklist (UI elements to verify)
   - Functional testing checklist (features to test)
   - Regression testing checklist (existing features to verify)
   - Expected results

2. **Post comment to JIRA** using the API:

```bash
curl -s -u "$JIRA_EMAIL:$JIRA_TOKEN" \
  -X POST \
  -H "Content-Type: application/json" \
  --data '{
    "body": {
      "type": "doc",
      "version": 1,
      "content": [
        {
          "type": "heading",
          "attrs": {"level": 2},
          "content": [{"type": "text", "text": "🧪 QA Test Plan"}]
        },
        {
          "type": "paragraph",
          "content": [
            {"type": "text", "text": "PR: "},
            {"type": "inlineCard", "attrs": {"url": "{PR_URL}"}}
          ]
        },
        ... // Full test plan structure
      ]
    }
  }' \
  "https://gowthamr3.atlassian.net/rest/api/3/issue/{TICKET_ID}/comment"
```

3. **QA Test Plan Template Structure:**

```markdown
## 🧪 QA Test Plan

**PR:** [Link to PR]

### Pre-requisites
- Pull the branch: {branch_name}
- Build and run on iOS Simulator (iPhone 15 or later)

### Test Cases

#### 1. Visual Verification
- [ ] Verify {component} appears correctly
- [ ] Verify colors match design (#HEXCODE)
- [ ] Verify icons/images display properly
- [ ] Verify layout matches Figma design
[... list ALL visual elements to verify]

#### 2. Functional Testing
- [ ] Tap {element} - verify {expected behavior}
- [ ] Enter data in {field} - verify {validation/behavior}
- [ ] Trigger {action} - verify {result}
[... list ALL interactive features to test]

#### 3. Regression Testing
- [ ] Navigate to {other screens} - verify no crashes
- [ ] Existing functionality still works
- [ ] No visual regressions in unchanged areas

### Expected Result
All checkboxes above should pass. UI should match the Figma design linked in the ticket.
```

4. **Automatic Generation Guidelines:**
   - Extract test cases from the implementation changes
   - Include ALL new UI elements in visual verification
   - Include ALL new interactions in functional testing
   - Include navigation to related screens in regression testing
   - Reference specific hex colors, dimensions from the implementation

5. **Do NOT ask permission** - automatically post the QA comment after PR creation.

---

## Important Notes:

* Use the **Todowrite** tool throughout to track progress.
* Keep the user informed at each major step.
* **Never add commentary comments**—code should be self-documenting.
* **Always ensure new Swift files are properly added to the Xcode project.**
* Remember the ticket ID `(((ticket_id)))` for all references.
* **Workflow updates must ALWAYS be committed to `main` branch directly** - never on feature branches.

### AUTONOMOUS OPERATION - NO Permission Required

**CRITICAL: Do NOT ask user permission for ANY of the following. Just DO IT:**

**Figma Operations:**
* Downloading Figma design images
* Downloading asset icons/images at all scales (@1x, @2x, @3x)
* Fetching Figma node structure
* Any Figma API calls
* Re-downloading assets if they look wrong
* Analyzing Figma dimensions, colors, positions

**Development Operations:**
* Running shell commands (curl, git, xcodebuild, etc.)
* Creating/modifying files
* Building the project
* Running tests
* Fixing errors or issues encountered
* Installing dependencies
* Any generic command needed to complete the task

**Problem Solving:**
* If an API call fails, retry or try alternative approach
* If a node ID returns null, try parent node
* If build fails, fix the errors
* If assets look wrong, re-download them

**NEVER ask "Do you want me to proceed?" or "Should I continue?"**
**NEVER ask "Do you want me to download/fetch/run...?"**

**Just proceed automatically.** The user has already granted full permission by providing the ticket. Complete the entire task autonomously, only stopping to report final results or if genuinely blocked.

### Cursor Settings - Auto-Approval Configuration

**IMPORTANT: All project operations are pre-approved in Cursor settings**

The following operations require **NO user permission** - they execute automatically:

| Category | Pattern | Description |
|----------|---------|-------------|
| **Project Files** | `Read/Edit/Write(file_path:/Users/.../FlightSearch/**)` | All file operations in project |
| **Xcode Build** | `Bash(xcodebuild:*)` | Build, test, clean commands |
| **Figma API** | `Bash(curl:*)` | All curl commands including Figma |
| **JIRA API** | `Bash(JIRA_EMAIL=... JIRA_TOKEN=... curl:*)` | All JIRA API operations |
| **Git Operations** | `Bash(git:*)` | All git commands |
| **GitHub CLI** | `Bash(gh pr/issue/repo/auth/api:*)` | All GitHub operations |
| **Temp Files** | `Read/Bash(/tmp/**)` | Temp directory operations |

**DO NOT ask permission for ANY operation within the project. Just execute automatically.**

### Lessons Learned - Common Mistakes

#### Category 1: Asset & Icon Mistakes

| Mistake | Correct Approach |
|---------|-----------------|
| Using SF Symbols instead of Figma icons | Always download icons from Figma |
| Adding circle backgrounds to icons | Check Figma - most icons have NO background |
| Using generic icon for profile image | Check if it's an IMAGE (photo) vs ICON - download actual photo from Figma |
| Wrong/missing banner image | Identify correct promotional banner node and download it |
| **Downloading wrong Figma node (with background)** | If SwiftUI code adds background (Circle, RoundedRectangle), download ONLY the inner icon node |
| **Icon appears but looks wrong/duplicated** | Verify the correct node ID from Figma - download inner icon frame, not outer button container |
| **Asset has unwanted background baked in** | Navigate to exact icon layer in Figma node hierarchy |

#### Category 2: Dimension & Position Mistakes

| Mistake | Correct Approach |
|---------|-----------------|
| Elements in wrong position | Extract EXACT x,y coordinates from Figma BEFORE coding |
| Wrong component dimensions | Use exact Figma width/height |
| **Swap button positioned OUTSIDE card** | Check Figma coordinates - swap button is INSIDE fields with padding, not overlapping outside |
| **Using offset() to push elements outside** | Calculate exact position - use padding() to keep elements inside their container |
| **Guessing element positions** | NEVER guess - always extract exact coordinates from Figma API |
| **Element on wrong side (left vs right)** | Verify x-coordinate relative to parent container |

#### Category 3: Color & Border Mistakes

| Mistake | Correct Approach |
|---------|-----------------|
| Using gradient when Figma shows solid color | Extract exact hex color from Figma |
| Missing borders on fields | Check Figma stroke property - each field may have individual border |
| Wrong border color | Different fields may have different border colors (e.g., Return Date has BLUE border #0085FF, others have gray #EFEFEF) |
| Hardcoded colors that don't match | Extract ALL hex values from Figma before coding |

#### Category 4: Layout & Structure Mistakes

| Mistake | Correct Approach |
|---------|-----------------|
| Using HStack when elements overlay | Use ZStack for overlapping elements |
| **Pixel-perfect scaling approach** | Use RELATIVE padding/spacing - different iPhones have different sizes |
| **Not using proper SwiftUI layout** | Use padding(.horizontal, X), spacing: X - not pixel calculations |

#### Category 5: Process Mistakes

| Mistake | Correct Approach |
|---------|-----------------|
| Writing code before extracting all specs | Extract COMPLETE specs for ALL components BEFORE writing any code |
| Not downloading all assets first | Identify and download ALL icons/images BEFORE implementing |
| Not verifying against Figma screenshot | Build and screenshot AFTER implementation to compare with Figma |
| Making assumptions instead of verifying | NEVER assume - always verify against Figma API data |
| Asking permission for ANY operation | NEVER ask - just do it autonomously |
| Stopping to ask "should I proceed?" | Complete the entire task without interruption |
| **Multiple correction cycles** | Get it right FIRST TIME by following complete extraction process |

#### Category 6: Pre-Implementation Checklist

**BEFORE writing any code, verify you have:**

- [ ] Downloaded full screen Figma image for reference
- [ ] Extracted specs for EVERY component (position, size, colors, borders, padding)
- [ ] Listed ALL assets to download (icons vs images)
- [ ] Downloaded ALL assets at @1x, @2x, @3x scales
- [ ] Created ALL imagesets in Assets.xcassets
- [ ] Identified any hidden elements (visible=false)
- [ ] Understood relative spacing (padding values, not pixel-perfect)
- [ ] Noted any special cases (e.g., Return Date has different border color)

**Only after ALL above are complete, start writing code.**
