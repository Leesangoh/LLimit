# CodexBar vs LLimit: Analysis & Roadmap

## Executive Summary

CodexBar is a mature macOS menu-bar app supporting **31 AI providers** with sophisticated fallback chains (OAuth → CLI → Web → API), cost tracking, browser cookie import, WidgetKit, CLI, and status-page incident badges. LLimit currently supports **2 providers** (Claude + Codex) but has a clear multi-account UX advantage that CodexBar lacks — CodexBar is single-account-per-provider.

**Key takeaway**: LLimit should not chase 31-provider parity. Instead, focus on the top 5-7 providers that matter, then differentiate with multi-account, team-oriented features, and a cleaner UX.

---

## 1. Provider Comparison Table

| Provider | CodexBar | LLimit | Auth Method | Data Source | Complexity |
|----------|----------|--------|-------------|-------------|------------|
| **Claude** | Yes (OAuth+CLI+Web) | Yes (OAuth) | OAuth PKCE | /api/oauth/usage + /profile | Done |
| **Codex** | Yes (OAuth+CLI+Web) | Yes (API) | OAuth (CLI login) | chatgpt.com/backend-api | Done |
| **Cursor** | Yes | No | Browser cookies | Browser local storage | Medium |
| **Copilot** | Yes | No | GitHub device flow | GitHub API | Medium |
| **Gemini** | Yes | No | Gemini CLI OAuth | Google Gemini API | Medium |
| **Windsurf/OpenCode** | Yes | No | Browser cookies | OpenCode API | Low-Medium |
| **Kiro** | Yes | No | CLI command | kiro-cli /usage | Low |
| **JetBrains AI** | Yes | No | Local XML config | IDE config file | Low |
| **OpenRouter** | Yes | No | API token | OpenRouter API | Low |
| **Perplexity** | Yes | No | Browser cookies | Perplexity API | Low-Medium |
| **z.ai** | Yes | No | API token (keychain) | z.ai API | Low |
| **Vertex AI** | Yes | No | gcloud OAuth | GCP Cloud Monitoring | High |
| **Others (19+)** | Yes | No | Various | Various | Varies |

---

## 2. Architecture Map

### CodexBar Architecture

```
┌─────────────────────────────────────────────────┐
│                   CodexBar App                    │
├─────────────────────────────────────────────────┤
│  StatusItemController (AppKit NSMenu)             │
│  └─ UsageMenuCardView (SwiftUI per provider)      │
│     └─ Metrics, Cost, Credits, Identity           │
├─────────────────────────────────────────────────┤
│  UsageStore (state management + background poll)  │
│  └─ Per-provider refresh tracking                 │
│  └─ Failure gates (exponential backoff)           │
│  └─ Session transition detection                  │
├─────────────────────────────────────────────────┤
│  ProviderDescriptorRegistry (31 providers)        │
│  └─ @ProviderDescriptorRegistration macro         │
│  └─ ProviderDescriptor (metadata + branding +     │
│     fetchPlan + pipeline)                         │
├─────────────────────────────────────────────────┤
│  ProviderFetchStrategy (protocol)                 │
│  ├─ OAuthStrategy                                 │
│  ├─ CLIStrategy                                   │
│  ├─ WebDashboardStrategy                          │
│  ├─ APITokenStrategy                              │
│  └─ LocalProbeStrategy                            │
├─────────────────────────────────────────────────┤
│  Auth Layer                                       │
│  ├─ Keychain (OAuth tokens, API keys)             │
│  ├─ SweetCookieKit (browser cookie extraction)    │
│  ├─ CookieHeaderCache                             │
│  └─ CLILoginRunner (PTY-based CLI auth)           │
└─────────────────────────────────────────────────┘
```

### LLimit Architecture (Current)

