# Implementation

How the lab was operated: gateway configuration, Kanban commands, the custom skill, the local simulator, Signal subscription order, and the SQLite checks that proved event 152. Command examples use placeholders for any chat identifier. Do not commit real phone numbers.

Return to the [README](../README.md).

## Environment

| Item | Value observed |
| --- | --- |
| Guest | Kali Linux on VMware Workstation |
| Account | `cybergeorge` |
| Hermes | v0.23.1 |
| Config | `~/.hermes/config.yaml` |
| Database | `~/.hermes/kanban.db` |
| Skill | `~/.hermes/skills/productivity/sms-custom-sender/` |
| Gateway source | `~/.hermes/hermes-agent/gateway/` |
| Dashboard | `http://127.0.0.1:9119` |

The Kanban section of `config.yaml` included:

```yaml
kanban:
  review_dispatch: true
```

That block was read with:

```bash
grep -n -A 15 '^kanban:' ~/.hermes/config.yaml
```

## Profiles

Tasks in the completed research and notification set were assigned to `default`. The tracking child was assigned to `dhlpackagetracker`. The dashboard profile description for `default` is the offensive-security and review text visible in [Figure 1](screenshots/01-kanban-dhl-task-and-blocked-review.png).

## Kanban operations used

```bash
hermes kanban list
hermes kanban watch
hermes kanban show TASK_ID
hermes kanban dispatch

hermes kanban create \
  --title "Build free local SMS gateway simulator" \
  --assignee default \
  --workspace dir:/home/cybergeorge/hermes-workspaces/sms-gateway \
  --body "Build a local HTTP API. Do not send real SMS or require a paid provider credential."

hermes kanban edit TASK_ID --body "Revised scope"
hermes kanban unblock TASK_ID
```

Create, edit, and unblock were captured for the tracking tasks in [Figure 7](screenshots/07-revised-specifications-and-unblock.png). The simulator create was captured in [Figure 4](screenshots/04-local-tests-and-kanban-create.png).

## Task inventory

| Task ID | Title / role | Assignee | Outcome |
| --- | --- | --- | --- |
| `t_440451bd` | SMS gateway provider research | `default` | Done |
| `t_0022291f` | Custom sender skill | `default` | Done; live signup left blocked for a human |
| `t_2d59e102` | Alphanumeric sender ID research | `default` | Done |
| `t_53e53c1e` | Local SMS gateway simulator | `default` | Done |
| `t_f13e50d6` | Signal notification test | `default` | Done, no device message |
| `t_a23f00c8` | Signal notification test 2 | `default` | Done, no device message |
| `t_8ef3019d` | Signal notification test 3 | `default` | Done, Signal delivered, event 152 |
| `t_5025728e` | Parent tracking page | `default` | In Progress after unblock |
| `t_cd335286` | Child tracking demo | `dhlpackagetracker` | Todo, waiting on parent |

`t_0022291f` also recorded parent `t_61dab438` in the inspector. That parent link is visible in Figure 1; the parent card itself is not one of the nine primary outcomes above.

## Custom skill

Path confirmed in [Figure 3](screenshots/03-hermes-skills-directory.png):

```text
~/.hermes/skills/productivity/sms-custom-sender/
```

Local suite:

```bash
cd ~/.hermes/skills/productivity/sms-custom-sender
PYTHONDONTWRITEBYTECODE=1 python3 scripts/test_send_sms.py
```

Observed result: `Ran 35 tests in 0.781s` and `OK`. The names visible in the run cover dry-run, recipient checks, country codes, gateway error shapes, missing keys, and sender validation (length, spaces, punctuation, numeric senders).

The worker that built the skill stopped before a live send. Its result, visible in Figure 1, required:

- a human to complete email or Signal signup verification,
- a non-US number that can receive alphanumeric sender IDs,
- a brand name.

It also recorded that a US number cannot receive alphanumeric sender IDs on the gateways that were researched. The worker log for that stop is [Figure 2](screenshots/02-blocked-task-worker-log.png).

## Local simulator

Task `t_53e53c1e` was created against:

```text
dir:/home/cybergeorge/hermes-workspaces/sms-gateway
```

The body in Figure 4 asks for a local HTTP API with configurable sender names, message history, simulated delivery, and realistic API errors, and it forbids real SMS and paid credentials. The card is in the Done column in [Figure 8](screenshots/08-parent-task-in-progress.png).

## Signal subscription

Subscribe a task before it is allowed to complete:

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
hermes kanban notify-list
```

[Figure 6](screenshots/06-signal-subscription-redacted.png) shows the opposite order: `hermes kanban list` already has the Signal tests in `done`, and a loop then subscribes those IDs. That loop cannot replay the completions it missed. Chat identifiers in the capture are covered.

### Verified delivery

| Field | Value |
| --- | --- |
| Task | `t_8ef3019d` |
| Completion event | 152 |
| Last event checkpoint | 152 |
| Last notification checkpoint | 152 |
| Device | Completion message received |

`t_f13e50d6` and `t_a23f00c8` are not part of that verification. They completed in Kanban only.

## SQLite inspection

The database path used in the session was `~/.hermes/kanban.db`. The investigation compared three facts for a finished task:

1. The completion row in the task event log, and its event ID.
2. When the Signal subscription for that task ID was created.
3. The stored event checkpoint and notification checkpoint.

If the subscription and the checkpoint are already past the completion event, the watcher has nothing to send. The successful retest is the case where all three of event ID, event checkpoint, and notification checkpoint are 152, and the phone received the message.

Gateway source was searched with:

```bash
rg -n 'kanban_watchers|KanbanNotify|notify_watcher' ~/.hermes/hermes-agent/gateway
```

Matches included `gateway/run.py`, `gateway/kanban_watchers.py`, `gateway/kanban_watchers_notifier.py`, and `gateway/kanban_watchers_dispatcher.py`, shown in Figure 5.

## Tracking-demo edits

These are the bodies saved in Figure 7. They replace the earlier specification; they are not evidence that the application was built.

Parent `t_5025728e`:

```text
Build and test a functional package-tracking web application using DHL's official Shipment Tracking API, with a mock mode for demonstrations. Include a responsive tracking interface, shipment history, optional consent-based location sharing, and a secure administrative dashboard. Preserve project files in /home/cybergeorge/hermes-workspaces/dhl-tracking-lab. Coordinate implementation with child task t_cd335286.
```

Child `t_cd335286`:

```text
Implement a clearly labeled package-tracking demo with optional DHL API integration. Display realistic simulated shipment information when no API key is configured. Request GPS location only through explicit informed consent; tracking must work without GPS. Collect only necessary diagnostic information with disclosure and consent. Build a secure authenticated admin dashboard with environment-configured credentials, location maps for consented records, and no hardcoded passwords. Test locally before any public deployment. Use Cloudflare Quick Tunnel only for an explicitly approved demo deployment.
```

```bash
hermes kanban unblock t_5025728e
hermes kanban dispatch
```

After that unblock, Figure 8 shows the parent In Progress and Figure 9 shows the child still in Todo.

## Files this repository contains

Documentation and the nine screenshots only. The skill, the simulator, `kanban.db`, and any tracking-demo workspace stayed on the Kali guest. They are described here from CLI output and screenshots; they are not re-shipped as application source.
