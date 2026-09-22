# Issue Categories

A taxonomy of workflow issues with detection patterns and severity levels.

## Category 1: Dead Ends

**Severity:** 🔴 CRITICAL

Entry point leads nowhere or to wrong destination. User cannot complete their goal.

### Detection Patterns

```swift
// 1. Navigation to section that doesn't contain the feature
selectedSection = .tools  // but feature isn't in ToolsView

// 2. Sheet case exists but handler missing
enum SheetType {
    case orphanedFeature  // in enum...
}
// but sheetContent(for:) has no case for .orphanedFeature

// 3. View exists but no entry point
struct FeatureView: View { ... }  // never instantiated
```

### Automated Detection

```bash
# Find enum cases
grep "case " Models/SheetType.swift | wc -l

# Find switch cases in handler
grep "case \." Views/SheetContent.swift | wc -l

# If counts differ, investigate
```

### User Impact

- Complete goal failure
- Distrust of UI elements
- App abandonment

---

## Category 2: Wrong Destination

**Severity:** 🔴 CRITICAL

Entry point works but leads to unexpected place.

### Detection Patterns

```swift
// Card says "Track Warranty" but opens generic settings
FeatureCard(
    title: "Track Warranty",
    action: { selectedSection = .settings }  // Wrong!
)
```

### Automated Detection

Manual review required. Compare:
1. Card/button title
2. Destination content

### User Impact

- Confusion
- Lost trust in navigation
- Support requests

---

## Category 3: Incomplete Navigation

**Severity:** 🟠 HIGH

User lands on correct section but must find feature manually.

### Detection Patterns

```swift
// Navigates to section top, not specific feature
action: { selectedSection = .tools }
// User must scroll through Tools to find PriceWatch

// Should be:
action: { activeSheet = .priceWatch }
// Or: selectedSection = .tools; scrollTarget = .priceWatch
```

### Detection Heuristic

Flag any entry point where:
- Action uses `selectedSection = .xxx`
- Entry point label implies specific feature (not section)

### User Impact

- Extra steps (scroll, search)
- Mild frustration
- May not find feature

---

## Category 4: Missing Auto-Activation

**Severity:** 🟠 HIGH

Feature requires mode/state that isn't set when navigating.

### Detection Patterns

```swift
// "Bulk Edit" card navigates but doesn't enable selection
action: { selectedSection = .myItems }
// User sees normal list, must find Select button

// Should be:
action: {
    selectedSection = .myItems
    isSelectMode = true
}
```

### Detection Heuristic

Look for entry points with:
- Mode-related labels ("Bulk", "Select", "Edit Multiple")
- No accompanying state change

### User Impact

- "Where is the feature?"
- Extra button taps to activate
- May think card is broken

---

## Category 5: Two-Step Flow (Friction)

**Severity:** 🟡 MEDIUM

User must make intermediate selection before reaching feature.

### When Acceptable

```swift
// Feature requires item context
case .aiAssistant:
    ItemPickerSheet(
        onSelect: { item in showAI(for: item) }
    )
// This is NECESSARY - AI needs an item to analyze
```

### When Problematic

```swift
// Picker shown when only 1 eligible item
if eligibleItems.count >= 1 {  // Should check == 1
    showItemPicker(items: eligibleItems)
}
```

### Optimization

```swift
// Skip picker if only one item
if eligibleItems.count == 1 {
    showFeature(for: eligibleItems[0])
} else {
    showItemPicker(items: eligibleItems)
}
```

### User Impact

- Extra tap
- Acceptable if necessary
- Annoying if avoidable

---

## Category 6: Missing Feedback

**Severity:** 🟡 MEDIUM

Action completes without user confirmation.

### Detection Patterns

```swift
// Save without toast
func save() {
    try modelContext.save()
    dismiss()  // User doesn't know if it saved
}

// Should be:
func save() {
    try modelContext.save()
    ToastManager.shared.success("Saved")
    dismiss()
}
```

### Detection Heuristic

Search for:
- `modelContext.save()` without toast/banner
- `modelContext.delete()` without confirmation
- `dismiss()` after data changes

### User Impact

- Uncertainty ("did it work?")
- May retry, creating duplicates
- Anxiety about data safety

---

## Category 7: Inconsistent Patterns

**Severity:** 🟢 LOW

Same feature accessed differently in different places.

### Detection Patterns

```swift
// Location A: Sheet
Button("Export") { activeSheet = .export }

// Location B: Navigation
Button("Export") { selectedSection = .export }

// Location C: Direct action
Button("Export") { exportData() }
```

### Detection Heuristic

Group entry points by destination, compare action types.

### User Impact

- Mental model confusion
- Maintenance burden
- Inconsistent UX

