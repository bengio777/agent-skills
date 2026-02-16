# Version Coaching — Criteria & Placement Map
**Parent skill:** `prd-builder/SKILL.md`

Read this file during Phase 2 (Version Coaching).

---

## Coaching Criteria

Evaluate each feature from the brain dump against these six criteria to
determine which version it belongs in:

### 1. Persona Dependency
Which persona needs this? If that persona isn't the v1 target (Tier 1), the
feature probably isn't v1 either. A feature for Marcus (Tier 2, v2-v3) belongs
in v2 at earliest, regardless of how cool it sounds.

### 2. Dependency Chain
Does this feature require other features to exist first? Map the chain:
if F007 depends on F003, and F003 depends on F001, then F007 cannot ship
before F001 and F003. Place it in the version where its dependencies are
already complete.

### 3. Complexity vs. Value
High value + low complexity = earlier version. High complexity + niche value =
later. Be honest when something sounds exciting but doesn't justify its build
cost yet. A week of work for a feature one persona uses once a month is a v3
feature, not v1.

### 4. Architectural Implications
Some v3 features require v1 architectural decisions. Flag these aggressively:
"You won't build multi-tenancy until v3, but design your data model with
org-scoped records now, or face a painful migration later." The feature goes
in v3; the architectural decision goes in v1.

### 5. Revenue Unlock
Features that enable monetization or meaningfully expand the addressable market
get version priority over features that polish the experience. A payment
integration that unlocks revenue is more important than a settings page that
improves UX.

### 6. Learning Value
Some features are worth building early because they teach you something
critical about users, even if they're not the "most important" on paper. A
feedback widget might be low-priority by traditional measures, but if it's
how you learn what to build in v2, it's a v1 feature.

---

## Constraint-Driven Scoping

After applying the six criteria, pressure-test the v1 scope against the user's
stated constraints:

**Solo builder, 4 weeks?** v1 gets 3-5 features maximum. Cut aggressively.
**Team of 3, 8 weeks?** v1 can hold 8-12 features. Still prioritize ruthlessly.
**No budget constraints, 6 months?** More features are possible, but resist the
urge to bloat v1. Ship lean, learn fast.

If the v1 feature list exceeds what's realistic for the constraints, coach the
user on what to move to v1.1 or v2. Don't pad the timeline — move the features.

---

## Version Placement Map Format

Present coaching results in this format:

```
FEATURE BRAIN DUMP → VERSION PLACEMENT

v1.0 (MVP — [timeline based on constraints])
  ✅ F001 Invoice creation ← P0, core persona need, revenue-enabling
  ✅ F002 Email delivery ← P0, useless without delivery mechanism
  ✅ F003 Payment tracking ← P0, this is the actual problem being solved
  ⚡ F004 Payment reminders ← P1, high value but not launch-blocking

v2.0 (Retention — after v1 user feedback)
  📋 F006 Client portal ← Requires v1 learning on how clients actually pay
  📋 F007 Expense tracking ← Power user request, expands value prop
  📋 F008 QuickBooks sync ← Dependency: need stable data model first

v3.0 (Growth — expand to teams/agencies)
  🔮 F010 Team workspaces ← New persona (Marcus), Tier 2
  🔮 F011 Role permissions ← Architectural: plan data model in v1
  🔮 F012 White-label invoices ← Revenue unlock for agency segment

v4.0+ (Platform)
  🌐 F015 Public API ← Developer persona (DevRaj), Tier 3
  🌐 F016 Template marketplace ← Requires critical mass of users

⚠️ ARCHITECTURAL DECISIONS NEEDED IN v1 FOR LATER VERSIONS:
  - Org-scoped data model (enables F010, F011 in v3)
  - Event-driven patterns (enables F015 webhooks in v4)
  - Stable UUIDs for all entities (enables F015 API in v4)
```

After presenting, discuss with the user. Key conversation starters:
- "I put [feature] in v2 because [reason]. Do you see it differently?"
- "This v1 scope is [realistic/aggressive] for [constraints]. Want to trim?"
- "These architectural decisions for v1 add ~[time] but prevent major rewrites later. Worth it?"
