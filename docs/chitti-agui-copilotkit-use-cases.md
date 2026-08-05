# Chitti × AG-UI / CopilotKit — derived use cases

Derived against the real codebase (`amaljithkuttamath.github.io`,
`src/lib/chitti/`), not against a generic agent app. Chitti is a **browser-only,
zero-backend, BYOK data-analyst agent** built in Astro + vanilla TypeScript: it
fetches live numbers from World Bank / OWID / IMF / WHO, computes over them,
charts them, and verifies the answer. Every use case below is judged against
those constraints, because they are what make most of the standard CopilotKit
story inapplicable.

---

## 0. The one fact that decides everything

Two constraints pull in opposite directions:

- **AG-UI does not need a server.** Its TypeScript SDK exposes `AbstractAgent`,
  whose `run(input)` returns an `Observable<BaseEvent>`. A subclass can run the
  entire loop in the browser. `HttpAgent` is a convenience, not a requirement.
  So Chitti's `session.ask()` can become a conforming AG-UI agent with **no
  backend added and no BYOK posture broken**.
- **CopilotKit is a React/UI layer.** Adopting it means an `@astrojs/react`
  island plus React, React-DOM and the CopilotKit packages inside a PWA whose
  current runtime dependency list is Astro and three font packages.

The conclusion this leads to, stated up front: **AG-UI is a near-free fit at the
library layer; CopilotKit is expensive at `/apps/chitti` and cheap on a new
surface.** The staging in §4 follows from that.

---

## 1. Chitti already speaks this protocol — in bespoke form

Chitti independently built most of what AG-UI standardizes. That is the reason
to look at it, and also the reason to be selective: re-plumbing something that
already works is not a use case.

| Chitti today | AG-UI event | CopilotKit surface |
| --- | --- | --- |
| `AgentCallbacks.onStatus` + `AbortedError` | `RunStarted` / `RunFinished` / `RunError` | run lifecycle in `useAgent` |
| `TraceEvent{tool, argSummary, status, detail}` | `ToolCallStart` / `ToolCallArgs` / `ToolCallEnd` / `ToolCallResult` | `useRenderToolCall` |
| `AgentOutput.finding` (streamed prose) | `TextMessageStart` / `Content` / `End` | chat message stream |
| `InsightBrief` plan card + `matchStepToEvent` tick-off | `ActivitySnapshot` / `ActivityDelta` | activity renderer |
| planner brief, verifier rationale | `ReasoningStart` / `ReasoningMessage*` / `ReasoningEnd` | reasoning renderer |
| `chartSpec` / `rows` / `citations` / `dashboard` | `StateSnapshot` / `StateDelta` (JSON Patch) | `useAgent` state binding |
| `share.ts` / `dashboard-share.ts` permalinks | `MessagesSnapshot` + `StateSnapshot` | restore-by-replay |
| `verifyStatus` / `confidence` / `issues` | `Custom` (no protocol equivalent — see §3.1) | custom renderer |
| `runSubAgent` nested receipts | `StepStarted` / `StepFinished` nesting | nested step cards |

The gaps in that table are where the actual use cases live.

---

## 2. The use cases, ranked by value-to-cost

### UC-1 — `ChittiAgent extends AbstractAgent` (the enabler)

Not a feature; the precondition for everything else. A pure adapter in
`src/lib/chitti/agui/` that wraps the existing `createSession()` and translates
its callbacks into the AG-UI event stream:

```
onStatus('loading')      → RunStarted
TraceEvent status:running→ StepStarted + ToolCallStart/Args
TraceEvent status:ok     → ToolCallResult + StepFinished
onChart(spec)            → StateDelta  (JSON Patch on /chart)
onFiles(files)           → StateDelta  (/files)
finding text             → TextMessage{Start,Content,End}
citations ledger         → StateDelta  (/citations, append)
AbortedError             → run cancellation
```

**Why it is cheap:** the session already emits every one of these as a
first-class callback. This is a translation layer over an existing seam, it
adds nothing to the shipped bundle, and it is unit-testable in exactly the style
the repo already uses (pure function, direct test). Zero user-visible risk.

**Why it is worth doing even if nothing else lands:** it makes Chitti's agent
consumable by any AG-UI client — including ones that do not exist yet — without
committing to CopilotKit at all.