```
┌─────────────────────────────────────────────────┐
│                   LLimit App                      │
├─────────────────────────────────────────────────┤
│  MenuBarExtra (SwiftUI scene)                     │
│  └─ MenuBarLabel (NSImage rasterised bars)        │
│  └─ MenuContentView (account cards)               │
├─────────────────────────────────────────────────┤
│  RefreshCoordinator (timer + parallel refresh)    │
│  └─ Per-account state tracking                    │
│  └─ Rate limit coalescing + disk cache            │
├─────────────────────────────────────────────────┤
│  AccountStore (multi-account management)          │
│  └─ Per-account credential snapshots              │
│  └─ Auto-detection of CLI config dirs             │
├─────────────────────────────────────────────────┤
│  UsageAPI (protocol)                              │
│  ├─ AnthropicUsageAPI (Claude)                    │
│  └─ OpenAIUsageAPI (Codex)                        │
├─────────────────────────────────────────────────┤
│  Auth Layer                                       │
│  ├─ ClaudeAuthSource (per-account OAuth + keychain)│
│  ├─ CodexAuthSource (auth.json)                   │
│  └─ ClaudeOAuthLogin (PKCE flow)                  │
└─────────────────────────────────────────────────┘
```

### Key Architectural Differences

| Aspect | CodexBar | LLimit |
|--------|----------|--------|
| Provider model | 1 account per provider | N accounts per provider |
| Fetch strategy | Multi-strategy fallback chain | Single strategy per provider |
| Auth storage | Keychain + cookie cache | Per-account JSON snapshots |
| UI framework | AppKit NSMenu + SwiftUI cards | SwiftUI MenuBarExtra |
| State management | UsageStore (centralized) | AccountStore + RefreshCoordinator |
| Registration | Compile-time macro | Hardcoded enum |

---

## 3. UI/UX Feature Parity Checklist

| Feature | CodexBar | LLimit | Priority |
|---------|----------|--------|----------|
| Menu bar usage bars | Yes | Yes | Done |
| Popover with account cards | Yes | Yes | Done |
| Per-account usage display | Yes (1 per provider) | Yes (N per provider) | LLimit stronger |
| Settings UI | Yes (tabbed, per-provider) | Yes (basic) | Done |
| Refresh interval config | Yes (1m/2m/5m/15m) | Yes (1-120m) | Done |
| Threshold notifications | Yes | Yes | Done |
| Launch at login | Yes | Yes | Done |
| Cost tracking (tokens/USD) | Yes (Claude+Codex) | No | Must-have |
| Status page badges | Yes | No | Optional |
| Merge icons mode | Yes | No | Optional |
| WidgetKit support | Yes | No | Optional |
| CLI support | Yes | No | Optional |
| Keyboard shortcuts | Yes | No | Not relevant |
| Auto-update (Sparkle) | Yes | No | Must-have |
| Browser cookie import | Yes | No | Must-have (for new providers) |
| Pace/trend indicator | Yes | No | Optional |
| Display mode (% left vs used) | Yes | No | Nice to have |
| Overview tab (top 3) | Yes | No | Optional |
| Credits display | Yes | No | Must-have (for Cursor etc.) |

---

## 4. Gap Analysis

### Where CodexBar is Stronger
1. **Provider coverage** — 31 vs 2
2. **Cost tracking** — token costs from local JSONL logs
3. **Fallback strategies** — OAuth → CLI → Web per provider
4. **Browser cookie import** — SweetCookieKit for 6+ browsers
5. **Status page polling** — incident badge overlays
6. **Auto-update** — Sparkle framework
7. **CLI/Widget** — cross-platform CLI + WidgetKit

### Where LLimit is Already Stronger
1. **Multi-account per provider** — CodexBar is single-account-per-provider
2. **Per-account OAuth** — separate credential snapshots per account
3. **macOS 26 support** — tested and working
4. **Simpler codebase** — easier to extend and maintain
5. **Direct API usage** — no CLI dependency for Claude plan/email

### Where LLimit Can Build a Durable Moat
1. **Multi-account UX** — teams with multiple Claude/Codex orgs
2. **Account-level cost tracking** — per-account token spend
3. **Account grouping** — "Work" vs "Personal" vs "Client X"
4. **Cross-account aggregation** — total spend across all accounts
5. **Account-scoped notifications** — different thresholds per account

---

## 5. Proposed Roadmap

### Phase 1: Parity (4-6 weeks)

