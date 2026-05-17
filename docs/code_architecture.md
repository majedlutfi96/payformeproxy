 # Payforme Proxy Code Structure

This layout uses a practical Go structure:

- **`cmd/`** for binaries
- **`internal/`** for app implementation
- **`pkg/`** for public reusable interfaces/types
- **`configs/`** for example JSON config
- **`docs/`** for architecture notes
- **tests next to packages** using Go convention

## Directory Tree

```text
payformeproxy/
├── go.mod
├── README.md
├── configs/
│   └── config.example.json
├── cmd/
│   └── payforme/
│       └── main.go
├── internal/
│   ├── app/
│   │   ├── app.go
│   │   └── app_test.go
│   ├── auth/
│   │   ├── basic.go
│   │   └── basic_test.go
│   ├── chains/
│   │   ├── registry.go
│   │   ├── registry_test.go
│   │   ├── algorand/
│   │   │   ├── client.go
│   │   │   ├── payment.go
│   │   │   └── payment_test.go
│   │   └── solana/
│   │       ├── client.go
│   │       ├── payment.go
│   │       └── payment_test.go
│   ├── config/
│   │   ├── config.go
│   │   ├── loader.go
│   │   ├── store.go
│   │   └── loader_test.go
│   ├── logging/
│   │   ├── logger.go
│   │   └── logger_test.go
│   ├── payments/
│   │   ├── challenge.go
│   │   ├── processor.go
│   │   ├── processor_test.go
│   │   └── x402.go
│   ├── policy/
│   │   ├── whitelist.go
│   │   ├── whitelist_test.go
│   │   ├── wallet_policy.go
│   │   └── wallet_policy_test.go
│   ├── proxy/
│   │   ├── server.go
│   │   ├── handlers.go
│   │   ├── middleware.go
│   │   └── server_test.go
│   ├── tui/
│   │   ├── model.go
│   │   ├── screens.go
│   │   └── model_test.go
│   ├── users/
│   │   ├── user.go
│   │   ├── repository.go
│   │   └── repository_test.go
│   └── wallets/
│       ├── wallet.go
│       ├── repository.go
│       ├── selector.go
│       └── selector_test.go
├── pkg/
│   └── payforme/
│       ├── errors.go
│       ├── types.go
│       └── interfaces.go
└── docs/
    ├── architecture.md
    ├── configuration.md
    └── testing.md
```

## Package Responsibilities

### `cmd/payforme`

Application entrypoint.

`main.go` should eventually load configuration, initialize logging, initialize wallet repositories, register supported chains, create the proxy server, start the HTTP proxy, and optionally start TUI mode.

### `internal/app`

Application composition layer.

`app.go` should define the `App` struct, `New`, `Run`, and `Shutdown`. It should wire config, logger, proxy server, payment processor, and chain registry.

`app_test.go` should test dependency wiring, startup failure handling, and graceful shutdown behavior.

### `internal/auth`

Basic authentication.

`basic.go` should define `BasicAuthenticator`, `Authenticate(username, password string) bool`, and proxy authentication helpers.

`basic_test.go` should test valid credentials, invalid credentials, missing credentials, and multiple users.

### `internal/config`

JSON configuration loading and storing.

`config.go` should define `Config`, `ServerConfig`, `AuthConfig`, `UserConfig`, `WalletConfig`, `ChainConfig`, `PolicyConfig`, and `TUIConfig`.

`loader.go` should implement JSON config loading, validation, and defaults.

`store.go` should implement JSON config saving, atomic writes, and optional backup strategy.

`loader_test.go` should test valid config loading, missing files, invalid JSON, default values, and validation errors.

### `internal/logging`

Logging setup.

`logger.go` should define a logger abstraction and setup production/development structured logging, using standard library `rs/zerolog`.

`logger_test.go` should test logger construction, log level parsing, and safe defaults.

### `internal/proxy`

HTTP proxy implementation using `github.com/elazarl/goproxy`.

`server.go` should define `Server`, `NewServer`, `Start`, `Shutdown`, and goproxy initialization.

`handlers.go` should contain request interception, response interception, 402 detection, and retry-after-payment flow.

`middleware.go` should contain basic auth enforcement, domain whitelist checks, user context extraction, and request logging middleware.

`server_test.go` should test proxy startup configuration, 402 response handling, auth rejection, whitelist rejection, and successful retry flow with a mocked payment processor.

### `internal/payments`

Generic `x402` payment logic.

`x402.go` should define x402 challenge structs, payment requirement structs, payment signature structs, and header encoding/decoding helpers.

`challenge.go` should read and decode the `Payment-Required` header, select compatible payment options, and validate challenge fields.

`processor.go` should define `Processor`, `Pay`, chain selection, wallet selection, and payment header generation.