---

### UC-2 — Human-in-the-loop on the ambiguous series pick ⭐ *highest value*

This one attacks a weakness the codebase documents about itself.

`kb.ts` states plainly that flat scoring "does not merely miss; it is sometimes
confidently wrong," and gives the example of *"how long people live"* resolving
to the **child mortality** series. `fastpath.ts` codifies the only defence
currently available: *"refuse, don't guess"* — `MIN_MATCH_SCORE` deliberately
under-fires, and every rejection falls through to the agent.

So today the app has exactly two moves when a phrase is ambiguous: **guess** or
**refuse**. CopilotKit's `useHumanInTheLoop` supplies the missing third: **ask**.

When `findSeriesWithReceipt` returns candidates inside a scoring margin, the
agent emits a tool call that renders a disambiguation picker and *suspends the
run* until the user chooses:

> "how long people live" could mean:
> - Life expectancy at birth, total (years) — `SP.DYN.LE00.IN`
> - Mortality rate, under-5 (per 1,000) — `SH.DYN.MORT`
> - Survival to age 65, female (%) — `SP.DYN.TO65.FE.ZS`

The `SearchReceipt` — which terms and synonyms fired, how many candidates were
considered — is already carried on the `TraceEvent` and already rendered as a
dedicated card. It is exactly the evidence a picker needs, and it is already
being computed. The scoring margin that gates the picker is the same number
`MIN_MATCH_SCORE` is calibrated against.

**Why this is the strongest case:** it makes the product *better*, not merely
differently plumbed. It also lets `MIN_MATCH_SCORE` stop being the sole arbiter
of a decision it is documented as being bad at — the fast path can widen its
range without widening its risk, because doubt now has somewhere to go.

Same mechanism, three more natural applications:

- **Before an expensive fetch.** Every-country batched World Bank pulls — confirm
  rather than spend.
- **Before `execute_js`.** The sandbox runs model-written JavaScript. A visible
  approval step for code the model wrote is a defensible default for a tool that
  can also make recursive `llm()` calls.
- **At a budget wall.** `MAX_TOOL_CALLS` / `MAX_LLM_PER_TURN` / `MAX_DELEGATIONS_PER_TURN`
  currently stop a turn. "Budget reached — continue?" is a better ending than a
  truncated one.

---

### UC-3 — The dashboard as agent state, not localStorage-then-reconcile

Today: `dashboard.ts` is a versioned document in `localStorage`;
`ui/dashboards-view.ts` privately owns `currentDashId` / `sharedDashState`;
`ui/dash-chat.ts` runs a **dashboard-scoped session** so "now add China" resolves
against the board and not the thread; and `syncDashboardsAfterTurn` reconciles
the view *after* a turn completes.

That is three separate mechanisms — private module state, a scoped session, and
a post-hoc sync — doing what `StateSnapshot` + `StateDelta` do as one primitive.

With the dashboard modelled as AG-UI state:

- **Tiles appear mid-turn.** `addTile` / `renameTile` / `moveTile` /
  `touchTileData` / `markTileStale` each become a JSON Patch delta streamed as it
  happens, instead of a state mutation the view learns about afterwards. That is
  a direct realisation of `dash-chat.ts`'s own stated design goal — *"the answer
  is the tile"* — with the last remaining lag removed.
- **"Now add China" needs no scoped session.** The agent reads the board's
  current state as context, so the referent is the board's live contents rather
  than a conversation history that happens to be scoped to it. The scoping trick
  exists to prevent thread/board cross-contamination; shared state removes the
  problem instead of isolating it.
- **`syncDashboardsAfterTurn` goes away.** Bidirectional state is the protocol's
  job.

`dashboard.ts` is already a "plain, whitelist-rebuildable value type with no
behavior baked in" — which is precisely the shape a JSON-Patch state channel
wants. The design work is done; this is the payoff.

**One thing that must survive:** the whitelist rebuild. See §3.2.

---

### UC-4 — Frontend tools: let the agent operate the app

Chitti's tools all *return data*; the UI reacts to it. `useFrontendTool`
registers browser-side handlers the agent can call directly. Nearly every
candidate is already implemented as a UI function:

