# PRD: Simple CLI Calculator

## Introduction/Overview
Build a simple, fast command-line calculator that evaluates arithmetic expressions from arguments, stdin, or an interactive prompt. It should support common operators with standard precedence, provide clear error messages, and behave predictably across platforms.

## Goals
- Provide a zero-friction CLI for quick arithmetic from the terminal.
- Support basic operations with parentheses and unary minus.
- Offer multiple input modes: argument, stdin, and REPL.
- Return correct results with clear formatting and consistent exit codes.
- Be safe (no unsafe eval of user input) and cross-platform.

## User Stories

### US-001: Evaluate expression from CLI argument
**Description:** As a terminal user, I want to pass an expression as an argument and get the result so I can quickly compute values inline.

**Acceptance Criteria:**
- [ ] Running `calc "2 + 3 * 4"` prints `14` to stdout and exits `0`.
- [ ] Parentheses override precedence: `calc "(2 + 3) * 4"` -> `20`.
- [ ] Supports floats: `calc "7.5 / 2.5"` -> `3`.
- [ ] Unary minus works: `calc "-2 * (3 + 1)"` -> `-8`.
- [ ] Invalid expression prints error to stderr and exits non-zero.
- [ ] Output ends with a single newline, no extra whitespace.

### US-002: Evaluate expressions from stdin
**Description:** As a script author, I want to pipe expressions into the tool so it can compute results in a pipeline.

**Acceptance Criteria:**
- [ ] When no positional expression is provided, tool reads from stdin.
- [ ] Each non-empty line is treated as one expression; result per line.
- [ ] Example: `printf "1+1\n2*3\n" | calc` prints `2\n6\n`.
- [ ] Blank lines are ignored and produce no output lines.
- [ ] On any line error, print `line N: <message>` to stderr; continue to next line; exit non-zero if any line fails.

### US-003: Interactive REPL mode
**Description:** As a power user, I want an interactive prompt to try expressions iteratively.

**Acceptance Criteria:**
- [ ] `calc` with no args and no stdin opens a prompt like `calc> `.
- [ ] Entering an expression prints the result on the next line.
- [ ] Commands: `help`, `exit`/`quit`, and EOF (Ctrl-D) to leave with exit `0`.
- [ ] Input history (up/down) if the platform/implementation allows.
- [ ] Invalid input shows a concise error, keeps the session running.

### US-004: Basic operators and precedence
**Description:** As a user, I want common arithmetic with predictable precedence so results match standard math rules.

**Acceptance Criteria:**
- [ ] Supports: addition `+`, subtraction `-`, multiplication `*`, division `/`.
- [ ] Supports parentheses `(` `)` and unary minus.
- [ ] Operator precedence: parentheses > unary minus > `*` `/` > `+` `-`.
- [ ] Division by zero yields error: `calc "1/0"` -> stderr message, non-zero exit.
- [ ] Whitespace is optional anywhere between tokens.

### US-005: Help, version, and usage
**Description:** As a new user, I need discoverability so I can learn options quickly.

**Acceptance Criteria:**
- [ ] `calc --help` prints usage, options, examples; exits `0`.
- [ ] `calc --version` prints semantic version like `vX.Y.Z`; exits `0`.
- [ ] Usage shows examples for argument, stdin, and REPL modes.

### US-006: Precision and formatting
**Description:** As a user, I want control of numeric precision to avoid noisy floating-point output.

**Acceptance Criteria:**
- [ ] Optional `--precision <n>` limits decimal places in output (default sensible, e.g., `10`).
- [ ] Trailing zeros after decimal may be trimmed; `3.5000` -> `3.5`.
- [ ] Integer results print without decimal point (e.g., `6/2` -> `3`).
- [ ] Negative zero `-0` is normalized to `0`.
- [ ] Non-numeric results are never produced.

### US-007: Exit codes and errors
**Description:** As an automation engineer, I need reliable exit codes for scripting.

**Acceptance Criteria:**
- [ ] Success: `0`; parse/runtime error: non-zero (use `1`).
- [ ] Errors print concise messages to stderr; no stack traces by default.
- [ ] In stdin mode with mixed success/failure, exit non-zero if any line failed.
- [ ] In REPL, errors do not change the process exit code unless the session exits due to error.

## Functional Requirements
- FR-1: Support numeric literals with `.` decimal separator; scientific notation optional (TBD).
- FR-2: Implement `+ - * /`, parentheses, and unary minus with defined precedence and associativity.
- FR-3: Provide three input modes: argument expression, stdin (line-per-expression), and interactive REPL.
- FR-4: Print results to stdout; errors to stderr; always terminate lines with `\n`.
- FR-5: Provide `--help` and `--version`; show usage examples in `--help`.
- FR-6: Optional `--precision <n>` flag clamps decimal places; default value documented.
- FR-7: Return exit code `0` on success; `1` on any error.
- FR-8: Do not use unsafe language-level `eval` on user input; use a proper tokenizer/parser or a well-audited math expression library.
- FR-9: Handle division by zero and mismatched parentheses as errors with helpful messages.
- FR-10: In stdin mode, ignore empty/whitespace-only lines.

## Non-Goals (Out of Scope)
- NG-1: Advanced math functions (sin, cos, log), variables, or memory registers.
- NG-2: Units, currency formatting, localization of decimal separators.
- NG-3: Big-number arbitrary precision; default is standard double/float semantics.
- NG-4: Complex numbers, matrices, or vector operations.
- NG-5: Graphing or GUI.

## Design Considerations
- Keep CLI name short, e.g., `calc` (rename if collision; final name TBD).
- Consistent, minimal output suited for piping and scripting.
- Error messages should be human-readable and mention the token/position when feasible.
- REPL prompt `calc> `; print nothing extra on successful evaluation beyond the result.

## Technical Considerations
- Language/runtime is flexible; choose a cross-platform option. Avoid unsafe `eval`.
- If using floating-point, document precision limits and apply rounding per `--precision`.
- Consider a simple recursive-descent parser or small dependency with no network calls.
- Performance target: handle expressions up to 1,000 chars instantly (<10ms typical).
- Packaging: executable available on macOS/Linux; Windows support desirable but optional.
- Testing approach: golden tests for expression → expected output; negative tests for errors.

## Success Metrics
- Users can compute a basic expression in under 5 seconds from the terminal.
- `--help` provides enough info to use all modes without external docs.
- Zero known crashes on malformed input across tested platforms.
- Test suite achieves high coverage of operators and error paths.

## Open Questions
1. Grammar scope: Should we include exponent `^` and modulo `%` in v1?
2. Scientific notation: Accept `1e-3` numeric literals?
3. Precision default: What default value for `--precision` (e.g., 10)? Should trimming zeros be default or opt-in?
4. Integer division: Should `/` always produce floating results (recommended), or provide an `--integer` mode?
5. CLI name: Is `calc` acceptable or do we need an alternate name to avoid collisions?
6. Error detail: Include position indicators like `at column 5` in errors?
7. Versioning: Semantic version to start at `v0.1.0`?

