# scg-notification-contracts

[![CI](https://github.com/next-trace/scg-notification-contracts/actions/workflows/ci.yml/badge.svg)](https://github.com/next-trace/scg-notification-contracts/actions/workflows/ci.yml)
[![Buf Checks](https://img.shields.io/github/actions/workflow/status/next-trace/scg-notification-contracts/ci.yml?branch=main&label=buf%20checks)](https://github.com/next-trace/scg-notification-contracts/actions/workflows/ci.yml)

Canonical, stable Protobuf contracts for SupplyChainGuard (SCG) notifications. These contracts are shared across services and clients to ensure consistent, evolvable event and notification schemas.

- Single source of truth for notification messages and enums used across SCG domains
- Stability first: linting and breaking-change checks enforced via Buf and CI
- Generated Go packages for easy consumption in Go microservices

## Overview

SCG emits domain events (e.g., compliance, logistics, custody). This repository defines notification-oriented contracts that describe how those events are communicated to users and systems across channels (email, SMS, push, webhooks, in-app, digests, escalations). Contracts live under proto/scg/** and are versioned to maintain compatibility.

Key areas covered include:
- Enums for channels, severity, categories, audiences, and statuses
- Models for preferences, deliveries, digests, and core notifications
- Event messages for major supply chain workflows (onboarding, compliance, logistics, digital twin, risk, system, etc.)

## Tooling

This repo uses Buf for linting, breaking-change detection, and code generation.

Install Buf and Go plugins:
- Using Go (simplest, cross-platform):
  - go install github.com/bufbuild/buf/cmd/buf@latest
  - go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
  - go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
- Or see Buf’s docs for other install methods: https://buf.build/docs/installation

Run linting:
- buf lint

Check for breaking changes (compare against main):
- buf breaking --against '.git#branch=main'

You can also run the local helper which wraps the above:
- ./scg proto-gen

Important: JSON naming
- All message fields explicitly set json_name to snake_case across the schemas.
- When serializing to JSON, standard Go/Protobuf JSON marshallers will emit snake_case names.
- Regenerate code after any .proto change to ensure json struct tags are up-to-date:
  - buf generate

Configuration lives in:
- buf.yaml (module name, lint, breaking)
- buf.gen.yaml (codegen plugins and output locations)

## Code Generation (Go)

Generate Go code into gen/go using Buf:
- buf generate

Notes:
- This uses the go and go-grpc plugins configured in buf.gen.yaml
- Ensure protoc-gen-go and protoc-gen-go-grpc are on your PATH (see Tooling)
- Generated packages are laid out under gen/go/proto/scg/... and are importable via the module path

## Usage Example (Go)

In a Go service, add this module:
- go get github.com/next-trace/scg-notification-contracts@vX.Y.Z

Import generated types, for example a shared notification event:

import (
    sharedevents "github.com/next-trace/scg-notification-contracts/gen/go/proto/scg/shared/v1"
)

envelope := &sharedevents.NotificationEventEnvelope{
    EventUuid:    "e7a1c3e4-1234-5678-9abc-def012345678",
    EventType:    "system.maintenance",
    ActorUuid:    "user-123",
    TraceUuid:    "trace-abc",
    SchemaVersion: 1,
}

// Note: All JSON output uses snake_case via explicit json_name options in the .proto files.
// After buf generate, the Go structs include json tags aligned with snake_case.

## Versioning and Stability

We follow Semantic Versioning (SemVer):
- MAJOR: backward-incompatible changes to protobuf contracts
- MINOR: backward-compatible additions (new messages/fields with sensible defaults)
- PATCH: backward-compatible fixes and clarifications

Consumer guidance:
- Pin to a released tag in go.mod, e.g.: require github.com/next-trace/scg-notification-contracts v1.2.3
- Avoid relying on main in production services
- When upgrading MAJOR versions, run buf breaking against your current tag to assess impact:
  - buf breaking --against '.git#tag=v1.2.3'

## Testing and Quality

- CI runs build, tests, linting, security scans, and protobuf generation
- Buf linting and breaking checks are executed in CI to protect stability

Quick checks locally:
- buf lint
- buf breaking --against '.git#branch=main'
- buf generate
- ./scg ci  # builds, tests, lints, security, and proto generation

## License

This project is licensed under the MIT License. See the LICENSE file.

## Security

If you discover a security issue, please do not open a public issue. Instead, follow the guidance in SECURITY.md (or contact the maintainers privately if that file is not present yet). We aim to respond promptly.

## Changelog

Changes are tracked via Git tags and Git history. When available, see CHANGELOG.md for curated release notes. Each release describes added, changed, and removed contracts to aid safe upgrades.

---

Focus: This repository is contract-first and stability-focused. Please run Buf checks and avoid breaking changes unless performing a planned MAJOR release.