| Frontend tool | Already exists as |
| --- | --- |
| `open_dashboard` / `pin_tile` | `ui/dashboards-view.ts` |
| `share_answer` / `copy_markdown` | `ui/actions.ts` (`shareTurn`, `buildFindingOkf`) |
| `export_csv` | `ui/turns.ts` CSV wiring |
| `set_sources` / `switch_provider` / `toggle_rlm` | `ui/config.ts` |
| `browse_catalog` | `catalog.ts` (already pure and offline) |
| `breakdown` / `distribution` | `eda.ts` (already pure, no model) |

This is mostly a registration exercise, and it changes what Chitti *is*: from
"answers questions about data" to "operates the analysis surface." "Chart child
mortality for South Asia, pin it to a new board called *Health*, and set the
sources to World Bank and WHO" becomes one turn.

**Guardrails to keep:** `sourcesLocked` in `ui/config.ts` exists for a reason,
and the key gate must stay ahead of anything that spends. Frontend tools should
be registered *behind* those checks, not around them.

---

### UC-5 — Reasoning and Activity events for the receipt

Chitti's trace is unusually rich for a client-side agent: a plan card from
`InsightBrief`, nested `llm()` children indented under their `execute_js`
parent, per-step tokens, cost, wall-clock duration and serialized data size, a
`derived` provenance flag on model-authored files, and the verify stamp.

Two AG-UI event families map onto it better than tool events do:

- **`ActivitySnapshot` / `ActivityDelta`** for the plan checklist. `matchStepToEvent`
  ticks plan steps off against later tool events — an activity that *updates in
  place* is exactly that, and it replaces re-rendering the trace on each event.
- **`Reasoning*`** for the planner's brief and the verifier's rationale. Both are
  currently smuggled through as synthetic `TraceEvent`s (`tool: 'plan'`,
  `tool: 'verify'`). They are reasoning, not tool calls, and the protocol now
  says so.

Net effect: `ui/trace.ts` (478 lines of bespoke card rendering, including the
nested-receipt cards and the panel summary) gets substantially smaller, and the
trace becomes legible to any AG-UI client.

---

### UC-6 — The fast path as a zero-LLM run

Worth deriving explicitly because it is the thing most likely to be broken by a
careless migration.

`fastpath.ts` answers "indicator × countries × years" with **no model call, no
key, no cost, and nothing capable of hallucinating** — and `handleAskSubmit`
tries it *before* the key gate. In AG-UI terms that is a completely legitimate
run:

```
RunStarted → ToolCall*(find_series) → ToolCall*(fetch_series)
           → StateSnapshot(chart + rows + citations) → RunFinished
```

Zero `TextMessage*` events, zero `Reasoning*` events, no LLM anywhere. AG-UI is
an *agent-to-UI* protocol, not an LLM protocol, so this maps cleanly.

**Use it as the migration's acceptance test.** If the no-key path still answers
instantly and still refuses rather than guesses, the adapter preserved the
property the app cares most about.

---

### UC-7 — Share and restore as snapshot replay

`share.ts` and `dashboard-share.ts` hand-roll: whitelist-encode → JSON →
`deflate-raw` → base64url → `#share=` fragment, with `ui/restore.ts` driving each
renderer by hand on the way back in.

`MessagesSnapshot` + `StateSnapshot` are the protocol's version of "here is the
complete state, render it." A restored permalink becomes *replay two events into
the agent client* rather than a bespoke restore path per renderer.

**This use case carries the sharpest risk in the document — see §3.2. Do not
adopt it before UC-3 has proven the whitelist survives.**

---

### UC-8 — One agent, many surfaces ⭐ *best CopilotKit-specific case*

Once UC-1 exists, `ChittiAgent` is a portable `AbstractAgent`. CopilotKit v2
spans React, Angular, Vue, React Native, Slack and Teams over AG-UI — so the same
agent can power a **chat sidebar on the rest of the site**.

Concretely: the essays at `/work/sae-explorer`, `/work/why-trust-bench`,
`/work/understanding-llms` are static prose. A CopilotKit sidebar backed by the
same agent turns any of them into a surface where a reader can ask a data
question and get a cited, charted answer without leaving the page.

