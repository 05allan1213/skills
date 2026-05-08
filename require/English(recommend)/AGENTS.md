# AGENTS.md

Behavioral guidelines for AI coding agents working on Go projects — personal and work.

These rules focus on **how to implement**, not **what to implement**.

Core goals:

- Communicate clearly in Chinese
- Understand before coding
- Keep changes small and surgical
- Prefer simple Go code
- Avoid unnecessary dependencies
- Verify changes honestly
- Protect user work and Git history
- Use Codex Superpowers workflows when helpful

**Personal preferences are defaults, not overrides.** When working in an existing repository, follow the repository's current style, Go version, dependency policy, error-handling pattern, and deployment conventions first. If personal habits conflict with existing project conventions, the project wins.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

---

## 1. Communication

Default to Chinese when communicating with the user.

Use Chinese for requirement clarification, implementation plans, progress updates, test reports, risk explanations, and final summaries.

Do not force-translate common technical terms. Prefer natural Chinese technical writing: handler, service, middleware, mock, TDD, CI/CD, goimports, go test, Git commit.

Code, identifiers, file names, error messages, logs, comments, and commit messages must match the existing project style. Never add Chinese comments unless the codebase already uses them consistently. Never mix English and Chinese comments in the same file.

---

## 2. Understand Before Coding

Do not start coding before understanding the task and the existing code.

Before implementing, clarify:

- What is the real goal?
- Where is the relevant logic?
- What is the smallest safe change?
- What should not be changed?
- Are there API, config, schema, or deployment impacts?

If something is unclear, stop and ask. If multiple interpretations exist, present them. If a simpler approach exists, say so. Push back when the requested solution is unnecessarily complex or risky.

**For work projects especially:** read the relevant code first. Do not reshape the project style based on personal preferences.

---

## 3. Simplicity First

Use the minimum code needed to solve the problem.

Avoid speculative features, premature abstractions, single-use interfaces, unrequested configurability, large rewrites, and excessive boilerplate.

Ask yourself: *Would a senior Go engineer consider this overcomplicated?* If yes, simplify.

---

## 4. Surgical Changes Only

Touch only what is necessary for the current task. Every changed line should trace directly to the user's request.

Do not refactor unrelated code, rename unrelated symbols, move files, or reformat unrelated files.

The following must not be changed without explicit reason and prior explanation of impact, scope, compatibility, and verification plan:

- Go version in `go.mod`
- Dockerfile base image versions
- Makefile targets
- CI workflow files
- `.gitignore`
- Public API response format
- Config key names
- Database schema
- Prometheus metric names
- Helm values structure
- Kubernetes resource names

If you notice unrelated problems, mention them separately. Do not fix them silently.

---

## 5. Superpowers Workflow

When Superpowers skills are available, use them based on task complexity. Do not mechanically run every workflow for tiny changes.

For non-trivial implementation tasks, explicitly state which Superpowers workflow is being used before starting.

Before coding, the agent should say whether it is using one of the following workflows:

- `brainstorming`
- `writing-plans`
- `executing-plans`
- `test-driven-development`
- `requesting-code-review`
- `finishing-a-development-branch`

If no Superpowers workflow is used, briefly explain why the task is small enough to proceed directly.

For work projects: any medium-to-large requirement, risky change, or task touching API / config / data structure / deployment **must** be planned before execution.

### 5.1 Brainstorming

Use `brainstorming` for unclear, large, or risky requirements. Do not start coding while the requirement is still ambiguous. For small obvious tasks, a short clarification is enough.

### 5.2 Writing Plans

Use `writing-plans` for non-trivial implementation work. Keep plans practical.

```text
目标：
- xxx

范围：
- xxx

步骤：
1. xxx
2. xxx
3. xxx

验证：
- xxx

风险：
- xxx
```

### 5.3 Executing Plans

Use `executing-plans` when a plan exists. Follow the approved plan, make one meaningful change at a time, do not silently expand scope, stop if the plan becomes invalid. If a new issue is discovered, explain it before changing direction.

### 5.4 Test-Driven Development

Use `test-driven-development` when useful. Prefer TDD for bug fixes, core business logic, validation logic, state transitions, parsing/formatting, Redis key generation, PromQL generation, and error handling behavior.

Preferred flow: write a failing test → implement the smallest fix → run the test → refactor only if needed.

Do not use TDD for documentation-only, comment-only, trivial wiring, or formatting-only changes.

### 5.5 Requesting Code Review

Use `requesting-code-review` for larger or riskier changes. Review for scope control, correctness, error handling, test coverage, backward compatibility, unexpected API/config/schema changes, unnecessary dependencies, and hidden deployment impact. Fix issues before final reporting.

### 5.6 Finishing a Development Branch

Use `finishing-a-development-branch` before considering a branch done. Summarize what changed, what was verified, what was not verified, known risks, suggested commit message, and suggested next step. Do not run Git commit, push, reset, clean, or checkout without explicit user approval.

---

## 6. Goal-Driven Execution

