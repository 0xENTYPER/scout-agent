# Scout for Orbio

> A local treasury and verification layer for agents running through Orbio.

![Scout local QA workspace](assets/product.png)

Scout is a product I am building for the Orbio ecosystem. It helps an operator decide which model an agent may use, how much CREDIT it may spend, what reserve must remain protected, and whether the result is good enough to accept.

The screenshot shows Scout's local QA workspace, the verification module used to turn an agent outcome into repeatable evidence.

- **Status:** active development
- **Product:** private local application
- **Platform:** Orbio model and tool gateway
- **Repository:** public engineering case study, no source code

## What Orbio is

[Orbio](https://www.orbio.so/developers/docs) is infrastructure for products that use AI models, tools, and managed agent resources through one account and balance.

For a product builder, it provides:

- an OpenAI-compatible gateway for model inference;
- access to tools such as web, social, and onchain reads;
- OAuth 2.1 with PKCE so users can connect their own Orbio account;
- balance and usage access for cost-aware products;
- isolated infrastructure permissions for sandboxes, deployments, servers, databases, and inboxes;
- optional application fees paid to the product developer.

Instead of maintaining a separate billing and provider integration for every model, an application can operate against one gateway while the user remains in control of their own Orbio balance and permissions.

## Why Scout exists

Giving an agent access to many models does not answer the operational questions:

- Which route is affordable for this task?
- How much can the agent spend today?
- What balance must remain untouched?
- Did the paid call actually improve the result?
- Can a failed or uncertain request be retried safely?
- What evidence should a human review before approving the outcome?

Scout adds that missing control layer around Orbio. It treats inference as a budgeted operation, not an unlimited chat session.

## Product model

Scout currently combines two connected modules.

### Orbio Agent Treasury Manager

The primary product surface manages an agent's operating budget. It reads the available Orbio CREDIT balance, compares model routes, estimates task cost, protects a reserve, and requires explicit approval before paid execution.

### Scout QA

The verification surface turns a web task into a repeatable browser, HTTP, or file scenario. It preserves the failure evidence, prepares a narrowly bounded repair candidate in an isolated copy, and reruns the original scenario before reporting success.

Together they answer both sides of agent operations: **should this task spend money, and did the result work?**

## Treasury lifecycle

**Connect -> price -> plan -> approve -> execute -> reconcile -> record**

1. The operator connects their own Orbio credential.
2. Scout reads the current balance and the public model catalogue.
3. The operator defines reserve, daily limit, expected workload, and per-task cap.
4. Scout compares eligible model routes against the policy.
5. Planning remains read-only until the operator explicitly enables paid execution.
6. One approved request is sent through the selected Orbio route.
7. Balance is read again and the actual spend is reconciled.
8. The decision and outcome are written to a local ledger.

The application never receives wallet-signing authority. Funding and account ownership remain outside Scout.

## Technical architecture

| Layer | Responsibility |
| --- | --- |
| Local control service | Owns policy evaluation, task planning, execution locks, and run coordination. |
| Browser workspace | Makes balance, runway, route, limits, evidence, and approvals visible. |
| Orbio adapter | Reads balance and model availability, then sends only explicitly approved inference requests. |
| Cost engine | Estimates task cost from expected input/output usage and current route pricing. |
| Policy engine | Enforces protected reserve, daily hard limit, and per-task CREDIT cap. |
| Decision ledger | Records route, estimate, policy decision, balance delta, and outcome without storing private prompts or responses. |
| QA runner | Executes deterministic browser, HTTP, and file checks against local applications. |
| Candidate workspace | Applies a bounded repair away from the original project and verifies source integrity before review. |

The current implementation is a local Python service with a browser-based operator interface and a SQLite-backed policy and decision ledger. The Orbio integration is isolated behind an adapter so pricing, balance, and execution behavior can be tested independently from the UI.

## Budget policy

Scout turns a raw balance into an operating policy:

- **Protected reserve** is never offered to a task.
- **Daily hard limit** caps total paid execution for the day.
- **Per-task cap** rejects a route before a request is sent.
- **Expected workload** converts the remaining budget into visible runway.
- **Route comparison** separates model quality from affordability.
- **Single-flight execution** permits only one paid request at a time.

This makes the budget inspectable before an agent acts and measurable after it finishes.

## Failure model

Paid agent operations need stricter failure handling than an ordinary UI request.

| Condition | Scout behavior |
| --- | --- |
| Missing credential | Planning stays available; paid execution remains disabled. |
| Insufficient balance | The task is blocked before inference. |
| Reserve or cap violation | The route is rejected with the policy reason. |
| Stale or unavailable pricing | Scout refuses to present an estimated route as current. |
| Request already in flight | A second paid execution cannot start. |
| Unknown provider outcome | The event is recorded and never retried automatically. |
| Verification failure | The result remains failed even if inference completed successfully. |

The important distinction is that a successful API response is not the same as a successful product outcome.

## Local security boundaries

- Orbio credentials stay in process memory and are not written to the decision ledger.
- The service binds to loopback and rejects cross-origin control requests.
- A per-process session token protects local control actions.
- Browser verification is restricted to localhost targets.
- Credential-like file paths and selectors are rejected.
- Symlinks are ignored when preparing a candidate workspace.
- Repair candidates have explicit file and project-size limits.
- Source hashes are checked so the original project cannot be silently modified.

## Why the interface is quiet

The UI leads with balance, runway, policy status, selected route, expected cost, and the next approval. Provider metadata and execution logs stay available, but they do not compete with the decision the operator needs to make.

The design follows three rules:

1. **Cost before capability.** A powerful route is irrelevant if it violates the operating budget.
2. **State before action.** The operator sees what is protected and what will change before pressing run.
3. **Evidence before success.** A task is complete only when its original verification scenario passes.

## What works today

- live Orbio balance and model catalogue reads;
- model-route comparison with estimated task cost;
- protected reserve, daily limit, and per-task cap;
- explicit paid-execution gate and single-flight locking;
- before-and-after balance reconciliation;
- local decision ledger without prompt or response storage;
- repeatable browser, HTTP, and file checks;
- saved run history and evidence;
- isolated repair candidates and before/after verification;
- local inference mode for workflows that should not spend CREDIT.

Scout has also been tested against an independent TodoMVC-style application: it detected a broken Enter-key flow, prepared an isolated correction, and passed the same browser scenario after the change.

## Product direction

- Sign in with Orbio for a smoother user-owned balance connection;
- reusable treasury policies for different agent roles;
- shared approval roles for teams;
- clearer estimated-versus-actual spend reporting;
- route performance history based on verified task outcomes;
- CI-triggered verification and review links;
- optional Orbio-native agent economics for funding future work.

## Public scope

This repository documents the real product, architecture, operational model, and current QA interface.

The application source, execution engine, private test data, production configuration, credentials, and internal Orbio account details remain private.

---

Built by [0xENTYPER](https://github.com/0xENTYPER).
