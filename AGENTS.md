# Repository Guidelines

## Project Structure & Module Organization
The runtime lives under `app/` (Cosmos SDK wiring) and `cmd/appd/` (CLI entrypoint). Custom logic for loyalty rewards and AI credits is in `x/app/`, mirrored by protobuf definitions in `proto/app/app/v1/`. Generated Go types are committed in `x/app/types/` and `proto/github.com/`. Tests reside alongside their keepers (for example, `x/app/keeper/ai_credits_test.go` and `integration_test.go`). Operational tooling is grouped under `deploy/`, `networks/`, `monitor/`, and `tools/`.

## Build, Test, and Development Commands
- `make build` – compiles the node into `./build/appd`.
- `make install` – installs `appd` into your `$GOBIN`.
- `make test` – runs vet, vulnerability checks, and unit tests.
- `go test ./...` – executes all Go tests; when running in sandboxes, set `GOCACHE`, `GOMODCACHE`, and `GOPATH` to writable paths (e.g., `GOCACHE=$(pwd)/.gocache`).
- `make proto-gen` – regenerates protobuf bindings (requires Ignite CLI); follow with `go mod tidy`.

## Coding Style & Naming Conventions
Go code uses gofmt defaults (tabs for indentation). Stick to Cosmo SDK idioms: keepers in `x/<module>/keeper`, types under `x/<module>/types`, and message servers in `msg_server.go`. Name test helpers with camelCase (`earnPurchasePoints`), and prefer descriptive prefixes for constants (e.g., `MaxCreditsPerUse`). Before submitting changes, run `gofmt` on edited Go files.

## Testing Guidelines
Unit tests rely on Go’s standard testing package with testify assertions. Name tests `Test<Scenario>` and, for table-driven cases, wrap them with `t.Run`. Integration coverage for loyalty/credits lives in `x/app/keeper/integration_test.go`; extend those flows when introducing new state transitions. Aim to keep `make test` green and consider running targeted suites (`go test ./x/app/keeper/...`) before pushing.

## Commit & Pull Request Guidelines
Commit messages should be imperative and focused on a single concern (e.g., `refactor: align reward rule lookups`). Reference relevant issues in the body when applicable. Pull requests must describe the functional change, list test evidence (`go test ./...` output or limitations), and mention any follow-up work. Include screenshots or log snippets when altering developer tooling or documentation that impacts operational workflows.

## Security & Configuration Tips
Respect existing parameter guards (`x/app/types/security.go`) and update both keeper logic and params validation when expanding limits. Store-sensitive configuration (keys, RPC endpoints) belongs in `config.yml` or environment variables, not in source files.*** End Patch
