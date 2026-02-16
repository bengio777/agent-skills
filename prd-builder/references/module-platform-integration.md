# Deep-Dive Module: Platform & Integration Strategy
**Parent skill:** `prd-builder/SKILL.md`
**Offered via:** `deep-dive-modules.md`

---

## When to Use

Offered when the product's value depends on connecting with other tools or when
the long-term vision involves becoming a platform. Skip for standalone utilities.

## Part 1: Integration Strategy

### Interview Questions
- What tools do target personas already use daily?
- Which tools must the product connect to for adoption?
- Where does data currently live that needs importing?

### Integration Depth Levels

| Depth | Description | Example |
|-------|-------------|---------|
| Display | Read-only, show external data | Show Stripe payment status |
| Import | One-time or periodic data pull | Import contacts from CSV |
| Sync | Bidirectional, ongoing | Two-way calendar sync |
| Trigger | Event-driven cross-tool actions | Invoice paid → update CRM |
| Embed | Deep UI integration | Slack app, Chrome extension |

### Integration Architecture Map
Per version, classify each integration as required (product breaks without it)
or enhancing (adds value, not blocking). Document: auth method, rate limits,
data freshness, failure handling, entity mapping, migration path to deeper
integration levels.

## Part 2: Platform Strategy

### Platform Evolution Stages

| Stage | Versions | Your Role | Platform Work |
|-------|----------|-----------|---------------|
| Tool | v1-v2 | Build everything | None — focus on core value |
| Extensible Tool | v2-v3 | Expose APIs/webhooks | Public API, docs |
| Platform | v3-v4 | Provide infrastructure | Developer portal, marketplace, SDKs |
| Ecosystem | v4+ | Govern the ecosystem | Quality control, revenue sharing |

### v1 Architectural Decisions That Enable Platform Later
- **API-first design**: Build your product on the same API you'd expose publicly
- **Event-driven architecture**: Internal events become external webhooks trivially
- **Multi-tenancy from day one**: Org-scoped data prevents painful migration
- **Stable UUIDs**: External developers need non-guessable, stable identifiers
- **Idempotent operations**: Integration consumers will send duplicate requests

### Platform Monetization Considerations
API access pricing, marketplace revenue share, premium tier gating, lock-in
effects on retention.

## Output Locations in PRD
- Section 4.1 (System Architecture) — integration patterns, event design
- Section 4.4 (API Design) — platform-trajectory-informed
- Section 4.5 (Third-Party Integrations) — detailed per-version plan
- Section 6.1 (Version Roadmap) — platform stages mapped to versions
- Section 6.2 (Architecture Evolution) — platform-enabling decisions