---

## Category 8: Orphaned Features

**Severity:** 🟡 MEDIUM

Views/features that exist but have no entry point.

### Detection Patterns

```swift
// View defined
struct CoolFeatureView: View { ... }

// But never used except in previews
#Preview { CoolFeatureView() }
```

### Automated Detection

```bash
# Find all View structs
grep -r "struct.*: View" Sources/ --include="*.swift" -l

# For each, check if instantiated outside previews
for file in $(grep -r "struct.*: View" Sources/ -l); do
  views=$(grep "struct.*: View" "$file" | sed 's/.*struct \([A-Za-z]*\):.*/\1/')
  for view in $views; do
    count=$(grep -r "$view(" Sources/ --include="*.swift" | grep -v "#Preview" | wc -l)
    if [ "$count" -eq 0 ]; then
      echo "Orphaned: $view in $file"
    fi
  done
done
```

### User Impact

- Wasted development effort
- Dead code
- Or: missing feature discovery!

---

## Category 9: Mock Data

**Severity:** 🔴 CRITICAL

Feature displays fabricated or hardcoded data when real user data exists in the model.

### Detection Patterns

```swift
// 1. Fake async delay simulating a real fetch
DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
    self.repairInfo = RepairContext(
        repairabilityScore: 7,           // ❌ Hardcoded
        estimatedRepairCost: "$85-150",   // ❌ Made up
        commonFailurePoints: ["Battery"]  // ❌ Not from user data
    )
}

// 2. Static data where model relationship has real values
let alternatives = [
    AIAlternative(name: "Product X", price: "$299")  // ❌ Fabricated
]
// But item.rmaRecords has real repair costs!

// 3. Simplified decision logic ignoring available data
if item.assetAge > lifespan * 0.8 {
    return .replace  // ❌ Ignores repair costs, warranty, ratings
}
```

### Automated Detection

```bash
# Find asyncAfter with hardcoded data (mock fetches)
grep -rn "asyncAfter" Sources/ --include="*.swift" -A 5

# Find hardcoded numeric values in view/viewmodel logic
grep -rn "Score.*=.*[0-9]" Sources/ --include="*.swift" | grep -v "test\|Test\|preview\|Preview"

# Find placeholder strings
grep -rn '"\\$[0-9]' Sources/ --include="*.swift" | grep -v "test\|Test\|format"
```

### User Impact

- User sees fake data and makes real decisions based on it
- Erodes trust when numbers don't match their experience
- Feature appears functional but provides no actual value

### Design Principle Violated

**Data Integrity** — If the app tracks data relevant to a feature, the feature must use it.

---

## Category 10: Unwired Data

**Severity:** 🟠 HIGH

Model properties or relationships exist that would improve a feature, but the feature doesn't read them.

### Detection Patterns

```swift
// Model tracks repair costs...
class Item {
    var rmaRecords: [RMARecord]?      // Has repairCostInCents
    var maintenanceRecords: [MaintenanceRecord]?  // Has costInCents
    var currentMarketPriceInCents: Int?  // From Price Watch
    var warrantyDeductibleInCents: Int?  // User-entered
}

// ...but decision engine only checks age and rating
func computePath() -> ItemDecisionPath {
    if item.assetAge > lifespan { return .replace }  // ❌ Ignores all cost data
    if item.userRating < 3 { return .replace }
    return .keep
}
```

### Automated Detection

```bash
# Find model properties
grep -rn "var.*InCents\|var.*Cost\|var.*Price\|var.*Records" Sources/Models/ --include="*.swift"

# Find feature views that should use this data
grep -rn "class.*ViewModel\|func compute\|func calculate\|func fetch" Sources/Features/ --include="*.swift"

# Cross-reference: which properties are never read by which features?
```

### Cross-Reference Checklist

For each feature, ask:
1. What model properties exist that are relevant?
2. Which of those does the feature actually query?
3. What's the gap?

| Feature | Relevant Model Data | Actually Used | Gap |
|---------|-------------------|---------------|-----|
| Decision Engine | repair costs, warranty, ratings, condition, brand | age, rating | repair costs, warranty, condition, brand |
| Cost Comparison | repair costs, replacement price, market price | none | all |

### User Impact

- Feature works but gives shallow/generic advice
- User has entered data that the app ignores
- Missed opportunity to provide personalized, useful analysis

---

## Category 11: Platform Parity Gap

**Severity:** 🟠 HIGH

Feature works on one platform (iOS/macOS) but is broken or missing on the other.

### Detection Patterns

