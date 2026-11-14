# Testing and Validation

What was actually checked on 13 Nov 2026, what passed, and what remains unverified. The narrative is in the [README](../README.md#19-verified-testing-and-project-completion).

## Verified

| Check | Evidence | Result |
| --- | --- | --- |
| Gateway process | Dashboard Gateway Status: Running, Hermes v0.23.1, `127.0.0.1:9119` | Pass |
| Kanban persistence | Cards retained across Ready, Blocked, Todo, In Progress, and Done | Pass |
| Worker lifecycle | Claim and heartbeat on the blocked skill; two active sessions while the parent ran | Pass |
| Skill unit tests | `python3 scripts/test_send_sms.py` → `Ran 35 tests in 0.781s`, `OK` | Pass |
| Skill task | `t_0022291f` Done, with live signup still called out as human work | Pass |
| Provider research | `t_440451bd` and `t_2d59e102` Done | Pass |
| Local simulator task | `t_53e53c1e` created against `sms-gateway` and later Done | Pass |
| Signal subscribe command | `notify-subscribe --platform signal` accepted for the listed task IDs | Pass |
| Signal delivery | `t_8ef3019d`, event 152, checkpoints 152, message received | Pass |
| Dependency wait | `t_5025728e` In Progress, `t_cd335286` Todo | Pass |
| Human security gate | Edits saved for both tracking tasks; `Unblocked t_5025728e` | Pass |
| Screenshot set | Nine files under `docs/screenshots/`, identifiers covered | Pass |

## Not verified

| Check | Why it is open |
| --- | --- |
| `t_f13e50d6` and `t_a23f00c8` as Signal proofs | Done in Kanban, no device message |
| Live alphanumeric SMS send | Worker blocked on signup, number type, and brand name |
| VMware recovery | Host disk error recorded; resumed guest operation not confirmed |
| Tracking UI, mock mode, live DHL API, admin login | Tasks revised and parent restarted; no acceptance run is in this repository |
| Public deployment or Cloudflare tunnel | The revised spec allows a tunnel only after an explicit approval; none was verified |

## Signal acceptance record

The pass condition for automated notification is all four of the following.

| Criterion | `t_8ef3019d` |
| --- | --- |
| Task completed after the subscription existed | Yes, on the controlled retest |
| Completion event ID | 152 |
| Last event checkpoint | 152 |
| Last notification checkpoint | 152 |
| Operator Signal device | Message received |

The failed pass condition was “card is in Done.” That condition is true for `t_f13e50d6` and `t_a23f00c8` and is not sufficient.

### How the miss was diagnosed

1. The Done column said the first Signal tests had finished.
2. The phone had no corresponding message.
3. Gateway search showed the notifier modules loaded, so the process was not missing. [Figure 5](screenshots/05-gateway-kanban-notifier.png).
4. SQLite showed completion events that the notification checkpoint had already passed.
5. [Figure 6](screenshots/06-signal-subscription-redacted.png) shows subscriptions being added only after those tasks were already `done`.
6. The retest created the subscription while `t_8ef3019d` was still blocked, then released the task.
7. Event 152 and both checkpoints landed on the same ID, and the device received the message.

## Reproduction

Use a lab where Hermes is already running and `signal-cli` is linked. Replace the chat ID locally; do not commit it.

```bash
hermes kanban create \
  --title "Signal notification validation" \
  --assignee default \
  --initial-status blocked \
  --body "Write a small proof artifact and complete successfully."

hermes kanban notify-subscribe TASK_ID \
  --platform signal \
  --chat-id YOUR_SIGNAL_NUMBER

hermes kanban unblock TASK_ID
hermes kanban dispatch
hermes kanban watch
```

Then confirm, in `~/.hermes/kanban.db`, that the completion event ID equals the last event checkpoint and the last notification checkpoint, and confirm the Signal device received that completion.

Known miss: if `notify-subscribe` runs after the task is already `done`, Kanban still shows Done and Signal stays silent.

Skill regression, from the guest:

```bash
cd ~/.hermes/skills/productivity/sms-custom-sender
PYTHONDONTWRITEBYTECODE=1 python3 scripts/test_send_sms.py
```

Expected: 35 tests, `OK`.

## Evidence index

| File | Demonstrates |
| --- | --- |
| [01-kanban-dhl-task-and-blocked-review.png](screenshots/01-kanban-dhl-task-and-blocked-review.png) | Gateway up, Ready parent, blocked skill, Signal channel selected |
| [02-blocked-task-worker-log.png](screenshots/02-blocked-task-worker-log.png) | Claim, heartbeat, human stop |
| [03-hermes-skills-directory.png](screenshots/03-hermes-skills-directory.png) | Skill path on disk |
| [04-local-tests-and-kanban-create.png](screenshots/04-local-tests-and-kanban-create.png) | 35 tests passing; simulator task created |
| [05-gateway-kanban-notifier.png](screenshots/05-gateway-kanban-notifier.png) | Notifier modules and `review_dispatch: true` |
| [06-signal-subscription-redacted.png](screenshots/06-signal-subscription-redacted.png) | Subscribe-after-complete, chat IDs covered |
| [07-revised-specifications-and-unblock.png](screenshots/07-revised-specifications-and-unblock.png) | Body edits and unblock |
| [08-parent-task-in-progress.png](screenshots/08-parent-task-in-progress.png) | Parent running; Done column |
| [09-child-task-waiting-on-parent.png](screenshots/09-child-task-waiting-on-parent.png) | Child waiting on parent |

Captions for each file are in the [README](../README.md#session-evidence-index).