`processor_test.go` should test Algorand option selection, unsupported chains, malformed challenges, wallet lookup failure, and successful mocked payment flow.

### `internal/chains`

Multi-chain support.

`registry.go` should define `Registry`, chain handler registration, lookup by network string, and network prefix matching.

`registry_test.go` should test registering chains, duplicate registration, unknown chains, and network prefix matching.

### `internal/chains/algorand`

Algorand implementation.

`client.go` should define the algod client wrapper, suggested params retrieval, network validation, and signing helpers.

`payment.go` should implement ASA transfer construction, sponsored fee group construction, `PAYMENT-SIGNATURE` payload encoding, and multi-wallet support.

`payment_test.go` should test ASA transfer construction, sponsored fee group construction, missing asset validation, and header payload shape.

### `internal/chains/solana`

Solana implementation placeholder.

`client.go` should define an RPC client wrapper, network config, and wallet signing abstraction.

`payment.go` should eventually implement SPL token transfer, x402 payment payload generation, and fee payer handling.

`payment_test.go` should define future test cases for SPL transfer, unsupported networks, missing token accounts, and fee payer handling.

### `internal/wallets`

Wallet management.

`wallet.go` should define `Wallet`, wallet ID, owner user ID, chain, address, encrypted secret reference, and labels/tags.

`repository.go` should implement wallet lookup, creation, update, deletion, and initial JSON-backed storage.

`selector.go` should implement wallet selection by user, chain, domain, and fallback rules.

`selector_test.go` should test wallet selection by user, wallet selection by chain, no wallet found, and domain-specific wallet selection.

### `internal/users`

User management.

`user.go` should define `User`, username, password hash, enabled/disabled status, and user-specific policies.

`repository.go` should implement user lookup, credential lookup, and JSON-backed user persistence.

`repository_test.go` should test finding users, missing users, disabled users, and credential verification integration.

### `internal/policy`

Whitelisting and access rules.

`whitelist.go` should implement allowed domains, blocked domains, wildcard matching, and per-user policy checks.

`wallet_policy.go` should implement allowed wallets per user, allowed chains per user, domain-to-wallet mapping, and later spending limits.

Tests should cover allowed domains, blocked domains, wildcard domains, user-specific policy, and wallet access denial.

### `internal/tui`

Terminal UI configuration.

`model.go` should define the TUI application model, current screen state, config editing state, and error/success messages.

`screens.go` should define placeholders for users, wallets, chains, whitelist, and server settings screens.

`model_test.go` should test initial model state, screen transitions, and config update actions.

### `pkg/payforme`

Public package for shared types and interfaces.

`interfaces.go` should define public interfaces such as `ChainPayer`, `WalletStore`, `UserStore`, `PolicyChecker`, and `PaymentProcessor`.

`types.go` should define public shared types such as `UserID`, `WalletID`, `ChainName`, `NetworkID`, `PaymentRequest`, and `PaymentResult`.

`errors.go` should define sentinel errors such as `ErrUnauthorized`, `ErrWalletNotFound`, `ErrChainUnsupported`, `ErrPaymentRequiredMalformed`, and `ErrPolicyDenied`.

## Config Example Responsibilities

`configs/config.example.json` should include example configuration for:

- **Proxy address**
- **Basic auth users**
- **Wallets**
- **Algorand network**
- **Domain whitelist**
- **User wallet permissions**
- **TUI enabled flag**

## Docs Responsibilities

`docs/architecture.md` should explain request flow, 402 handling flow, payment processor design, chain adapter design, and wallet selection design.

`docs/configuration.md` should document the JSON config schema, example config, and future environment overrides.

`docs/testing.md` should document unit test strategy, mocking chain clients, proxy integration tests, and avoiding real payments in tests.

## Recommended Initial Dependencies

For the first real implementation stage, keep dependencies minimal:

```text
github.com/elazarl/goproxy
```

Later, add chain-specific dependencies only when implementing each chain:

```text
github.com/algorand/go-algorand-sdk/v2
```

For logging, prefer the standard library:

```text
log/slog
```

For TUI later:

```text
github.com/charmbracelet/bubbletea
github.com/charmbracelet/lipgloss
```

## Suggested Architecture Style

Use a simple layered architecture:

```text
cmd -> internal/app -> internal/proxy -> internal/payments -> internal/chains
                                  └-> internal/auth
                                  └-> internal/policy
                                  └-> internal/wallets
                                  └-> internal/users
```

This keeps the project beginner-friendly while still applying Go best practices:

- **Small packages**
- **Interfaces at boundaries**
- **Tests next to code**
- **No circular dependencies**
- **Concrete implementations in `internal/`**
- **Shared public contracts in `pkg/payforme`**