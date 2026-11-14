# Engineering Challenges

Each incident follows the same chain: objective, initial attempt, failure, investigation, root cause, corrective action, retesting, verified outcome. Narrative context and the screenshots sit in the [README](../README.md#17-engineering-obstacles-and-troubleshooting).

| ID | Challenge | Status |
| --- | --- | --- |
| A | No Signal message after Kanban completion | **Resolved** |
| B | Child task left in Todo | **Resolved** (correct wait) |
| C | Tracking specification blocked | **Resolved** for the gate; application **Outstanding** |
| D | VMware host disk full | **Investigated** |
| E | Screenshots split across guest and host | **Resolved** |
| F | Live SMS sender signup from a headless worker | **Mitigated** |
| G | Done column treated as notification proof | **Resolved** |

## Incident A — Missing Signal notifications

**Objective.** Deliver a Signal message when a Kanban task completed, without polling the board.

**Initial attempt.** Signal CLI was available to Hermes. Notification test tasks `t_f13e50d6` and `t_a23f00c8` were allowed to run to completion. The expectation was that `kanban_watchers_notifier.py` would forward the terminal events.

**Failure.** Both cards reached Done. The operator device received no completion message. The board looked healthy.

**Investigation.**

- Source search under `~/.hermes/hermes-agent/gateway` found `kanban_watchers.py`, `kanban_watchers_notifier.py`, `kanban_watchers_dispatcher.py`, and the mixin import in `gateway/run.py`. See [Figure 5](screenshots/05-gateway-kanban-notifier.png).
- `~/.hermes/config.yaml` contained `kanban.review_dispatch: true`.
- `~/.hermes/kanban.db` had completion events for the finished tasks.
- Subscriptions for those task IDs were created after the completions. [Figure 6](screenshots/06-signal-subscription-redacted.png) shows a subscribe loop running while the Signal tests are already `done`.

**Root cause.** The notifier checkpoint is a watermark. A subscription created after the completion event starts past that event. Hermes does not replay historical completions. The gateway process and the Signal transport were present; the workflow order was wrong.

**Corrective action.**

1. Create the next validation task with `--initial-status blocked`.
2. `hermes kanban notify-subscribe` while the task is blocked.
3. Unblock and dispatch.
4. Compare the new completion event ID with the event checkpoint and the notification checkpoint.

**Retesting.** Task `t_8ef3019d`.

**Verified outcome.** Completion event 152. Last event checkpoint 152. Last notification checkpoint 152. The completion message was received on Signal. **Resolved.**

## Incident B — Parent-child dependency blocking

**Objective.** Run a parent coordination task and a child implementation task under Auto orchestration.

**Initial attempt.** Parent `t_5025728e` (`@default`) and child `t_cd335286` (`@dhlpackagetracker`) were on the `researcher` board. The child was expected to become runnable on its own.

**Failure.** The child stayed in Todo while the parent was In Progress. That looked like a stuck dispatcher.

**Investigation.** The Todo column is labeled as waiting on dependencies or unassigned. The child card remained there. The parent card was claimed and In Progress. Captures: [Figure 8](screenshots/08-parent-task-in-progress.png) and [Figure 9](screenshots/09-child-task-waiting-on-parent.png).

**Root cause.** The child depends on the unfinished parent. Auto orchestration was honoring that edge.

**Corrective action.** Do not force the child to Ready while the parent is open. Let the parent finish.

**Retesting.** Board state at 19:55 on 13 Nov 2026, with Gateway Status: Running and two active sessions.

**Verified outcome.** Parent in progress, child waiting. Behavior confirmed. No dispatcher patch was required. **Resolved.**

## Incident C — Security review block

**Objective.** Have Hermes orchestrate a package-tracking demonstration, including an optional location feature and an admin view.

**Initial attempt.** Parent and child tasks were created from the original bodies and workers were allowed to implement them.

**Failure.** The agent blocked the specification. The original bodies, visible in scrollback behind later commands, asked for visitor and location collection without a sufficient consent and disclosure story, and for an administrative log protected by a hardcoded password.

**Investigation.** The operator treated the block as a requirements defect. Kanban state showed the work held for review rather than partially implemented under the unsafe text.

**Root cause.** The task text conflicted with consent, minimum collection, and credential handling for an authorized lab demo. This was not a tool failure.

**Corrective action.** Both bodies were replaced with `hermes kanban edit`, then the parent was released:

```bash
hermes kanban unblock t_5025728e
```

The saved text requires a labeled demo, mock shipment data when no API key is set, GPS only after explicit consent, an admin password taken from the environment, and no public deployment unless one is explicitly approved. Capture: [Figure 7](screenshots/07-revised-specifications-and-unblock.png).

**Retesting.** After unblock, `t_5025728e` was observed In Progress. `t_cd335286` remained Todo because of Incident B.

**Verified outcome.** The human gate and the edited scope are verified. Building, deploying, and acceptance-testing the demo are **Outstanding**. **Resolved** for the control only.

## Incident D — VMware storage exhaustion

**Objective.** Keep the Kali guest available for Hermes.

**Initial attempt.** Continue the session on the existing VMware disk chain.

**Failure.** VMware reported that the Windows host did not have enough free space to grow a Kali virtual-disk segment. The reported file was a snapshot-chain segment of the form `...\kalinov26\kalinov26-000001-s030.vmdk`.

**Investigation.** The message is a host capacity error. Hermes configuration was not the failing component.

**Root cause.** The Windows host filesystem was full.

**Corrective action.** Leave the VMware dialog open, free space on the host, and retry the disk operation. Do not delete `.vmdk` segments or snapshot metadata by hand; they belong to the disk chain.

Host checks that apply:

```powershell
Get-PSDrive C
Start-Process "ms-settings:storagesense"
```

**Retesting.** Not recorded.

**Verified outcome.** Cause identified. Guest recovery after freeing space was **not verified**. **Investigated.**

## Incident E — Cross-platform screenshot management

**Objective.** Commit the session captures with the engineering record.

**Initial attempt.** Take screenshots in the Kali guest and assemble the repository on Windows.

**Failure.** The image files were not on the machine where the repository was being written.

**Investigation.** The transferable set was the nine captures taken on 13 Nov 2026 between 16:36 and 19:55 ET. Phone numbers and Signal chat identifiers were covered before commit. The lab account name in the shell prompt was left visible.

**Root cause.** The agent runtime and the documentation workspace were on different sides of VMware.

**Corrective action.** Copy the files through VMware shared folders into `docs/screenshots/` and link each one from the README with a caption.

**Retesting.** All nine paths resolve and are referenced from the README.

**Verified outcome.** **Resolved** for this evidence set. The screenshots were not regenerated or replaced.

## Incident F — Live alphanumeric SMS onboarding

**Objective.** Find a gateway that can send with a custom or alphanumeric sender name, and wrap that in a Hermes skill.

**Initial attempt.** Research tasks `t_440451bd` and `t_2d59e102`, then skill task `t_0022291f` on `~/.hermes/skills/productivity/sms-custom-sender/`.

**Failure.** The worker built the skill and then blocked. Live registration needed a human inbox or Signal channel, a non-US number, and a brand name. A US number could not satisfy the alphanumeric-sender criterion on the gateways that were researched. See [Figure 1](screenshots/01-kanban-dhl-task-and-blocked-review.png) and [Figure 2](screenshots/02-blocked-task-worker-log.png).

**Investigation.** The block reason was in the task’s final result, not in a crashed process. Local tests of the skill still passed (35 tests, Figure 4).

**Root cause.** Identity verification and numbering policy are outside a headless worker. One of the original success criteria was not satisfiable with the number type under test.

**Corrective action.** Keep the skill and the passing local suite. Add `t_53e53c1e`, a local HTTP simulator that must not send real SMS, on workspace `~/hermes-workspaces/sms-gateway`.

**Retesting.** Figure 4 shows the tests passing and the simulator task being created. Figure 8 shows `t_53e53c1e` in Done.

**Verified outcome.** Research and local skill tests completed. Live sender onboarding did not. **Mitigated.**

## Incident G — Kanban Done is not delivery

**Objective.** Decide whether Signal automation worked.

**Initial attempt.** Use Done on `t_f13e50d6` and `t_a23f00c8` as the pass condition.

**Failure.** Those cards were Done and the phone was silent. Both observations were accurate.

**Investigation.** Completion status, event ID, subscription time, notification checkpoint, and device receipt were separated into different checks. Incident A is the result of that split.

**Root cause.** The pass condition stopped at the first layer.

**Corrective action.** A notification test passes only when the completion event ID, the event checkpoint, the notification checkpoint, and the Signal receipt all agree.

**Retesting.** `t_8ef3019d` / event 152 / checkpoints 152 / message received.

**Verified outcome.** **Resolved** as the lab’s acceptance rule for messaging. The two earlier Done cards stay listed as incomplete notification proofs.
