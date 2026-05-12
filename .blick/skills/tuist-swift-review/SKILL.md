---
name: tuist-swift-review
description: Project-specific PR-review rules for the tuist/tuist Swift codebase (cli). Focuses on the things only this repo knows — preferring value types, the testing framework choice, and migration of XCTest to Swift Testing.
---

# Tuist Swift Review

This skill is intentionally narrow. **Generic Swift style, formatting,
naming, and lint hygiene are already covered by SwiftFormat / SwiftLint
in CI — do not flag those.** Focus on the rules below.

For each finding, cite `path:line` and quote the relevant snippet.

---

## 1. Prefer structs over classes

Default to `struct` for new types. Reach for `class` only when reference
semantics, identity, inheritance, or `deinit` are actually required.

### Flag

- **A new `class` declaration that has no stored mutable identity, no
  inheritance, no `deinit`, and is not bridged to an Objective-C / Cocoa
  API.** Recommend converting it to a `struct`. **Severity: medium.**

### Do not flag

- Existing classes left unchanged by the diff.
- Classes that subclass a framework type (`NSObject`, `XCTestCase`,
  `Operation`, etc.) or conform to a protocol that requires reference
  semantics.
- Types that genuinely need reference identity (caches, long-lived
  coordinators, actors-with-state-shared-by-reference).

---

## 2. Testing framework — Swift Testing, not XCTest

New tests must be written with **Swift Testing** (`import Testing`,
`@Test`, `#expect`, `#require`). XCTest is legacy in this repo.

When a diff **modifies** an existing XCTest case (i.e. the test file
already uses `XCTestCase` / `func testXxx()` and the PR changes one of
those tests), consider whether the test should be migrated. However,
**do not require migration when**:

- The test file contains more than 5 existing XCTest cases and migrating
  only the touched test would create paradigm mixing within the file.
- The change is incidental (e.g., updating an expected JSON string,
  changing a mock return value) rather than adding new test logic.

In these cases, leave a note rather than a blocking finding.

### Flag

- **A newly added test file or test function that uses `XCTestCase` /
  `XCTAssert*` / `func testXxx()`.** Recommend Swift Testing.
  **Severity: high.**
- **A modified XCTest test where the file has ≤5 test cases and the
  change is substantive (new assertions, modified test flow).**
  Ask the author to rewrite the touched test(s) using `@Test` / `#expect`
  / `#require`. **Severity: medium.**
- **Mixing `XCTAssert*` calls inside a Swift Testing `@Test`** (or vice
  versa). Pick one framework per test.

### Do not flag

- Pre-existing XCTest tests that the diff does not touch.
- XCTest-only APIs that have no Swift Testing equivalent yet (e.g.
  `XCUITest` UI automation) — leave those on XCTest.
- Test helpers/fixtures that aren't themselves test cases.
- Modified XCTest files with >5 tests where migration would mix paradigms.

When suggesting the migration, point at the project's Swift Testing
pattern (see `cli/AGENTS.md`): use `@Test(.inTemporaryDirectory)` and
`FileSystem.temporaryTestDirectory` for tests that need a temp dir, and
`#require` for unwrapping.