**Why this is the right place to spend the React budget:** it is a *new* surface,
so the island cost is additive rather than a rewrite, and none of `/apps/chitti`'s
deliberate design has to be re-implemented to get it. It is also the only use
case here with a clear reason to reach for CopilotKit's chat UI rather than build
against AG-UI directly.

---

## 3. Costs and things that must not be lost

### 3.1 The verification semantics have no protocol equivalent

`receipts.ts` distinguishes four honest outcomes — `verified`, `unverified`,
`unavailable`, `skipped` — and the code comments are emphatic that the UI must
never present a non-`verified` status as verified, and must never stamp off a
defaulted-true `pass`. `AgentOutput.verification` is `null` on an aborted turn
specifically so an interrupted run cannot earn a badge.

AG-UI has nothing for this. It has to ride on `Custom` events with a renderer
that reproduces the four-state discipline exactly. **This is the app's entire
trust posture and it is stricter than anything the protocol offers.** Any
adoption plan that treats verification as "just another tool result" has already
lost the thing worth keeping.

### 3.2 The whitelist is structural, and `StateSnapshot` is not

`share.ts` and `dashboard.ts` are whitelist-by-construction: they copy only
known-shape fields off whatever object they are handed, so a stray `apiKey` — or
any unlisted field, at dashboard, tile, spec, row or citation level — is
*structurally incapable* of reaching the serialized output.

A naive `StateSnapshot` serializes whatever is in agent state. In a BYOK app
where the key lives in the browser alongside everything else, that is a
credential-exfiltration path into a URL fragment.

**Mitigation:** the state channel must be fed through the existing whitelist
builders, never off the session object directly. Treat this as a hard invariant,
the same way `sw-cache.ts` treats provider hosts as always-`bypass`.

### 3.3 Bundle and offline cost

Current runtime dependencies: Astro + three font packages. CopilotKit adds
React, React-DOM, `@astrojs/react` and the CopilotKit packages to a PWA with a
hand-mirrored service worker (`sw-cache.ts` ↔ `public/apps/chitti/sw.js`) that
must be kept in sync by hand. Every byte added is a byte the offline cache
carries. This is the single strongest argument for keeping `/apps/chitti` on its
existing vanilla UI.

### 3.4 The bespoke UI is not incidental

CopilotKit's chat components would displace `chitti.astro`'s markup and CSS: the
turn template, the ink-stamped VERIFIED badge, the confidence-tinted finding, the
chart↔table hover linking in `ui/charts.ts` ⇄ `ui/evidence.ts`, the search-receipt
card. Re-implementing that inside `useRenderToolCall` renderers is real work with
no user-visible gain.

### 3.5 Test surface

The 690+ test suite is mostly pure functions and the session — an AG-UI adapter
fits that style well and should be tested the same way. But `ui/*` coverage and
the `?chittidebug` / `__chittiDebug` screenshot seam (which harnesses depend on)
would need rework for any surface that moves to React.

---

## 4. Recommendation

**Adopt AG-UI in the library. Adopt CopilotKit only on a new surface.**

| Stage | Work | Risk | Ships to users |
| --- | --- | --- | --- |
| 1 | UC-1 — `ChittiAgent extends AbstractAgent` in `src/lib/chitti/agui/` | none | nothing |
| 2 | UC-2 — HITL disambiguation on ambiguous series | low | the biggest single improvement |
| 3 | UC-3 — dashboard as AG-UI state; retire `syncDashboardsAfterTurn` | medium (§3.2) | tiles appear mid-turn |
| 4 | UC-5 — reasoning/activity events; shrink `ui/trace.ts` | low | richer, standard trace |
| 5 | UC-8 — CopilotKit sidebar on the essays, backed by the same agent | contained | new surface |
| — | UC-7 — share-as-snapshot | high | defer until §3.2 is proven |

Stage 1 is worth doing on its own merits regardless of whether any later stage
lands: it costs nothing at runtime, it is testable in the repo's existing idiom,
and it makes the agent portable.

Stage 2 is the one that makes the product better rather than differently
plumbed, and it is the reason to care about CopilotKit at all — it converts a
weakness the codebase already documents about itself into a UI affordance.

`/apps/chitti` should stay vanilla.
