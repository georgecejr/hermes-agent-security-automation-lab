# Hermes Agent — Autonomous AI Red Team & Security Automation Lab

**Multi-Agent Orchestration | Autonomous Security Research | Persistent Workflow Automation | Signal Integration**

**Project status:** Self-hosted Hermes Agent environment configured, operated, and validated on Kali Linux. Persistent Kanban orchestration, custom skill workflows, SQLite event tracing, and Signal completion notifications were demonstrated end to end. This repository is an engineering case study with documentation and session evidence. It is not a packaged application release.

**Repository note:** The GitHub repository name remains `enterprise-shipment-tracking-platform` for historical continuity. The engineering focus of this work is the Hermes automation lab. A package-tracking web exercise appears later as a supporting case study only.

**Evidence posture:** Phone numbers visible in the original captures were covered before these files were committed. Shell prompts still show the lab account name. No email addresses appear in the Hermes screenshots.

**How to read this document:** The lab did not work perfectly on the first attempt. Several capabilities only became reliable after failed tests, SQLite forensics, specification revisions, and retesting. Resolution statuses used below are:

| Status | Meaning |
| --- | --- |
| **Resolved** | Corrective action implemented and success verified |
| **Mitigated** | Workaround or partial solution established |
| **Investigated** | Cause examined; final resolution not confirmed |
| **Outstanding** | Issue remains unresolved or lacks verification |

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Key Technical Achievements](#2-key-technical-achievements)
3. [System Architecture](#3-system-architecture)
4. [Technology Stack](#4-technology-stack)
5. [Infrastructure Setup](#5-infrastructure-setup)
6. [Hermes Agent and Gateway Configuration](#6-hermes-agent-and-gateway-configuration)
7. [Multi-Agent Orchestration](#7-multi-agent-orchestration)
8. [Kanban Task Management](#8-kanban-task-management)
9. [Worker Lifecycle and Execution Monitoring](#9-worker-lifecycle-and-execution-monitoring)
10. [Automated Security Research](#10-automated-security-research)
11. [Custom Skill Engineering](#11-custom-skill-engineering)
12. [Development Workflow and Task Creation](#12-development-workflow-and-task-creation)
13. [Signal Notification Integration](#13-signal-notification-integration)
14. [SQLite Persistence and Event Analysis](#14-sqlite-persistence-and-event-analysis)
15. [Case Study: Security-Aware Web Application Task Orchestration](#15-case-study-security-aware-web-application-task-orchestration)
16. [Engineering Challenges, Root-Cause Analysis & Resolutions](#16-engineering-challenges-root-cause-analysis--resolutions)
17. [Security Architecture](#17-security-architecture)
18. [Testing and Validation](#18-testing-and-validation)
19. [Project Outcomes](#19-project-outcomes)
20. [Reproduction Guide](#20-reproduction-guide)
21. [Future Engineering Extensions](#21-future-engineering-extensions)
22. [Conclusion](#22-conclusion)
23. [Suggested Repository Metadata](#23-suggested-repository-metadata)
24. [Session Evidence Index](#24-session-evidence-index)

---

## 1. Project Overview

### Problem

Traditional AI assistants generally require manual prompting and coordination for each individual task. Complex engineering and security workflows benefit from persistent execution, specialized agent roles, durable task state, and operational visibility that survives across sessions.

### Solution

A self-hosted Hermes Agent environment running inside Kali Linux provides persistent Kanban orchestration, specialized agent profiles, reusable skills, SQLite-backed event history, and integrated messaging. The operator delegates work through the Hermes control plane; workers claim tasks, emit lifecycle events, and surface completion through Signal when subscriptions are configured correctly.

### Implementation

The engineering work covered infrastructure setup, gateway operation, agent execution workflows, security research tasks, reusable skill development, local gateway simulation, Signal notification integration, and validation through SQLite event records and device-side delivery confirmation. Multiple steps failed before they succeeded. The Signal path, dependency waits, and security-review blocks are documented as engineering incidents, not as footnotes.

### Outcome

The lab demonstrated autonomous task execution, persistent workflow management, parent-child dependency handling, custom skill engineering, and independently verified Signal completion notifications. The DHL package-tracking exercise is one supporting demonstration inside that broader automation environment, not the primary product identity of the project.

### Engineering chronology

The documented process follows this sequence. Failures and corrections are part of the path, not exceptions to it:

```text
Initial Goal
  -> Environment Setup (VMware / Kali / Hermes install)
  -> Gateway Configuration
  -> Agent Orchestration
  -> Research Tasks
  -> Skill Development
  -> Kanban Automation
  -> Signal Integration Attempts
  -> Unexpected Failure (no notifications)
  -> Investigation (SQLite cursors / gateway notifier)
  -> Root-Cause Analysis (subscribe-after-complete race)
  -> Corrective Action (subscribe while blocked, then unblock)
  -> Retesting (t_8ef3019d)
  -> Verified Outcome (event/checkpoint 152 + Signal receipt)
  -> Security Review of web-app specs
  -> Dependency orchestration observation
  -> Documentation / screenshot transfer
  -> Project Outcomes
```

That chronology is intentional. The finished capabilities matter, but so do the debugging methodology, timing constraints, and human-in-the-loop controls required to reach them.

---

## 2. Key Technical Achievements

| Achievement | Evidence basis |
| --- | --- |
| Hermes Agent configuration on Kali Linux | Gateway UI, CLI usage, skills paths, and session screenshots |
| Gateway operation | Dashboard reports Gateway Status: Running |
| Autonomous worker execution | Claim, heartbeat, spawn, and completion lifecycle events |
| Persistent Kanban task management | Board columns, task IDs, and SQLite-backed state |
| Parent-child dependency handling | Parent in progress while child remained in Todo |
| Automated SMS gateway research | Completed Kanban tasks for provider and sender-ID research |
| Custom skill development | `sms-custom-sender` skill directory and local test suite |
| Local SMS gateway simulation | Completed simulator task with dedicated workspace |
| Signal notification integration | Subscriptions, notifier modules, and device delivery |
| SQLite event tracing | Event records and notification checkpoint inspection |
| End-to-end notification validation | Task `t_8ef3019d`, event/checkpoint 152, Signal receipt |
| Security-aware requirements revision | Blocked unsafe specs, edited bodies, unblocked parent |

A completed Kanban task establishes the recorded execution outcome for that workflow. It does not automatically prove every possible feature of a generated artifact was independently acceptance-tested outside Hermes.

---

## 3. System Architecture

The lab separates a control plane from task execution, persistence, and operator communication.

```mermaid
flowchart TD
    Operator[Operator]
    Gateway[Hermes Gateway]
    Profiles[Agent Profiles]
    Kanban[Kanban Task Engine]
    SQLite[(SQLite Database)]
    Worker[Worker Execution]
    Skills[Custom Skills]
    Watcher[Notification Watcher]
    SignalCLI[Signal CLI]
    Recipient[Signal Recipient]

    Operator --> Gateway
    Gateway --> Profiles
    Gateway --> Kanban
    Kanban --> SQLite
    Gateway --> Worker
    Worker --> Skills
    Gateway --> Watcher
    Watcher --> SignalCLI
    SignalCLI --> Recipient
```

### Control plane

The Hermes Gateway hosts the local dashboard, plugin surfaces, profile configuration, and orchestration controls. Observed session evidence shows the Kanban plugin active, gateway status running, and orchestration set to Auto when workers were in flight.

### Task execution

Workers claim Kanban tasks, bind to workspaces, emit heartbeat and lifecycle events, and execute against assigned agent profiles. Specialized profiles such as `default` and `dhlpackagetracker` demonstrate role-based ownership.

### Persistence

Task state, lifecycle events, notification subscriptions, and notification cursors are recorded in SQLite (`~/.hermes/kanban.db` in the observed environment). Event IDs and checkpoints are the primary audit trail for notification debugging — and were the decisive evidence when Signal messages failed to appear.

### Communication

The gateway Kanban notifier watches eligible terminal events and forwards messages through the Signal adapter / `signal-cli` path to the operator device. Early tests proved that transport availability alone is not enough; subscription timing relative to event cursors determines whether a completion is ever delivered.

---

## 4. Technology Stack

| Layer | Technology | Classification |
| --- | --- | --- |
| Host OS | Windows host running VMware Workstation | Observed |
| Lab OS | Kali Linux virtual machine | Observed |
| Agent runtime | Hermes Agent v0.23.1 (Nous Research) | Observed |
| Languages / tooling | Python, Bash, Git | Observed |
| Persistence | SQLite | Verified via event inspection |
| Messaging | Signal CLI / Signal adapter | End-to-end verified |
| Service model | User-level Hermes gateway; systemd-compatible service patterns | Observed / applicable |
| Documentation workflow | Host/guest screenshot transfer and repository documentation sessions | Observed |
| Proposed demo app stack | Web UI, server-side tracking proxy, mock mode, admin dashboard | Specified for the case study; not independently verified as deployed |

Observed technology is what the lab actually ran. Proposed application components belong only to the later web-security case study and are not claimed as shipped product code in this repository.

---

## 5. Infrastructure Setup

| Component | Observed configuration |
| --- | --- |
| Windows host | VMware Workstation host for the Kali guest |
| Kali VM | Lab workstation used for Hermes installation and operation |
| Hermes home | `~/.hermes/` for config, skills, and Kanban database paths |
| Project workspaces | `~/hermes-workspaces/` for persistent task directories |
| Gateway UI | Local dashboard at `http://127.0.0.1:9119` |
| Lab account | Shell prompts show `cybergeorge` |

Representative directories used during the session:

```text
~/.hermes/
~/.hermes/skills/productivity/sms-custom-sender/
~/hermes-workspaces/sms-gateway/
~/hermes-workspaces/dhl-tracking-lab/
```

### Journey checkpoint: getting the lab online

**Original objective:** Stand up a durable Hermes environment on Kali so autonomous workers could run without depending on a single interactive chat session.

**Initial approach:** Install Hermes on the Kali guest, configure the gateway, and keep the control plane reachable on localhost.

**Obstacles encountered along the way:**

- Host/guest separation meant configuration lived in the VM while documentation and screenshot staging often happened on Windows.
- Persistent service behavior had to be confirmed operationally (dashboard Gateway Status: Running), not assumed from install completion alone.
- Later in the session, VMware reported host disk exhaustion against a Kali virtual disk segment, threatening continued execution. See [Incident D](#incident-d--vmware-storage-exhaustion).

**Engineering lesson:** Infrastructure success is not only “software installed.” It includes persistent gateway health, workspace path discipline, and host resource headroom.

Shared-folder transfer between Kali and Windows was later required to gather screenshot evidence for documentation. See [Incident E](#incident-e--cross-platform-screenshot-management).

---

## 6. Hermes Agent and Gateway Configuration

### Persistent gateway operation

The Hermes gateway remained available during orchestration sessions. Dashboard captures show Gateway Status: Running, with active session counts changing as workers executed.

### Agent profiles

Profiles define worker behavior and ownership. The `default` profile description visible in the dashboard references offensive security testing and reverse-engineering style work. Additional profiles such as `dhlpackagetracker` were used for specialized task assignment.

### Worker processes and monitoring

Workers are spawned for dispatchable tasks, emit lifecycle events, and can be inspected through the Kanban UI and CLI (`hermes kanban show`, `hermes kanban watch`).

### Gateway responsibilities

Observed responsibilities include:

- Serving the local control dashboard
- Hosting Kanban orchestration and dispatch controls
- Loading watchers/notifiers for terminal task events
- Integrating channel adapters such as Signal
- Preserving configuration in `~/.hermes/config.yaml`

Kanban configuration observed during investigation included `review_dispatch: true`.

### Journey checkpoint: gateway and notifier discovery

When Signal delivery failed despite completed tasks, the next hypothesis was “gateway notifier misconfiguration.” Source search under the Hermes gateway tree located the Kanban watcher/notifier modules and confirmed the local `kanban:` config block. That investigation is captured in Figure 5 and later proved the transport path existed — the defect was cursor timing, not a missing gateway process.

**Status for gateway operation itself:** Resolved (observed running during orchestration).  
**Status for first-pass Signal automation:** Failed initially; later Resolved after cursor-aware retesting (see Section 13 and Incident A).

---

## 7. Multi-Agent Orchestration

Hermes coordinates specialized roles rather than forcing every task through a single undifferentiated chat session.

| Mechanism | Observed behavior |
| --- | --- |
| Specialized roles | `default`, `dhlpackagetracker`, and researcher board context |
| Task delegation | Operator creates or revises tasks; workers claim assigned work |
| Worker assignment | Tasks can be assigned to profiles before dispatch |
| Dependencies | Child tasks remain waiting while parents are unfinished |
| Execution scheduling | Auto orchestration and dispatcher nudge controls |
| Task ownership | Assignee and model/profile fields visible on task cards |

### Journey checkpoint: dispatch vs. dependency waits

**Original objective:** Allow specialized workers to execute related work without manual babysitting of every handoff.

**Obstacle encountered:** A child task remained in Todo while a parent was In Progress. At first glance this looked like a stuck dispatcher.

**Technical evidence:** Board captures show parent `t_5025728e` In Progress and child `t_cd335286` Todo with dependency semantics visible in the UI (Figures 8 and 9).

**Root cause:** Expected dependency behavior, not a failed dispatch loop.

**Engineering lesson:** Multi-agent orchestration requires reading dependency state before declaring a worker or scheduler failure.

---

## 8. Kanban Task Management

Hermes Kanban provides durable task creation, assignment, dispatch, execution, blocked states, completion, and recovery. Board columns observed during the session include Triage, Todo, Scheduled, Ready, In Progress, Blocked, Review, and Done.

### Completed research and notification workflows

| Task ID | Workflow | Observed status |
| --- | --- | --- |
| `t_440451bd` | SMS gateway research | Completed |
| `t_0022291f` | Custom SMS gateway skill development | Completed |
| `t_2d59e102` | Alphanumeric sender ID research | Completed |
| `t_53e53c1e` | Local SMS gateway simulator | Completed |
| `t_f13e50d6` | Signal notification test | Completed |
| `t_a23f00c8` | Signal notification test 2 | Completed |
| `t_8ef3019d` | Controlled Signal integration validation | Completed |

### Supporting web-application orchestration tasks

| Task ID | Role | Last observed state in session evidence |
| --- | --- | --- |
| `t_5025728e` | Parent coordination task | Running / in progress after unblock |
| `t_cd335286` | Child implementation task | Todo, waiting on parent |

### Task lifecycle

```mermaid
stateDiagram-v2
    [*] --> Created
    Created --> Assigned
    Assigned --> Ready
    Ready --> Dispatched
    Dispatched --> InProgress
    InProgress --> Blocked
    Blocked --> Ready: operator unblock / revision
    InProgress --> Completed
    Completed --> [*]
    Ready --> Todo: waiting on dependency
    Todo --> Ready: dependency satisfied
```

Typical operator flow:

```bash
hermes kanban create --title "..." --assignee default --workspace dir:/path --body "..."
hermes kanban edit TASK_ID --body "..."
hermes kanban unblock TASK_ID
hermes kanban dispatch
hermes kanban show TASK_ID
hermes kanban list
hermes kanban watch
```

### Journey checkpoint: blocked work is useful, not only failures

Early board state showed a Ready tracking card beside a skill task blocked for human review. Blocking preserved progress while forcing operator decisions about signup verification, numbering constraints, and later security scope.

![Hermes Kanban dashboard showing a Ready DHL tracking card and a blocked SMS skill review task](docs/screenshots/01-kanban-dhl-task-and-blocked-review.png)

*Figure 1. Hermes Kanban board with an unassigned Ready task and a blocked skill-development task under review.*

The dashboard demonstrates persistent task state across Ready and Blocked columns, profile-aware assignment, Signal notification channel selection on the task inspector, and human-in-the-loop controls for triage, unblock, complete, and archive actions.

**Important accuracy note about completed Signal tests:** `t_f13e50d6` and `t_a23f00c8` completed as Kanban tasks but did **not** produce operator Signal messages. Kanban completion and notification delivery are separate success criteria. Only the controlled retest `t_8ef3019d` verified end-to-end Signal delivery.

---

## 9. Worker Lifecycle and Execution Monitoring

Worker observability is recorded both in the Kanban inspector and in SQLite task events. Typical lifecycle markers include created, assigned, spawned, worker_bound, heartbeat, dependency_wait, and completion/exit summaries.

### Journey checkpoint: autonomous workers still need humans

**Original objective:** Let a worker build and validate an SMS custom-sender skill with minimal interruption.

**Obstacle encountered:** The worker reached a point where live alphanumeric sender registration required human-held resources (signup verification, suitable non-US number, brand name). The task moved to Blocked rather than silently inventing credentials.

**Technical evidence:** Worker log and final-result text request operator action; lifecycle events show claim/heartbeat/dependency wait behavior.

![Blocked Hermes task worker log showing claim, heartbeat, and dependency-related blocking](docs/screenshots/02-blocked-task-worker-log.png)

*Figure 2. Task inspector worker log for a blocked SMS custom-sender skill task.*

The log demonstrates task claims, heartbeat activity, dependency wait behavior, and an explicit human-action request when alphanumeric sender registration could not be completed from a headless worker alone.

**Status:** Mitigated for skill construction (skill built/on disk and later local tests passed); live gateway signup remained human-gated.

### Autonomous worker execution

After requirements revision and unblock, the parent coordination task was observed in the In Progress column while previously completed Signal tests remained in Done.

![Hermes Kanban displaying an active parent task in progress with completed Signal tests in Done](docs/screenshots/08-parent-task-in-progress.png)

*Figure 8. Hermes Kanban displaying an active agent execution workflow for the parent coordination task.*

The board shows persistent in-flight ownership, Auto orchestration, gateway status running, and a Done column containing the completed research and Signal validation tasks.

### Dependency-aware task orchestration

The child implementation task remained in Todo while the parent was unfinished.

![Child Kanban task waiting in Todo while the parent task remains in progress](docs/screenshots/09-child-task-waiting-on-parent.png)

*Figure 9. Child task retained in Todo because its parent dependency had not completed.*

This state is expected dependency behavior, not a dispatcher failure. It shows Hermes preserving execution order across specialized assignees (`@default` for the parent and `@dhlpackagetracker` for the child).

**Status:** Resolved as correct dependency semantics (investigated and confirmed; no defective dispatcher fix required).

---

## 10. Automated Security Research

Hermes was used to automate research tasks related to SMS gateway providers and alphanumeric sender ID constraints.

Observed completed research outcomes include:

- Provider research for gateways that support custom or alphanumeric sender names
- Investigation of verification and numbering constraints, including the practical issue that US numbers often cannot receive alphanumeric sender IDs
- Documentation of human dependencies such as signup verification and brand naming before live send capabilities can be enabled

### Journey checkpoint: research exposed hard constraints

**Original objective:** Identify a workable SMS gateway path for custom sender names.

**Unexpected finding:** Criterion combinations involving US numbers and alphanumeric sender IDs were not simultaneously satisfiable on the researched gateways. The agent recorded that constraint instead of claiming a false success.

**Engineering lesson:** Autonomous research is valuable when it surfaces impossibility conditions early and forces scope revision before paid live sends.

These tasks exercised autonomous research execution under Kanban persistence, not a claim of production SMS infrastructure ownership.

---

## 11. Custom Skill Engineering

Reusable capabilities are packaged as Hermes skills under `~/.hermes/skills/`. The session focused on a productivity skill for SMS custom sender workflows and a follow-on local simulator that avoided paid live sends during development.

![Terminal listing the Hermes sms-custom-sender skills directory path](docs/screenshots/03-hermes-skills-directory.png)

*Figure 3. Operator preparing to inspect `~/.hermes/skills/productivity/sms-custom-sender/`.*

The skills directory is the reusable capability layer for agents. Skills can be tested locally, referenced by later Kanban tasks, and iterated without rebuilding the entire gateway stack.

### Journey checkpoint: local simulation after live-send blockers

Because live alphanumeric registration required human-held resources, the workflow shifted to a free local SMS gateway simulator that preserved skill integration without paid API traffic. That adaptation is a deliberate mitigation path, not a claim that live sender-ID onboarding completed.

The custom skill workflow included local Python tests covering dry-run behavior, gateway response handling, key resolution, and sender validation constraints.

---

## 12. Development Workflow and Task Creation

Local validation and Kanban task creation were part of the same operator loop: prove a skill locally, then delegate the next engineering slice to Hermes.

![Local SMS skill tests passing followed by hermes kanban create for a local gateway simulator](docs/screenshots/04-local-tests-and-kanban-create.png)

*Figure 4. Local skill test activity and creation of a Kanban task for a free local SMS gateway simulator.*

This screenshot validates local skill testing and Hermes task creation. It does not validate a DHL web application deployment or acceptance suite.

### Journey checkpoint: documentation and IDE-assisted workflow

Engineering sessions used the Kali Hermes CLI/UI for runtime work and separate documentation passes to capture evidence, redact identifiers, and publish the engineering record. That split workflow later required careful screenshot transfer between Kali and Windows ([Incident E](#incident-e--cross-platform-screenshot-management)). Documentation tooling assisted narrative assembly; runtime truth still came from Hermes CLI output, SQLite inspection, Signal receipt, and the preserved screenshots.

---

## 13. Signal Notification Integration

Signal integration is one of the strongest verified engineering outcomes in this lab — and also the clearest example that first attempts failed.

### 13.1 Signal notification architecture

```mermaid
flowchart TD
    Completion[Task completion]
    Events[(SQLite task_events)]
    Notifier[Hermes Kanban notifier]
    Adapter[Signal adapter / signal-cli]
    Device[Operator Signal device]

    Completion --> Events
    Events --> Notifier
    Notifier --> Adapter
    Adapter --> Device
```

Pipeline components:

1. Signal CLI configuration and linked device transport
2. Local HTTP daemon / adapter path used by Hermes
3. Hermes Gateway integration
4. Kanban notification subscriptions
5. Notification watcher operation
6. SQLite task events
7. Notification cursor / checkpoint behavior

![Gateway source search for Kanban notifier modules and kanban config inspection](docs/screenshots/05-gateway-kanban-notifier.png)

*Figure 5. Investigation of Hermes gateway Kanban watcher/notifier modules and local `kanban` configuration.*

The capture shows gateway modules such as `kanban_watchers.py`, `kanban_watchers_notifier.py`, and `kanban_watchers_dispatcher.py`, along with `review_dispatch: true` in configuration. This is architectural evidence for the notification path, not by itself proof of message delivery.

### 13.2 Full debugging cycle: from silent completions to verified delivery

#### 1. Original objective

Notify the operator over Signal when a Kanban task reached a terminal completion state, without manually polling the board.

#### 2. Initial approach

1. Configure Signal CLI / Hermes Signal adapter path.
2. Create Kanban tasks intended as notification tests (`t_f13e50d6`, `t_a23f00c8`).
3. Allow workers to complete those tasks.
4. Expect the gateway Kanban notifier to deliver completion messages.

#### 3. Obstacle encountered

The test tasks completed successfully in Kanban, but no completion messages arrived on the Signal device. From the board alone, the system looked healthy. From the operator’s phone, automation looked broken.

#### 4. Technical evidence

- Kanban listed the early Signal tests as `done`.
- Device-side receipt for those early tests did not occur.
- Later subscription listing showed Signal subscriptions associated with task IDs after work had already finished.

![Hermes kanban list and Signal notify-subscribe confirmations with chat identifiers covered](docs/screenshots/06-signal-subscription-redacted.png)

*Figure 6. Task list and Signal notification subscriptions registered against Kanban task IDs.*

The listing shows completed research and Signal test tasks alongside the later web-application cards. Chat identifiers are redacted in the committed screenshot. This figure is evidence of subscription management activity; it is also part of the timing story because subscriptions registered after completion cannot replay history.

#### 5. Investigation

Diagnostic work combined code-path discovery and database forensics:

1. Search gateway sources for Kanban watcher/notifier entry points (`Kanban_watchers`, `KanbanNotify`, `notify_watcher` patterns).
2. Inspect `~/.hermes/config.yaml` for the `kanban:` block (`review_dispatch: true` observed).
3. Inspect SQLite task events for completion event IDs.
4. Compare notification subscriptions and notification cursors against those event IDs.
5. Determine whether the notifier could ever have observed the terminal events.

```bash
rg -n 'Kanban_watchers|KanbanNotify|notify_watcher' ~/.hermes/Hermes-agent/gateway
grep -n -A 15 '^kanban:' ~/.hermes/config.yaml
hermes kanban list
hermes kanban notify-list
```

SQLite inspection focused on task events, subscription records, and event/notification checkpoints in `~/.hermes/kanban.db`.

#### 6. Root cause

Subscriptions had been created **after** the relevant completion events. Notification cursors therefore started beyond those terminal events. The watcher had nothing eligible to deliver for already-finished work.

**Important operational rule:** Subscribing to an already-completed task does not replay historical completion events.

This was not primarily a Signal CLI outage and not primarily a missing gateway process. It was an event-watermark ordering defect in the automation workflow.

#### 7. Solution

Force subscription-before-completion ordering:

1. Create a validation task with `--initial-status blocked`.
2. Register the Signal subscription while the task is still blocked.
3. Unblock and dispatch so completion happens after the subscription exists.
4. Compare completion event ID against notification checkpoint after execution.

#### 8. Verification

Task ID: `t_8ef3019d`

Controlled procedure:

1. Create the task with `--initial-status blocked`
2. Register the Signal subscription while the task is still blocked
3. Unblock the task and allow Hermes to execute it
4. Confirm worker completion and expected test artifact write
5. Verify SQLite completion event **152**
6. Verify last event checkpoint **152** and last notification checkpoint **152**
7. Confirm the completion message was received on the Signal device

| Checkpoint | Value |
| --- | --- |
| Completion event | 152 |
| Last event checkpoint | 152 |
| Last notification checkpoint | 152 |
| Delivery | Confirmed received through Signal |

Matching event and notification checkpoints is the key validation signal: the notifier processed through the completion event rather than skipping past it.

**Status:** Resolved.

#### 9. Engineering lesson

In event-driven agent systems, “task completed” and “operator notified” are different predicates. Durable success requires proving:

- the terminal event exists,
- the subscription existed before that event,
- the notifier cursor advanced through that event,
- and the external transport delivered the message.

### Operational commands

```bash
hermes kanban notify-subscribe TASK_ID \
  --platform signal \
  --chat-id YOUR_SIGNAL_NUMBER

hermes kanban notify-list
hermes kanban watch
hermes kanban list
```

---

## 14. SQLite Persistence and Event Analysis

SQLite is the lab's system of record for orchestration forensics.

| Record type | Role |
| --- | --- |
| Tasks | Durable Kanban cards, assignees, statuses, dependencies |
| Task events | Append-only lifecycle history used for audit and notification eligibility |
| Notification subscriptions | Platform/chat bindings for specific task IDs |
| Event / notification cursors | Watermarks controlling which events have been considered for delivery |

### Journey checkpoint: the database told the truth the UI softened

Kanban completion status alone suggested the early Signal tests had succeeded. SQLite event/cursor comparison showed why the operator still received nothing. That distinction is why the lab treats database inspection as a first-class debugging skill rather than an optional admin detail.

Diagnostic pattern used during Signal troubleshooting:

1. Confirm task completion status in Kanban
2. Inspect the completion event ID in SQLite
3. Compare subscription creation time / cursor against that event ID
4. Determine whether the notifier could ever have seen the terminal event
5. Re-test with subscription-before-completion ordering

This event-centric approach converted an apparently broken notification channel into a timing/cursor defect with a reproducible fix.

---

## 15. Case Study: Security-Aware Web Application Task Orchestration

The package-tracking exercise demonstrated agent task decomposition, parent-child dependencies, requirements analysis, security review, human intervention, revised implementation specifications, worker dispatch, and persistent workspace planning. It is a supporting case study inside the Hermes lab, not the primary identity of the repository.

### Observed task pair

| Task ID | Role | Notes |
| --- | --- | --- |
| `t_5025728e` | Parent | Coordinate delivery of a package-tracking application workflow |
| `t_cd335286` | Child | Implement the tracking demo; assigned to `dhlpackagetracker` |

### Journey checkpoint: security restrictions blocked the first specification

#### 1. Original objective

Use Hermes to orchestrate implementation of a package-tracking web demonstration with optional location features and an admin view.

#### 2. Initial approach

Create parent/child Kanban tasks and allow autonomous workers to implement from the original task bodies.

#### 3. Obstacle encountered

The original specification proposed a publicly exposed look-alike experience that would collect visitor information without sufficient disclosure, require GPS permission to show tracking information, and expose an administrative log page with a hardcoded password. The agent flagged these requirements and blocked progress pending review.

#### 4–6. Evidence, investigation, and root cause

Task bodies and blocked state made the conflict explicit: the requested behavior violated consent, transparency, and credential-handling expectations for an authorized demonstration. This was a policy/requirements failure, not a compiler error.

#### 7. Solution

Revise the parent and child bodies, then unblock:

```bash
hermes kanban edit t_5025728e \
  --body "Build and test a functional package-tracking web application using DHL's official Shipment Tracking API, with a mock mode for demonstrations. Include a responsive tracking interface, shipment history, optional consent-based location sharing, and a secure administrative dashboard. Preserve project files in /home/cybergeorge/hermes-workspaces/dhl-tracking-lab. Coordinate implementation with child task t_cd335286."

hermes kanban edit t_cd335286 \
  --body "Implement a clearly labeled package-tracking demo with optional DHL API integration. Display realistic simulated shipment information when no API key is configured. Request GPS location only through explicit informed consent; tracking must work without GPS. Collect only necessary diagnostic information with disclosure and consent. Build a secure authenticated admin dashboard with environment-configured credentials, location maps for consented records, and no hardcoded passwords. Test locally before any public deployment. Use Cloudflare Quick Tunnel only for an explicitly approved demo deployment."

hermes kanban unblock t_5025728e
hermes kanban dispatch
```

![Revised Hermes task bodies saved, followed by unblock of the parent task](docs/screenshots/07-revised-specifications-and-unblock.png)

*Figure 7. Operator revising parent and child task specifications, then unblocking the parent for resumed execution.*

#### 8. Verification

Edit/unblock CLI output was captured. The parent was later observed In Progress under the revised scope. The child remained Todo waiting on the parent.

#### 9. Engineering lesson

Autonomous implementation without a security gate can encode unsafe product behavior quickly. Human-in-the-loop blocking turned a dangerous specification into a consent-based design before more work accumulated on the wrong target.

**Status:** Resolved for orchestration/security-review controls. Application implementation, deployment, and acceptance testing remain Outstanding / not independently verified.

### Consent-based design requirements

The revised child-task scope called for a clearly labeled package-tracking demo, optional API integration, simulated shipment data when no API key is configured, explicit informed consent for location sharing, and a secure authenticated admin dashboard.

### What this repository does not claim

This repository does not contain evidence of:

- A production DHL integration
- Public deployment of the demo application
- Successful application acceptance testing
- A complete application source tree committed here

The verified claim is orchestration and security-review behavior around those tasks, plus the broader Hermes automation outcomes documented above.

---

## 16. Engineering Challenges, Root-Cause Analysis & Resolutions

This section consolidates the major obstacles. Individual incidents are also woven into the chronological sections above so the reader can see when they appeared in the real workflow.

### Incident summary

| ID | Challenge | Status |
| --- | --- | --- |
| A | Missing Signal notifications despite completed tasks | **Resolved** |
| B | Child task appeared stuck in Todo during parent execution | **Resolved** (correct dependency behavior) |
| C | Unsafe web-app specification blocked by security review | **Resolved** for review/revision controls; app delivery **Outstanding** |
| D | VMware host disk exhaustion against Kali VMDK segment | **Investigated**; recovery recommended, not verified |
| E | Screenshot evidence split across Kali guest and Windows host | **Resolved** for documentation asset handling |
| F | Live alphanumeric SMS sender onboarding from a headless worker | **Mitigated** via local simulator + human-gated signup requirements |
| G | Distinguishing Kanban completion from external notification success | **Resolved** by checkpoint-based validation on `t_8ef3019d` |

### Incident A — Missing Signal Notifications

| Stage | Detail |
| --- | --- |
| Original objective | Deliver Signal messages on Kanban task completion |
| Initial approach | Run notification test tasks and rely on gateway notifier after completion |
| Obstacle | Tasks completed; device received no messages |
| Technical evidence | Done-state Kanban cards; absent device receipt; later subscription listing |
| Investigation | Gateway notifier source search, config inspection, SQLite event/cursor comparison |
| Root cause | Subscriptions created after completion events; cursors started beyond eligible events |
| Solution | Create blocked task → subscribe → unblock/dispatch |
| Verification | `t_8ef3019d`; completion event 152; event/notification checkpoints 152; Signal receipt confirmed |
| Engineering lesson | Event watermarks dominate notification correctness |
| Status | **Resolved** |

Related evidence: Figures 5 and 6; Section 13 debugging cycle.

### Incident B — Kanban Dependency Blocking

| Stage | Detail |
| --- | --- |
| Original objective | Run parent/child implementation workflow with specialized assignees |
| Initial approach | Dispatch dependent cards under Auto orchestration |
| Obstacle | Child remained in Todo while parent was active |
| Technical evidence | Figures 8 and 9 board state |
| Investigation | Compared statuses, assignees, and dependency fields |
| Root cause | Parent unfinished; child correctly waiting |
| Solution | Continue parent execution; do not force-break dependency order |
| Verification | Board captures show expected parent/child relationship |
| Engineering lesson | Dependency waits can look like scheduler bugs |
| Status | **Resolved** (behavior confirmed correct) |

### Incident C — Security Review and Task Blocking

| Stage | Detail |
| --- | --- |
| Original objective | Orchestrate a package-tracking demonstration application |
| Initial approach | Autonomous implementation from the original task bodies |
| Obstacle | Agent blocked on high-risk collection/access requirements |
| Technical evidence | Blocked review state; original vs revised bodies; Figure 7 |
| Investigation | Operator reviewed task text and security implications |
| Root cause | Spec requested insufficient disclosure, coercive location behavior, insecure admin access |
| Solution | Rewrite bodies for consent-based design and env-configured credentials; unblock |
| Verification | Edit/unblock captured; parent later In Progress under revised scope |
| Engineering lesson | Human-in-the-loop gates are part of secure agent operations |
| Status | **Resolved** for controls; application delivery **Outstanding** |

### Incident D — VMware Storage Exhaustion

| Stage | Detail |
| --- | --- |
| Original objective | Keep the Kali lab available for continued Hermes execution |
| Initial approach | Continue normal VM operation on the Windows host |
| Obstacle | VMware reported insufficient free space for a Kali virtual disk segment |
| Technical evidence | Host-side error referencing a path such as `...\kalinov26\kalinov26-000001-s030.vmdk` |
| Investigation | Identified host filesystem capacity exhaustion rather than a Hermes application fault |
| Root cause | Windows host disk space exhaustion |
| Solution recommended | Keep the VMware dialog open, free host space safely, retry; do not manually delete snapshot-chain `.vmdk`/metadata files |
| Verification | Successful VM recovery was recommended but **not verified** in the available record |
| Engineering lesson | Agent labs inherit host virtualization constraints |
| Status | **Investigated** |

Useful host-side checks:

```powershell
Get-PSDrive C
Start-Process "ms-settings:storagesense"
```

### Incident E — Cross-Platform Screenshot Management

| Stage | Detail |
| --- | --- |
| Original objective | Preserve authentic session evidence in the repository |
| Initial approach | Capture screenshots during Kali sessions; stage docs from Windows |
| Obstacle | Evidence lived across guest/host boundaries |
| Technical evidence | Captures under the Windows Pictures/KaliScreenshots workflow; nine committed images |
| Investigation | Located transferable captures and verified redactions |
| Root cause | Split host/guest documentation workflow |
| Solution | Transfer via VMware shared folders, redact phone numbers, commit under `docs/screenshots/` |
| Verification | All nine screenshots present at stable relative paths and embedded in technical sections |
| Engineering lesson | Evidence handling is part of engineering delivery, not an afterthought |
| Status | **Resolved** |

### Incident F — Live SMS Sender Constraints

| Stage | Detail |
| --- | --- |
| Original objective | Support custom/alphanumeric sender workflows |
| Initial approach | Research providers and build `sms-custom-sender` skill for live use |
| Obstacle | Headless worker could not complete signup verification / suitable numbering / brand requirements; US-number + alphanumeric constraints conflicted |
| Technical evidence | Figures 1–4; blocked worker result text; local test suite and simulator task |
| Investigation | Provider research tasks + worker final-result analysis |
| Root cause | External identity/verification constraints outside autonomous worker control |
| Solution | Keep skill + local simulator path; leave live signup human-gated |
| Verification | Local tests observed passing; simulator task completed; live onboarding not claimed complete |
| Engineering lesson | Autonomous systems need explicit human gates for account/identity actions |
| Status | **Mitigated** |

### Incident G — False Confidence from Kanban “Done”

| Stage | Detail |
| --- | --- |
| Original objective | Treat completed notification tests as proof of Signal automation |
| Initial approach | Rely on Done-column status for `t_f13e50d6` / `t_a23f00c8` |
| Obstacle | Done status coexisted with zero device notifications |
| Technical evidence | Completed cards vs absent Signal receipt; later checkpoint comparison on `t_8ef3019d` |
| Investigation | Separated Kanban completion from notifier cursor advancement and transport receipt |
| Root cause | Success criteria were incomplete |
| Solution | Require event ID + notification checkpoint + device receipt |
| Verification | Event/checkpoint 152 match and confirmed Signal message |
| Engineering lesson | Define multi-layer acceptance checks for event-driven integrations |
| Status | **Resolved** |

---

## 17. Security Architecture

| Control area | Lab practice / requirement |
| --- | --- |
| Isolation | Hermes lab runs in a Kali VM on a Windows host; gateway listened on localhost in observed sessions |
| Authorization | Human unblock/review gates for blocked work; profile-based task ownership |
| Credential handling | No secrets committed; Signal chat IDs redacted in screenshots; application credentials required to stay server-side/env-based in the case study |
| Auditability | SQLite task events and notification checkpoints provide reconstructable history |
| Privacy | Phone numbers covered before commit; minimize personal data in logs and screenshots |
| Human oversight | Agents can block for signup, policy, or unsafe-spec reasons; operators revise and release work deliberately |

Out of scope for this project identity: impersonating commercial brands in production, covert fingerprinting, coercive geolocation prompts, publicly exposing sensitive logs, or shipping hardcoded administrative credentials.

---

## 18. Testing and Validation

### Evidence-based validation matrix

| Component | Validation | Result |
| --- | --- | --- |
| Hermes Gateway | Service/dashboard operation | Verified |
| Kanban Engine | Task creation, assignment, and dispatch | Verified |
| Worker Execution | Lifecycle events and task completion | Verified |
| SQLite Persistence | Event records and checkpoint inspection | Verified |
| Custom Skill Workflow | Local tests and completed skill/simulator tasks | Verified |
| Signal Transport | Direct messaging path available | Verified |
| Signal Automation | Completion notification delivered for `t_8ef3019d` | Verified |
| Parent-Child Dependencies | Dependency state observed on board | Verified |
| Human-in-the-loop controls | Spec revision and unblock captured | Verified |
| Early Signal test tasks | Kanban completion only | Incomplete as notification proofs |
| Live alphanumeric SMS onboarding | Account signup / production send | Not verified |
| VMware recovery after disk exhaustion | Successful resumed VM operation | Not independently verified |
| DHL / tracking application | Deployment and acceptance testing | Not independently verified |

### Verified engineering checks

- [x] Hermes Kanban tasks can be created, assigned, and dispatched
- [x] Task lifecycle events are persisted in SQLite
- [x] Blocked tasks can be updated and released for execution
- [x] Worker execution and completion can be observed through the CLI/UI
- [x] Signal subscriptions can be registered for task events
- [x] A controlled task completion produced a Signal notification received by the operator
- [x] Parent-child dependency ordering was observed on the board
- [x] Unsafe initial application requirements were blocked and revised
- [x] Early notification failures were explained via event-cursor analysis

### Application acceptance criteria — pending evidence

- [ ] Tracking interface renders correctly on desktop and mobile
- [ ] Valid tracking numbers return appropriately formatted shipment data
- [ ] Mock mode works without a DHL API key
- [ ] Live API mode keeps provider credentials server-side
- [ ] Consent and deletion flows are verified
- [ ] Administrative access controls are tested
- [ ] Public or tunnel deployment is approved and verified

Do not treat the pending application checklist as failed Hermes validation. It is a separate product-acceptance surface that this documentation repository does not claim to have closed.

---

## 19. Project Outcomes

What was actually demonstrated:

- Self-hosted autonomous AI infrastructure on Kali Linux using Hermes Agent
- Multi-agent task ownership through profiles and Kanban assignment
- Persistent workflow orchestration with blocked, dependent, and completed states
- Security research automation for SMS gateway and alphanumeric sender constraints
- Custom skill engineering with local tests and a simulator follow-on task
- Event-driven Signal notifications with cursor-aware validation after an initial failure
- SQLite-backed operational observability and incident analysis
- Human-in-the-loop security review before resuming a high-risk application specification
- Infrastructure troubleshooting across VMware storage and cross-platform evidence handling

What was not demonstrated in this repository:

- A shipped tracking application binary or complete source tree
- Production DHL API operation
- Verified public deployment
- Application-level acceptance test completion
- Verified recovery from the VMware disk-exhaustion incident
- Completed live alphanumeric sender onboarding without human-held account resources

The enduring engineering value is the progression itself:

**Initial Goal → Implementation → Unexpected Failure → Investigation → Root-Cause Analysis → Corrective Action → Retesting → Verified Outcome → Engineering Lessons**

---

## 20. Reproduction Guide

This guide reproduces the Hermes and Signal workflow pattern validated in the lab, including the ordering that earlier tests missed. Adjust paths and chat identifiers for your environment.

### Prerequisites

- Linux lab host or VM with Hermes Agent installed and gateway running
- Signal CLI linked for the operator destination used by Hermes
- Access to `hermes` CLI and the Kanban SQLite database path used by your install

### Procedure

1. Confirm gateway health from the dashboard or service status.
2. Create a controlled task in blocked state:

```bash
hermes kanban create \
  --title "Signal notification validation" \
  --assignee default \
  --initial-status blocked \
  --body "Write a small proof artifact and complete successfully."
```

3. Subscribe before execution:

```bash
hermes kanban notify-subscribe TASK_ID \
  --platform signal \
  --chat-id YOUR_SIGNAL_NUMBER
```

4. Unblock and dispatch:

```bash
hermes kanban unblock TASK_ID
hermes kanban dispatch
hermes kanban watch
```

5. Confirm completion in Kanban, then inspect SQLite event and notification checkpoints.
6. Confirm the Signal device received the completion message.
7. For skill work, run local tests before delegating larger follow-on tasks:

```bash
cd ~/.hermes/skills/productivity/sms-custom-sender
PYTHONDONTWRITEBYTECODE=1 python3 scripts/test_send_sms.py
```

### Expected result

The subscription exists before completion, the notifier checkpoint advances through the completion event, and the operator receives the Signal message.

### Common failure mode to avoid

If you subscribe after the task is already `done`, Kanban will still show completion while Signal stays silent. That is the exact failure mode that required the `t_8ef3019d` retest.

---

## 21. Future Engineering Extensions

Additional agentic security research capabilities that build naturally on this lab:

- Broader red-team research boards with stricter approval gates
- Reusable skills for reconnaissance note-taking, evidence packaging, and report drafting
- Richer notification routing by severity or board
- Hardened secret handling and sealed workspace policies
- Automated regression checks for notification cursor invariants
- Optional demo application implementation under the revised consent-based specification, if product evidence is desired later

---

## 22. Conclusion

This project is an engineering case study of a self-hosted Hermes Agent environment used for autonomous security research support, multi-agent orchestration, persistent Kanban execution, custom skill development, and Signal-based operational notification.

The primary accomplishment is not a shipment-tracking product. It is the configuration, integration, operation, troubleshooting, and validation of an autonomous AI automation lab with durable task state and independently confirmed messaging. Several important outcomes only appeared after failed attempts, database forensics, and deliberate retesting.

The screenshots preserved in `docs/screenshots/` are part of that engineering record. They document the workflow as it actually ran — including blocked reviews, dependency waits, skill development, notifier investigation, specification revision, and the path from silent Signal tests to verified delivery.

---

## 23. Suggested Repository Metadata

The existing GitHub repository name can remain unchanged until the owner decides to rename it. Recommended description and topics for the GitHub UI:

**Suggested description:**

> Self-hosted autonomous AI security automation lab using Hermes Agent, multi-agent Kanban orchestration, custom skills, SQLite event tracking, and Signal notifications.

**Suggested topics:**

`hermes-agent`, `agentic-ai`, `cybersecurity`, `security-automation`, `multi-agent-systems`, `kali-linux`, `red-team-lab`, `sqlite`, `signal-cli`, `workflow-orchestration`

---

## 24. Session Evidence Index

All screenshots below are preserved at their original paths. They are also embedded beside the corresponding engineering challenges above.

| File | Captured | Engineering role |
| --- | --- | --- |
| [01-kanban-dhl-task-and-blocked-review.png](docs/screenshots/01-kanban-dhl-task-and-blocked-review.png) | 13 Nov 2026, 16:36 ET | Blocked skill review + Ready board state |
| [02-blocked-task-worker-log.png](docs/screenshots/02-blocked-task-worker-log.png) | 13 Nov 2026, 16:36 ET | Worker claim/heartbeat and human-gated blocker |
| [03-hermes-skills-directory.png](docs/screenshots/03-hermes-skills-directory.png) | 13 Nov 2026, 16:39 ET | Custom skill path inspection |
| [04-local-tests-and-kanban-create.png](docs/screenshots/04-local-tests-and-kanban-create.png) | 13 Nov 2026, 16:58 ET | Local validation then simulator task creation |
| [05-gateway-kanban-notifier.png](docs/screenshots/05-gateway-kanban-notifier.png) | 13 Nov 2026, 18:36 ET | Notifier investigation during Signal debugging |
| [06-signal-subscription-redacted.png](docs/screenshots/06-signal-subscription-redacted.png) | 13 Nov 2026, 19:39 ET | Subscription management evidence |
| [07-revised-specifications-and-unblock.png](docs/screenshots/07-revised-specifications-and-unblock.png) | 13 Nov 2026, 19:42 ET | Security-driven requirements revision |
| [08-parent-task-in-progress.png](docs/screenshots/08-parent-task-in-progress.png) | 13 Nov 2026, 19:55 ET | Autonomous parent execution after unblock |
| [09-child-task-waiting-on-parent.png](docs/screenshots/09-child-task-waiting-on-parent.png) | 13 Nov 2026, 19:55 ET | Dependency wait mistaken for stuck work |

**Documentation basis:** Hermes CLI output, Kanban board state, worker summaries, SQLite event/checkpoint inspection, Signal delivery confirmation, VMware storage error messages, and the preserved screenshots listed above.