Convert tasks into verifiable goals.

```text
"Add validation"   → Add tests for invalid inputs, then make them pass.
"Fix the bug"      → Reproduce the bug with a test, then fix it.
"Refactor X"       → Ensure behavior is unchanged before and after.
```

For multi-step tasks, use a brief plan:

```text
1. [Step] -> verify: [check]
2. [Step] -> verify: [check]
```

Weak criteria such as "make it work" require clarification.

---

## 7. Go Coding Rules

Write idiomatic, simple Go. Follow existing project patterns first.

Avoid silent error drops, debug prints, and unnecessary panics:

```go
_ = err              // avoid
panic(err)           // avoid in business logic
fmt.Println("debug") // avoid
```

Do not put too much business logic in handlers. Move to service, client, or helper packages when needed — but do not over-layer simple code.

### context.Context

- External calls (HTTP, DB, Redis, etc.) must be bounded by context cancellation, deadline, or timeout. Do not use `context.Background()` directly for external calls when a request or operation context is available.
- Never store `context.Context` in a struct. Always pass it as the first parameter.
- Never pass nil context. Prefer an existing request or operation context. Use `context.TODO()` only as a temporary placeholder when no proper context is available, and do not use it to hide missing context design.

### Interfaces

- Define interfaces in the package that uses them, not the package that implements them.
- Keep interfaces small (1–2 methods).
- Do not export interfaces unless they are part of the public API.
- Only introduce interfaces when they improve testability or decoupling.

### Error Handling

Use standard Go error handling. `err == nil` and `err != nil` checks are normal and expected.

Add context when wrapping errors:

```go
if err != nil {
    return fmt.Errorf("load config: %w", err)
}
```

Use `errors.Is()` / `errors.As()` when:
- The project already uses sentinel errors or wrapped error matching.
- The current task genuinely needs to distinguish specific error types.
- A standard library or dependency returns errors that require this approach.

Do not introduce a complex error hierarchy unless the project already has one or the task explicitly requires it.

Other rules:
- Do not wrap the same error multiple times in the same call chain.
- Do not log and return the same error. Log only at the top level (handlers, main). Lower levels wrap and return.
- Do not use `github.com/pkg/errors`. Use standard library `errors` and `fmt.Errorf("%w")`.
- Do not panic in business logic. Panics are only for unrecoverable initialization errors in `main`.
- Do not leak sensitive internal errors to external API responses.

### defer

Always place defer after error checking:

```go
// ❌ Wrong — panics if err != nil
f, err := os.Open("file")
defer f.Close()
if err != nil { return err }

// ✅ Correct
f, err := os.Open("file")
if err != nil { return err }
defer f.Close()
```

Avoid defer inside long-running or resource-heavy loops. If each iteration must release resources promptly, use an inner function or explicit cleanup instead. Short loops with trivial operations are acceptable.

For critical cleanup such as file `Close()` or DB `Commit()`/`Rollback()`, handle cleanup errors explicitly when they matter. If using named returns, make sure the pattern matches the existing project style. Do not introduce named returns only to handle a defer error unless it improves clarity.

### Goroutines

- Use context cancellation to signal shutdown. Every goroutine should have a clear exit condition.
- Do not drop errors silently in goroutines.
- Avoid goroutine leaks.
- Do not use `time.Sleep()` for synchronization.
- When goroutines need lifecycle coordination, use `sync.WaitGroup`, channels, `errgroup`, or context-based lifecycle management. For long-running services, the priority is cancellability, clean exit, and observability — not necessarily waiting for completion.
- Add panic recovery only for long-running background tasks (worker, scheduler, consumer, watcher, message loop). Do not add recover everywhere — it hides bugs in ordinary business goroutines.

```go
// For long-running background tasks only
go func() {
    defer func() {
        if r := recover(); r != nil {
            log.Error("worker recovered from panic", "error", r)
        }
    }()
    // ...
}()
```

**Loop variable capture:** Check `go.mod` first. In Go 1.22+, loop variables are scoped per iteration — `item := item` is unnecessary. For Go < 1.22, or projects that consistently use loop variable copies, follow the existing style. For modern Go projects, do not add redundant `item := item` unless it improves clarity or avoids mutation/reference confusion.

### Other Rules

- Prefer explicit `NewXxx()` or `SetupXxx()` constructors. Avoid `init()` for application initialization. `init()` is acceptable when the project already uses it, a registration mechanism genuinely requires it, or test/tool code has a consistent existing pattern.
- Avoid global variables (except constants).
- Use the two-value form for map access only when you need to distinguish "key absent" from zero value. Direct access is fine when zero value is a valid default. For type assertions, prefer the two-value form to avoid panics.

```go
v, ok := m["key"]   // use when absence matters
v := m["key"]       // fine when zero value is a valid default

v, ok := i.(string) // prefer to avoid panic
```

- Prefer `time.Equal()` for semantic time comparison. Use `==` only when comparing the full struct value is intentional.
- Do not modify function arguments of reference types (slices, maps, pointers) unless explicitly documented.
- Prefer `strings.Builder` for repeated string concatenation in loops, especially when the loop may be large or performance matters.

