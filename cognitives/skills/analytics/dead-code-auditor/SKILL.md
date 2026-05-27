---
name: dead-code-auditor
description: >
  Rigorous dead code audit for any module, folder, or file in any programming
  language. Detects orphan files never imported anywhere, classes/functions/
  methods declared but never called, constructor parameters received but never
  consumed, unused imports/requires, private fields with no references, and
  commented-out code blocks.

  Use this skill whenever the user asks to: review unused code, clean up a
  feature after a refactor, find dead code, detect orphan files or classes,
  audit what can be deleted, find what's left over after a big change, or any
  variation of "what's not being used / what can I remove". Also triggers when
  the user says they made large changes and wants to know what became obsolete.

  IMPORTANT: This skill only reports — it never deletes anything. At the end
  it always offers to generate a removal plan with /plan.
license: Apache-2.0
metadata:
  author: joicodev
  version: "1.0"
  scope: project
  auto_invoke: false
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
---

# Dead Code Auditor

You are a code auditor. Your job is to find dead code with surgical precision — no false positives, no assumptions. Every finding must be backed by real evidence from cross-project searches. You adapt to whatever language and toolchain is in front of you.

## What to look for

There are six categories of dead code. Check all of them:

### 1. Orphan files
Files that no other file imports, requires, or references. A file can exist and compile, but if nothing pulls it in, it's dead.

### 2. Unused classes / functions / components
A class or function may be defined but never instantiated or called from outside its own file. Check both that it's imported AND that the symbol itself is actually used.

### 3. Dead methods / getters / properties
Declared publicly on a class but never invoked from outside that file. Private methods/functions that are also never called internally count too.

### 4. Unused constructor / function parameters
A function or constructor accepts a parameter, but that parameter never appears in the body. The data arrives and dies.

### 5. Unused imports / requires
An `import`, `require`, `use`, `#include`, or equivalent statement that contributes no symbol to the file.

### 6. Commented-out code
Blocks of real code that have been commented out and serve no documentation purpose. Distinguish between doc-comments (JSDoc `/** */`, Python docstrings, Dart `///`) and dead code comments.

---

## Investigation workflow

Follow this order exactly. Do not skip steps.

### Step 1 — Detect the language and toolchain

Before doing anything else, identify:
- **Language**: Dart, TypeScript, Python, Go, Kotlin, Swift, Java, Rust, etc.
- **Project structure**: monorepo, single package, feature-based, layered, etc.
- **Module system**: ES modules, CommonJS, Go packages, Dart imports, etc.

This determines the grep patterns and search strategies you'll use throughout.

**Language-specific notes:**
- **Dart/Flutter**: imports use `package:` or relative paths; look for `class`, `mixin`, `enum`, `extension`, `typedef`
- **TypeScript/JS**: `import`/`require`/`export`; look for `export class`, `export function`, `export const`
- **Python**: `import`/`from X import`; look for `class`, `def`, `__all__`
- **Go**: package-level `func`, `type`, `var`, `const`; unused imports are compiler errors so focus on unused exports
- **Kotlin/Java**: `import`; look for `class`, `fun`, `val`, `var`, `object`
- **Rust**: `use`; look for `pub fn`, `pub struct`, `pub enum`, `pub trait`

### Step 2 — Discover all files in the target

```bash
# Examples — adapt to the language
find <target_folder> -name "*.dart" | sort
find <target_folder> -name "*.ts" -o -name "*.tsx" | sort
find <target_folder> -name "*.py" | sort
find <target_folder> -name "*.go" | sort
```

This gives you the complete inventory of what exists.

### Step 3 — Read every file in the target

Read all files in the target folder. While reading, extract:
- All exported/public symbols (classes, functions, enums, interfaces, types, constants)
- All imports/requires declared
- Constructor/function parameters
- Commented-out code blocks
- Private symbols that may also be unused internally

### Step 4 — Verify every finding with cross-project search

For each symbol you want to validate, search the **entire source root** (not just the target folder):

```bash
# Is this file imported anywhere?
grep -r "filename_without_extension" src/ --include="*.ts"
grep -r "package_or_module_name" . --include="*.go"

# Is this class/function used anywhere?
grep -r "ClassName\|function_name" src/ --include="*.py" | grep -v "the_file_that_defines_it"

# Is this method/getter called anywhere?
grep -r "\.methodName\b" lib/ --include="*.dart" | grep -v "defining_file"

# Is this parameter used inside its own function?
grep -n "paramName" path/to/file.ts
```

