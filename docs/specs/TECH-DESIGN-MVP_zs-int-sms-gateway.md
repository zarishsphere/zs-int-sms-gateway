# TECH-DESIGN-MVP — `zs-int-sms-gateway`

> **Document:** Technical Design (MVP) | **Version:** 1.0.0-mvp
> **Repository:** [https://github.com/zarishsphere/zs-int-sms-gateway](https://github.com/zarishsphere/zs-int-sms-gateway)
> **Layer:** Layer 10 | **Catalog #:** 209
> **Language:** Go 1.26.1 | **License:** Apache 2.0

---

## Technical Summary

**SMS gateway adapters — Twilio, Africa's Talking, local operators.**

This document defines the **technical architecture, implementation design, complete repository tree, and acceptance criteria** for the MVP of `zs-int-sms-gateway`.

---

## Data Flow Architecture

```
SMS-GATEWAY System
    ↓ (Outbound (FHIR CommunicationRequest)
zs-int-sms-gateway/internal/adapter/sms-gateway_client.go
    ↓
zs-int-sms-gateway/internal/mapper/to_fhir.go
    ↓
NATS JetStream (zs.integration.sms-gateway.inbound)
    ↓
zs-core-fhir-engine (FHIR R5 CRUD)
    ↓
FHIR AuditEvent written
```

## Retry Pattern

```go
func (s *Syncer) syncWithRetry(ctx context.Context, payload []byte) error {
    backoff := []time.Duration{1, 2, 5, 10, 30}
    for attempt, wait := range backoff {
        err := s.doSync(ctx, payload)
        if err == nil {
            return nil
        }
        if attempt == len(backoff)-1 {
            s.nats.Publish("zs.dlq.sms-gateway", payload)
            return fmt.Errorf("max retries exceeded: %w", err)
        }
        time.Sleep(wait * time.Second)
    }
    return nil
}
```

## FHIR Mapping Example

```go
// internal/mapper/to_fhir.go
func ToFHIRPatient(src *Sms-gatewayPatient) *fhir.Patient {
    return &fhir.Patient{
        ResourceType: "Patient",
        Id:           &src.ID,
        Name: []fhir.HumanName{{
            Family: &src.LastName,
            Given:  []string{src.FirstName},
        }},
        BirthDate: src.DOB,
    }
}
```

---


## Owners & Governance

| Role | GitHub Handle | Responsibility |
|------|--------------|----------------|
| Platform Lead | `@arwa-zarish` | Final approval, RFC votes |
| Technical Lead | `@code-and-brain` | Architecture, Go/TS review |
| DevOps Lead | `@DevOps-Ariful-Islam` | CI/CD, infra, deployment |
| Health Programs | `@BGD-Health-Program` | Clinical content, country programs |

**PR Policy:** All changes via Pull Request. Minimum 1 owner review. CI must pass. No direct commits to `main`.


---

## MVP Acceptance Checklist

- [ ] All MVP files exist in repository with real content (not placeholders)
- [ ] CI pipeline passes on `main` branch
- [ ] No secrets, credentials, or PHI committed
- [ ] README.md reflects current state with setup instructions
- [ ] CODEOWNERS file present
- [ ] All MVP functional requirements verified manually or via automated tests
- [ ] Linked to `CATALOGS.md` and `TODO.md` in `zs-docs-platform`

---

*This document is the authoritative MVP specification for `zs-int-sms-gateway`.*
*Changes require a Pull Request with at least 1 owner approval.*
