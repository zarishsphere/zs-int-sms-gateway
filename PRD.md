# PRD — `zs-int-sms-gateway`

> **Document Class:** PRD | **Version:** 1.0.0 | **Status:** Bootstrapping
> **Repository:** [https://github.com/zarishsphere/zs-int-sms-gateway](https://github.com/zarishsphere/zs-int-sms-gateway)
> **Layer:** Layer 10 — Integrations | **Catalog #:** 209
> **License:** Apache 2.0 | **Governance:** RFC-0001

---

## 1. Overview

SMS gateway adapters — Twilio, Africa's Talking, local operator APIs for notifications.

---

## 2. Repository Metadata

- **Name:** `zs-int-sms-gateway`
- **Organization:** [https://github.com/zarishsphere](https://github.com/zarishsphere)
- **Language / Runtime:** Go 1.26.1
- **Visibility:** Public
- **License:** Apache 2.0
- **Default Branch:** `main`
- **Branch Protection:** Required (2-owner review for critical paths)

---

## 3. Platform Context

This repository is part of the **ZarishSphere** sovereign digital health operating platform — a free, open-source, FHIR R5-native system for South and Southeast Asia.

**Non-negotiable constraints:**
- Zero cost — all tooling must use genuinely free tiers
- FHIR R5 native — all clinical data modelled as FHIR R5 resources
- Offline-first — must work without network connectivity
- No-coder friendly — GUI-first, template-driven
- Documentation as Code — all decisions in GitHub

---

## 4. Goals & Objectives

- Bridge ZarishSphere FHIR R5 data with SMS_GATEWAY system
- Implement bidirectional data exchange where specified
- Provide robust error handling and retry logic

## 5. Functional Requirements

| ID | Requirement | Priority |
|----|------------|---------|
| F-01 | Bidirectional data exchange (or directional as specified) | P0 |
| F-02 | Configurable mapping between SMS_GATEWAY and FHIR R5 | P0 |
| F-03 | Error handling with dead letter queue (NATS) | P0 |
| F-04 | Retry with exponential backoff | P0 |
| F-05 | FHIR AuditEvent for all data exchanges | P1 |
| F-06 | Integration test with testcontainers-go | P0 |

## 6. Repository Tree

```
zs-int-sms-gateway/
├── README.md
├── LICENSE
├── go.mod
├── go.sum
├── .gitignore
├── .github/
│   ├── CODEOWNERS
│   └── workflows/
│       └── ci.yml
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── adapter/
│   │   ├── sms_gateway_client.go       # SMS_GATEWAY API client
│   │   └── fhir_mapper.go             # SMS_GATEWAY ↔ FHIR R5 mapping
│   ├── sync/
│   │   └── syncer.go                  # Sync orchestration logic
│   └── config/
│       └── config.go
├── config/
│   └── config.yaml
├── docs/
│   ├── openapi.yaml
│   └── MAPPING.md                     # Data field mapping documentation
└── tests/
    └── integration/
        └── suite_test.go
```

### CI/CD (`.github/workflows/ci.yml`)

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version-file: go.mod, cache: true }
      - run: go test ./... -race -coverprofile=coverage.out
      - uses: golangci/golangci-lint-action@v6
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aquasecurity/trivy-action@master
        with: { scan-type: fs, severity: CRITICAL,HIGH }
```

## 9. Ownership & Governance

| Role | GitHub User |
|------|-------------|
| Platform Lead | `@arwa-zarish` |
| Technical Lead | `@code-and-brain` |
| DevOps Lead | `@DevOps-Ariful-Islam` |
| Health Programs | `@BGD-Health-Program` |

All changes go through Pull Request → 1+ owner review → CI pass → merge.
Breaking changes require RFC + ADR.

---

## 10. Definition of Done

- [ ] All listed files exist with content
- [ ] CI pipeline passes (test + lint + security)
- [ ] README.md reflects current state
- [ ] OpenAPI / AsyncAPI spec present (services only)
- [ ] At least 1 integration test using testcontainers-go (Go) or Playwright (UI)
- [ ] No secrets committed (GitGuardian verified)
- [ ] CODEOWNERS file present
- [ ] Linked to CATALOGS.md and TODO.md

---

*This PRD is the canonical source of truth for this repository's purpose, structure, and requirements.*
*Changes require a PR against this file with owner approval.*