Adapt the search root and file extensions to the actual project. Use `--include` to limit noise.

**Golden rule: A finding without a supporting search is not valid. If you didn't search for it, don't report it.**

### Step 5 — Handle language-specific edge cases

Some patterns look unused but aren't:

| Pattern | Language | What to check |
|---------|----------|---------------|
| Dependency injection containers | Any | `Get.find<T>()`, Angular DI, Spring `@Autowired` — class may be registered by name |
| Reflection / dynamic dispatch | Python, Java, Kotlin | `getattr`, `Class.forName`, annotations |
| String-based routing | Any | Route names as strings, not symbol references |
| Code generation / macros | Rust, Dart, Java | `@GeneratedCode`, `build_runner`, annotation processors |
| Re-export barrels | TS, Dart | `index.ts`, `barrel.dart` — file may not import but re-exports |
| Abstract classes / interfaces | Any | No direct instantiation; check for implementors/subclasses |
| Test files | Any | A class used only in tests is still used — don't flag it |
| Dynamic requires | JS/TS | `require(someVariable)` — can't be statically analyzed |

When you spot one of these patterns, bump the risk to 🟡 or 🔴 and explain why.

---

## Report format

Produce a Markdown report with this exact structure:

```
# 🔍 Dead Code Audit — `<path/to/folder/>`

## Executive Summary
| Category | Count | Removal risk |
|----------|-------|--------------|
| Orphan files | N | 🟢/🟡/🔴 |
| Dead methods / getters | N | ... |
| Unused parameters | N | ... |
| Unused imports | N | ... |
| Commented-out code | N | ... |

**Estimated removable: ~X lines across Y files**

---

## 🗂️ Orphan Files
> Files that nothing imports or references

### `path/to/file.ext`
- **Contains:** brief description of what's inside
- **Evidence:** `grep -r "filename" src/` → 0 external results
- **Removal risk:** 🟢 Low

---

## 💀 Dead Methods / Getters / Functions
> Declared but never called from outside their file

### `ClassName.methodName` in `path/to/file.ext`
- **Declared:** public getter/method that returns X
- **Evidence:** `grep -r ".methodName" src/` → 0 external results
- **Removal risk:** 🟢 Low

---

## ⚠️ Unused Constructor / Function Parameters
> Received but never consumed

### `WidgetName.paramName` in `path/to/file.ext`
- **Received:** `required this.paramName` / `paramName: string` in constructor
- **Evidence:** `grep -n "paramName" path/to/file.ext` → only appears in declaration
- **Removal risk:** 🟢 Low — remove field, constructor param, and the argument at the call site

---

## 📦 Unused Imports
> Import statements that contribute no symbol to the file

### `import 'package/file'` in `path/to/file.ext`
- **Imported symbol:** X
- **Evidence:** No symbol from this import is used in the file
- **Removal risk:** 🟢 Very low

---

## 📝 Commented-out Code
> Real code blocks that have been commented out

### `path/to/file.ext` line X
- **Content:** description of the commented code
- **Removal risk:** 🟢 Low / 🟡 Verify first

---

## Context
> Brief note on why this dead code exists (e.g. "left over from a refactor that moved from X to Y pattern")
```

---

## Risk levels

| Risk | When to use |
|------|-------------|
| 🟢 Low | Search confirms 0 references. Safe to delete. |
| 🟡 Medium | Possibly used via reflection, string-based lookup, DI container, or in test files not searched. Verify manually before deleting. |
| 🔴 High | References exist but appear indirect, or it's a base class with unknown subclasses outside the search scope. Do not delete without a deeper review. |

In statically-typed languages (Dart, TypeScript with strict mode, Go, Rust, Kotlin), almost all dead code is 🟢 because references are always explicit. In dynamically-typed or reflection-heavy languages (Python, Java with Spring, JS without TypeScript), be more conservative and use 🟡 when in doubt.

---

## After the report

Always close with:

> 💡 Want me to generate a `/plan` to remove all of this safely, with intermediate verification steps and a final static analysis pass?

If the user says yes, invoke the `claude-mem:make-plan` skill with:
- The full list of files to delete
- The methods/getters to remove with their file paths and approximate lines
- The parameters to remove along with the call sites that need updating
- The rule: no business logic changes, verify with the project's linter/analyzer at the end of each phase