```swift
// 1. iOS-only dismiss button
#if os(iOS)
ToolbarItem(placement: .cancellationAction) {
    Button("Done") { dismiss() }  // ❌ macOS users trapped
}
#endif

// 2. Extension can't resolve properties cross-platform
// In DashboardView.swift:
var filteredItems: [Item] { dashboardVM.filteredItems }  // Internal computed
// In DashboardView+Extension.swift:
AppleCareSheet(items: filteredItems)  // ❌ "cannot find in scope" on macOS

// 3. Platform-specific type without cross-platform wrapper
let image = UIImage(data: imageData)  // ❌ Crashes on macOS
```

### Automated Detection

```bash
# Find iOS-only dismiss buttons (common bug pattern)
grep -rn "#if os(iOS)" Sources/ --include="*.swift" -A 3 | grep -i "dismiss\|done\|close\|cancel"

# Find extension files that might have cross-platform issues
grep -rl "extension.*View" Sources/ --include="*.swift" | xargs grep -l "dashboardVM\|viewModel\|private"

# Find platform-specific types without wrappers
grep -rn "UIImage\|UIColor\|NSImage\|NSColor" Sources/ --include="*.swift" | grep -v "#if\|#else"
```

### Verification Process

After ANY code change, build BOTH platforms:
```bash
build_sim      # iOS Simulator
build_macos    # macOS
```

### User Impact

- Feature completely broken on one platform
- macOS users trapped in sheets they can't dismiss
- Compile errors block entire build

---

## Severity Summary

| Severity | Categories | Action |
|----------|------------|--------|
| 🔴 Critical  | Dead End, Wrong Destination, Mock Data,              | Fix immediately        |
|              | Destructive No Confirm, Silent State Reset            |                        |
| 🟠 High      | Incomplete Nav, Missing Auto-Activation,              | Fix before release     |
|              | Unwired Data, Platform Parity, Promise-Scope,         |                        |
|              | Buried Action, Dismiss Trap, Context Dropping,        |                        |
|              | Notif Nav Fragility, Sheet Asymmetry,                 |                        |
|              | Empty State Missing, Error Recovery Missing,          |                        |
|              | Keyboard Obscures, Permission Dead End,               |                        |
|              | Modal Stacking, Nav Container Mismatch                |                        |
| 🟡 Medium    | Two-Step Flow, Missing Feedback, Gesture-Only,        | Fix if time permits    |
|              | Loading State Trap, Stale Nav Context,                |                        |
|              | Phantom Touch Target, Race Condition UX,              |                        |
|              | Invisible Selection, Orphaned Features                |                        |
| 🟢 Low       | Inconsistent Patterns, Double-Nested Nav              | Note for future        |

## Categories 12-32 (added v2.5.0)

The following categories were added to align with ui-path-radar's detection coverage. See `agents/layer3-issue-detection.md` Categories 21-32 for full detection patterns.

| #    | Category                    | Sev          | Description                              |
|------|-----------------------------|--------------|------------------------------------------|
| 21   | Destructive No Confirm      | 🔴 CRITICAL | Delete/clear with no dialog              |
| 22   | Silent State Reset          | 🔴 CRITICAL | Work lost on navigate away               |
| 23   | Empty State Missing         | 🟡 HIGH     | Blank screen when list empty             |
| 24   | Error Recovery Missing      | 🟡 HIGH     | Error shown, no retry path               |
| 25   | Keyboard Obscures           | 🟡 HIGH     | TextField covered by keyboard            |
| 26   | Permission Dead End         | 🟡 HIGH     | Denied, no path to Settings              |
| 27   | Modal Stacking              | 🟡 HIGH     | Sheets/alerts pile up                    |
| 28   | Nav Container Mismatch      | 🟡 HIGH     | Invalid tag for container                |
| 29   | Phantom Touch Target        | 🟢 MEDIUM   | Looks tappable but isn't                 |
| 30   | Race Condition UX           | 🟢 MEDIUM   | Conflicting ops triggered                |
| 31   | Invisible Selection         | 🟢 MEDIUM   | Selected but no indicator                |
| 32   | Double-Nested Nav           | ⚪ LOW      | NavStack inside NavStack                 |
| 33   | Invisible Control           | 🟡 HIGH     | Wired + placed, but no visible boundary  |

## Issue Documentation Template

When documenting an issue, include:

```yaml
issue:
  id: "issue-001"
  category: "dead_end"
  severity: "critical"

  entry_point:
    label: "Repair Advisor"
    file: "PromotionCards.swift"
    line: 110
    action: "selectedSection = .tools"

  expected: "Repair Advisor feature opens"
  actual: "Tools section (feature not there)"

  user_impact: "Cannot get repair guidance despite card promise"

  recommendation: "Create item picker sheet, then open RepairAdvisor"
  effort: "medium"

  similar_issues: ["issue-003"]  # Related patterns
```