| Feature | Dependencies | Risk | Impact |
|---------|-------------|------|--------|
| **Add Cursor provider** | Cookie import (SweetCookieKit or manual) | Medium (cookie auth) | High (very popular) |
| **Add Copilot provider** | GitHub device flow OAuth | Low | High (widely used) |
| **Add Gemini provider** | Gemini CLI detection | Low | Medium |
| **Cost tracking (Claude)** | Parse ~/.claude/projects/*/logs | Medium (file parsing) | High |
| **Auto-update (Sparkle)** | SPM dependency | Low | High (retention) |
| **Provider abstraction protocol** | Refactor UsageAPI | Low | Critical (enables scale) |

### Phase 2: Multi-Account Differentiation (4-6 weeks)

| Feature | Dependencies | Risk | Impact |
|---------|-------------|------|--------|
| **Account groups** ("Work", "Personal") | UI + data model | Low | High |
| **Per-account cost tracking** | Phase 1 cost tracking | Medium | High |
| **Cross-account aggregation** | Account groups | Low | Medium |
| **Account-scoped notifications** | Notification refactor | Low | Medium |
| **Team dashboard view** | Account groups | Medium | High |
| **Export/report** (CSV/JSON) | Cost data model | Low | Medium |

### Phase 3: Premium / Defensible Features (ongoing)

| Feature | Dependencies | Risk | Impact |
|---------|-------------|------|--------|
| **Usage trends & charts** | Historical data storage | Medium | High |
| **Budget alerts** (cost-based) | Cost tracking | Low | High |
| **API/webhook integration** | Server component | High | Medium |
| **WidgetKit support** | App Groups | Medium | Medium |
| **CLI for scripting** | Core refactor | Low | Low-Medium |
| **Browser extension** (cookie sync) | WebExtension | High | Medium |

---

## 6. Technical Recommendations

### Provider Adapter Interface

```swift
protocol ProviderAdapter: Sendable {
    static var providerType: ProviderType { get }
    static var displayName: String { get }
    static var iconName: String { get }

    func authenticate(account: Account) async throws
    func fetchUsage(account: Account) async throws -> NormalizedUsage
    func fetchProfile(account: Account) async throws -> ProviderProfile?
    func fetchCost(account: Account) async throws -> CostSnapshot?

    var isAvailable: Bool { get }
}
```

### Normalized Usage Model

```swift
struct NormalizedUsage {
    let windows: [UsageWindow]       // existing model works
    let credits: CreditsInfo?        // for credit-based providers
    let cost: CostInfo?              // token cost breakdown
    let identity: IdentityInfo?      // email, plan, org
    let fetchedAt: Date
}

struct CreditsInfo {
    let remaining: Double
    let total: Double
    let resetDate: Date?
}

struct CostInfo {
    let sessionCostUSD: Double?
    let dailyCostUSD: Double?
    let monthlyCostUSD: Double?
    let tokenBreakdown: TokenBreakdown?
}
```

### Multi-Account + Multi-Provider Composition

```
Account
  ├─ id: UUID
  ├─ name: String
  ├─ provider: ProviderType      // .claude, .codex, .cursor, ...
  ├─ group: AccountGroup?        // "Work", "Personal"
  ├─ authState: AuthState        // per-account credential
  └─ settings: ProviderSettings  // per-account overrides

AccountGroup
  ├─ name: String
  ├─ accounts: [Account]
  └─ aggregatedUsage: NormalizedUsage  // computed
```

---

## 7. "Build Next" Top 5

1. **Provider abstraction protocol** — refactor UsageAPI into a proper provider adapter interface. This unblocks everything else.

2. **Add Cursor provider** — #1 most requested after Claude/Codex. Cookie-based auth, monthly credits model. High user impact.

3. **Add Copilot provider** — GitHub device flow OAuth is well-documented. Large user base overlap.

4. **Cost tracking for Claude** — parse local JSONL logs for token costs. High differentiation when combined with multi-account.

5. **Auto-update via Sparkle** — critical for retention. Users won't manually re-install for updates.

---

## Provider Priority Matrix

```
                    HIGH IMPACT
                        │
         Cursor ────────┼──────── Claude Cost
                        │
    LOW EFFORT ─────────┼───────── HIGH EFFORT
                        │
         Copilot ───────┼──────── Gemini
         Kiro           │         Vertex AI
         OpenRouter     │
                        │
                    LOW IMPACT
```

Focus on the upper-left quadrant first: Cursor, Copilot, then Claude cost tracking.
