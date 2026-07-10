# GridX Protobuf Contracts

This repository contains the source-of-truth Protocol Buffer definitions for the GridX - P2P Energy Trading Platform.

The `.proto` files in this repo define the shared message and service contracts used by the platform's microservices.

Generated SDK code is published into separate SDK repositories:

- `go-sdk`
- `typescript-sdk`
- `cpp-sdk`

**NOTE**: `python-sdk` will be added later when we have finalized the plan to build the ai forecast engine.

## Repository Structure

```text
protobuf/
├── proto/
│   └── gridx/
│       └── test/
│           └── v1/
│               └── test.proto
├── buf.yaml
├── buf.gen.yaml
├── VERSION
├── CHANGELOG.md
└── .github/
    └── workflows/
```

## Requirements

Install buf before working with this repository.

### Arch Linux

```bash
sudo pacman -S buf
```

### macOS

```bash
brew install bufbuild/buf/buf
```

### Linux / Other

See the official Buf installation instructions: [Install buff](https://buf.build/docs/installation)

## Local Development Workflow

Before pushing protobuf changes, run the following commands locally.

1. Format proto files

    ```bash
    buf format -w
    ```

    This rewrites `.proto` files using standard formatting.

    To only check formatting without modifying files:

    ```bash
    buf format --diff --exit-code
    ```

1. Lint proto files

    ```bash
    buf lint
    ```

    This checks protobuf style, naming, package structure, and common schema issues according to buf.yaml

1. Check for breaking changes

    To check that your changes don't break the package for already existing codebases:

    ```bash
    buf breaking --against '.git#branch=main'
    ```

    Examples of breaking changes include:

    * Removing a field
    * Reusing an old field number
    * Changing a field type
    * Moving a message to another package
    * Renaming packages in a breaking way

1. Generate SDK code locally

    ```bash
    buf generate
    ```

    This generates local output into:

    ```text
    gen/
    ├── go/
    ├── typescript/
    └── cpp/
    ```

    The local `gen/` directory is ignored by Git in this repository.

    Generated code is committed only in the SDK repositories by GitHub Actions.

1. Clean generated files

    ```bash
    rm -rf gen/
    ```

## Full Local Check

Run this before pushing:

```bash
buf format -w
buf lint
buf breaking --against '.git#branch=main'
buf generate
```

If all commands pass, the protobuf definitions are ready for a pull request.

## Versioning

This repository uses automated versioning.

Developers should not manually edit release versions.

Versioning is handled by Release Please based on Conventional Commit messages.

Examples of commit messages:

```text
feat(order): add place order request
fix(matching): correct trade executed event field
docs: update Kafka topic documentation
```

Version bump behavior:

| Commit Type                    | Version Bump       |
| ------------------------------ | ------------------ |
| `fix:`                         | Patch              |
| `feat:`                        | Minor              |
| `feat!:` or `BREAKING CHANGE:` | Major              |
| `docs:` / `chore:`             | Usually no release |

## Generated SDK Release Flow

When a release is created from this repository:

```text
gridx-proto v0.2.0
    ↓
go-sdk v0.2.0
typescript-sdk v0.2.0
cpp-sdk v0.2.0
```

The SDK generation workflow:

1. Checks out this repository at the release tag.
1. Runs buf generate.
1. Copies generated Go code to go-sdk.
1. Copies generated TypeScript code to typescript-sdk.
1. Copies generated C++ code to cpp-sdk.
1. Updates SDK VERSION files.
1. Tags each SDK repository with the same version.

## Protobuf Guidelines

Follow these rules when editing `.proto` files:

### Use versioned packages

**Good:**

```proto
package gridx.order.v1;
```

**Avoid unversioned packages:**

```proto
package gridx.order;
```

### Keep file path and package aligned

**Good:**

If file path is `proto/gridx/test/v1/test.proto`

Then

```proto
package gridx.test.v1;
```

### Always set go_package

**Example:**

```proto
option go_package = "github.com/p2p-energy-trading-platform/go-sdk/gen/gridx/test/v1;testv1";
```

### Do not reuse field numbers

**Bad:**

Initially

```proto
message Example {
    string old_name = 1;
}
```

Then later

```proto
message Example {
    string new_meaning = 1;
}
```

**Instead, reserve removed fields:**

```proto
message Example {
    reserved 1;
    reserved "old_name";

    string new_field = 2;
}
```

### Prefer additive changes

Safe changes usually include:

* Adding a new message
* Adding a new field with a new field number
* Adding a new service method carefully
* Adding a new enum value carefully

Risky changes include:

* Removing fields
* Renaming packages
* Changing field types
* Reusing field numbers
* Moving messages between packages