---

## 8. Testing and Verification

### Writing Tests

Add or update tests when changing core logic. Prioritize: pure functions, request validation, config loading, parsing, response construction, error handling, key generation, cache logic, state transitions.

Prefer unit tests and table-driven tests with `t.Run()`:

```go
for _, tc := range tests {
    t.Run(tc.name, func(t *testing.T) {
        // ...
    })
}
```

Only copy `tc` (`tc := tc`) when required by the project's Go version or existing style.

Unit tests must not depend on real Redis, MySQL, Prometheus, Kubernetes, public internet, or fixed ports. Use interfaces, mocks, `httptest`, temp directories, or in-memory implementations.

Do not use `time.Sleep()` for synchronization in tests. Use channels, `sync.WaitGroup`, or fake clocks.

Do not delete tests to make the suite pass. Do not comment out core logic to make code compile.

### Running Verification

Every meaningful change must be verified. Default for Go changes:

```bash
goimports -w <changed-files>
go test ./<changed-package>
```

When appropriate:

```bash
go test ./...
go vet ./...
```

For infrastructure changes: `docker build`, `helm lint`, `kubectl apply --dry-run=client`, `promtool check`.

If the repository has an existing Makefile, task runner, CI script, or documented test command, prefer the project's existing verification command when appropriate (e.g., `make test`, `make lint`, `task test`).

### Reporting Results Honestly

Report with clear status:

```text
Passed:   xxx
Failed:   xxx
Not run:  xxx, because xxx
```

Never claim success without running the command. Do not use: "Should be fine", "Looks good", "Probably works", "Not tested but should pass".

---

## 9. Dependencies

Do not add new dependencies unless necessary. Before adding one, explain purpose, why the standard library is insufficient, why existing dependencies are insufficient, and the impact on `go.mod`, binary size, security, and deployment.

Do not run `go get -u`, `go get <pkg>@latest`, or `go mod tidy` without an explicit reason. Explain before running `go mod tidy`. Never modify the Go version in `go.mod` unless explicitly requested.

---

## 10. Configuration and Secrets

Do not hardcode IP addresses, ports, domains, credentials, tokens, namespaces, or service addresses. Use environment variables or existing secret management.

When adding config, document: name, default value, purpose, required/optional, sensitive/not, and usage per environment (local / Docker / Kubernetes).

Do not put secrets in code, README examples, plain config files, test fixtures, or logs.

---

## 11. Git Safety

Before modifying files, check `git status`. If there are uncommitted user changes, do not overwrite them. Mention the risk and ask first.

Never run without explicit user approval:

```bash
git commit / git push / git reset --hard / git checkout . / git clean -fd
```

Suggest commit commands; do not run them:

```bash
git add <files>
git commit -m "fix: validate request parameter"
```

Use English conventional commit types matching the project's existing style: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `ci`, `style`.

---

## 12. Reporting

After each meaningful module, stop and report:

```text
完成：
- xxx

实际修改：
- xxx

修改文件：
- xxx

已验证：
- xxx：通过/失败

未验证：
- xxx，原因：xxx

风险：
- xxx

建议提交：
git add xxx
git commit -m "xxx"
```

Do not continue to the next module without user confirmation when the task was split into modules, the next step changes scope or requires new dependencies, the next step touches API/config/schema/deployment, tests failed, or existing user changes may be affected.

For tiny changes, keep the report short but still state what changed and what was verified.

---

## 13. Stop Conditions

Stop and explain before proceeding if:

- Requirements are unclear
- The change is larger than expected
- A new dependency is needed
- Public API, config format, database schema, or Kubernetes/Helm behavior must change
- Tests cannot be run or tests fail
- User has uncommitted changes in affected files
- Backward compatibility is at risk

Do not hide uncertainty. Do not silently make risky decisions.

---

## 14. Forbidden Behaviors

Unless explicitly requested, do not:

- Rewrite the whole project or perform broad unrelated refactors
- Rename or move unrelated symbols or files
- Upgrade many dependencies at once
- Change API response structures, config field names, or public function signatures
- Change Prometheus metric names, Helm values structure, or Kubernetes resource names
- Delete existing tests or user code
- Leave debug code, fake TODO implementations, or pseudocode as completed work
- Claim tests passed without running them
- Use complex architecture to hide simple logic
- Commit, push, reset, clean, or checkout without approval
- Place defer before error checking
- Store context in structs
- Use `github.com/pkg/errors`
- Panic in business logic
- Use `time.Sleep()` for synchronization

---

## 15. Final Principle

Help the developer move safely and steadily.

```text
Understand first → Clarify when needed → Keep scope small
Plan for non-trivial work → Implement surgically → Test honestly
Report clearly → Protect Git history → Wait for confirmation when scope changes
```

Good work: small diffs, clear reasoning, few surprises, tests that actually ran, no unnecessary dependencies, no hidden changes, no overwritten user work.
