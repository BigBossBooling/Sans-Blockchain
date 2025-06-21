# DigiSocialBlock - Protocol Buffer Go Generation Guide

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Objective

This document outlines the standard process and commands for generating Go language data structures and supporting code from the Protocol Buffer (`.proto`) definition files used in the EchoNet (also referred to as DigiSocialBlock) project. Adhering to this process ensures consistency and allows developers to easily work with the DLI's core data types.

## 2. Pre-requisites

Before generating Go code from `.proto` files, ensure the following are correctly set up in your development environment:

*   **`protoc` (Protocol Buffer Compiler):**
    *   The `protoc` compiler (version 3.x or later) must be installed and accessible in your system's PATH. This tool parses `.proto` files and invokes plugins to generate code.
    *   Installation instructions can be found on the [Protocol Buffers GitHub repository](https://github.com/protocolbuffers/protobuf) or [official documentation](https://grpc.io/docs/protoc-installation/).
*   **`protoc-gen-go` Plugin:**
    *   This specific plugin is required to generate Go code from `.proto` definitions.
    *   Install or update it using Go tools:
        ```bash
        go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
        ```
    *   Ensure your Go `bin` directory (e.g., `$GOPATH/bin` or `$HOME/go/bin`) is in your system's PATH so `protoc` can find `protoc-gen-go`.
*   **Project `.proto` Files:**
    *   The `.proto` files for EchoNet (e.g., `echonet_v3_core.proto`, and future files like `dds_rpc.proto`, `did_method.proto`, etc.) must be saved in a consistent directory structure within the project. A recommended structure is:
        ```
        <project_root>/
        └── proto/
            └── <package_name>/  // e.g., echonet_core
                └── <version>/       // e.g., v3
                    └── <filename>.proto // e.g., echonet_v3_core.proto
        ```
*   **Dependent Google `.proto` Files:**
    *   Commonly used Google Protocol Buffer types like `google/protobuf/timestamp.proto` and `google/protobuf/any.proto` must be available in the `protoc` include path. These are typically bundled with the `protoc` installation or can be included from a vendored `google/protobuf` directory in the project if needed. If `protoc` is installed correctly, these should generally be found automatically.

## 3. `protoc` Command for Go Code Generation

The following `protoc` command structure should be used to generate Go code from the `.proto` definition files.

### 3.1. Command Structure
```bash
protoc --proto_path=<directory_containing_your_proto_root_packages> \
       --go_out=<output_directory_for_generated_go_files> \
       --go_opt=paths=source_relative \
       <path_to_your_specific.proto_file>
```

### 3.2. Example Command
Assuming:
*   Your project root is the current directory.
*   Your `.proto` files are structured under a top-level `proto/` directory (e.g., `proto/echonet_core/v3/echonet_v3_core.proto`).
*   You want the generated Go files to go into `internal/core/proto/v3` (matching the `go_package` option in the proto file).

The command for `echonet_v3_core.proto` would be:
```bash
protoc --proto_path=proto \
       --go_out=. \
       --go_opt=paths=source_relative \
       proto/echonet_core/v3/echonet_v3_core.proto
```
*(Note: `--go_out=.` combined with `go_package` option in the .proto file and `paths=source_relative` correctly places the output.)*

If `go_package` is `github.com/your_org/digisocialblock/internal/core/proto/v3;corepbv3`, the generated file will be placed in `github.com/your_org/digisocialblock/internal/core/proto/v3/echonet_v3_core.pb.go` relative to the output directory specified by `--go_out`. If `--go_out=.` is used, it means the path in `go_package` is created relative to the current directory.

A common practice is to set `--go_out` to a general `gen/go` directory or directly to the project root if `go_package` options are fully specified to place files correctly. For instance, if your Go modules are set up:
```bash
protoc --proto_path=proto \
       --go_out=./gen/go \
       --go_opt=paths=source_relative \
       proto/echonet_core/v3/echonet_v3_core.proto
```
This would create `gen/go/echonet_core/v3/echonet_v3_core.pb.go`. The `go_package` option in the `.proto` file (`option go_package = "github.com/your_org/digisocialblock/internal/core/proto/v3;corepbv3";`) defines the import path for the generated Go code, not necessarily its output directory structure relative to `--go_out` unless `paths=source_relative` is also used effectively with `go_package` structure.

Given the `go_package` option: `option go_package = "github.com/your_org/digisocialblock/internal/core/proto/v3;corepbv3";`
And assuming your Go project is `github.com/your_org/digisocialblock`, the most direct command from project root is:
```bash
protoc --proto_path=proto \
       --go_out=. \
       --go_opt=paths=source_relative \
       proto/echonet_core/v3/echonet_v3_core.proto
```
This will create the file at `internal/core/proto/v3/echonet_v3_core.pb.go`.

### 3.3. Explanation of Flags
*   `--proto_path=<path>` or `-I <path>`: Specifies the root directory where `protoc` should look for `.proto` files and their imports. If your imports are relative (e.g., `import "google/protobuf/timestamp.proto";`), this path should contain the `google/protobuf/` directory structure, or `protoc` should know where to find it. Multiple `--proto_path` flags can be used. For project-local protos, this is often the root of your proto definitions (e.g., `./proto`).
*   `--go_out=<output_directory>`: Specifies the root directory where the generated Go package directories and `.pb.go` files will be placed.
*   `--go_opt=paths=source_relative`: This option ensures that the generated Go files are placed in a directory structure relative to the output directory that mirrors the directory structure of the source `.proto` files. For example, if `--go_out=gen/go` and the input is `proto/echonet_core/v3/echonet_v3_core.proto`, the output will be `gen/go/echonet_core/v3/echonet_v3_core.pb.go`. If combined with a `go_package` that specifies a full path, it ensures the last part of the `go_package` path aligns with the file's relative location.

## 4. Expected Output

Executing the `protoc` command successfully will produce:
*   A Go file named `<your_proto_filename>.pb.go` (e.g., `echonet_v3_core.pb.go`).
*   This file will be located in a directory structure within your `--go_out` path that corresponds to the `go_package` option specified in the `.proto` file. For example, if `go_package = "github.com/your_org/project/gen/go/core/v3;corepbv3";` and `--go_out=.`, the file will be in `./github.com/your_org/project/gen/go/core/v3/`. A more typical setup is `go_package = "your_project_module_path/internal/gen/proto/core/v3;corepbv3"` and `--go_out=.` leading to `./internal/gen/proto/core/v3/your.pb.go`.
    Given our example `option go_package = "github.com/your_org/digisocialblock/internal/core/proto/v3;corepbv3";` and command `protoc --proto_path=proto --go_out=. --go_opt=paths=source_relative proto/echonet_core/v3/echonet_v3_core.proto`, the output will be `internal/core/proto/v3/echonet_v3_core.pb.go`.

The generated `.pb.go` file will contain:
*   Go struct definitions for each `message` in the `.proto` file.
*   Go type definitions for each `enum` in the `.proto` file.
*   Serialization methods (e.g., `Marshal()`, `Unmarshal()`) for each message.
*   Getter methods for message fields.
*   Other helper functions and type registrations provided by `protoc-gen-go`.
*   If RPC services were defined, it would also generate client and server interface stubs (usually with `protoc-gen-go-grpc`).

## 5. Verification Steps (Conceptual for Developer)

After running the `protoc` command:
1.  **No Errors:** The command should execute without any error messages from `protoc` or `protoc-gen-go`.
2.  **File Generation:** Verify that the expected `.pb.go` file has been generated in the correct output directory as per the `go_package` option and command flags.
3.  **Compilation:** Ensure that the generated Go code compiles successfully as part of your overall DigiSocialBlock Go project (e.g., by running `go build ./...` or `go test ./...` from your project root).
4.  **Usability:** Attempt to import the generated package (e.g., `corepbv3`) in a sample Go file and instantiate/use the generated structs to confirm they are accessible and usable.

## 6. Integration into Build System & CI/CD

Manually running `protoc` is feasible for initial setup but error-prone for ongoing development. This code generation step **must** be integrated into the project's automated build system.

*   **Makefile or Build Scripts:**
    *   Include a target in your `Makefile` (or equivalent build script) that runs the `protoc` command(s) for all `.proto` files.
    *   This target can be invoked manually by developers or automatically by the CI system.
*   **Go Generate Directives:**
    *   A common Go practice is to use `//go:generate` directives within a Go source file (often a `doc.go` or `generate.go` in the proto package directory).
    *   Example (`internal/core/proto/v3/generate.go`):
        ```go
        //go:generate protoc --proto_path=../../../../proto --go_out=. --go_opt=paths=source_relative ../../../../proto/echonet_core/v3/echonet_v3_core.proto
        package corepbv3
        ```
    *   Running `go generate ./...` from the project root will then execute these commands.
*   **CI/CD Pipeline:**
    *   The CI pipeline (as defined in `echonet_v3_cicd_strategy.md` and `roadmap/overall_mvp_implementation_roadmap.md`) should include a step to:
        1.  Verify that all generated `.pb.go` files are up-to-date with their source `.proto` files (e.g., by regenerating them and checking if `git diff` shows any changes; if it does, the commit may have missed checking in generated files).
        2.  Or, always regenerate the files as the first step of the build process in CI.
    *   This ensures that any changes to `.proto` definitions are automatically reflected in the compiled application and tested.

## 7. Example Directory Structure (Conceptual)

A typical project structure might look like this:

```
digisocialblock/
├── proto/  <-- Root for all .proto source files (--proto_path points here)
│   └── echonet_core/
│       └── v3/
│           └── echonet_v3_core.proto
├── internal/
│   └── core/
│       └── proto/
│           └── v3/  <-- Target for generated Go files (derived from go_package)
│               ├── echonet_v3_core.pb.go
│               └── generate.go (optional, for go:generate directives)
├── cmd/      <-- Main applications (e.g., DLI node, CLI client)
├── pkg/      <-- Shared Go libraries (SDKs, etc.)
├── go.mod
└── Makefile  <-- Includes 'make protos' or similar target
```

This guide ensures that all developers can consistently generate the necessary Go code from the authoritative Protocol Buffer definitions, forming a critical part of the development workflow for EchoNet.
