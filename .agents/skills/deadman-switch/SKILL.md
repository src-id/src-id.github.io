---
name: deadman-switch
description: Arm, monitor, and ping Dead Man's Switch failsafes for autonomous operations, dangerous migrations, long background tasks, or unattended deployments.
version: 1.0.0
author: WanForge
license: MIT
platforms: [linux, macos]
metadata:
  tags: [deadman-switch, failsafe, watchdog, automation, safety, heartbeat]
  related_skills: [systematic-debugging, bash-defensive-patterns, incident-response]
---

# Dead Man's Switch Safety & Automation

## Overview

Dead Man's Switch provides automated failsafe actions if an operator or autonomous agent fails to send periodic heartbeat check-ins (`ping`) within a given timeout window.

## When to Use

- **Risky Deployments / DB Migrations**: Arm automatic rollback switch before altering schema or configs.
- **Long Unattended Background Runs**: Alert operator via desktop notification or webhook if agent hangs/crashes.
- **Ephemeral Access / Firewall Rules**: Auto-revoke temporary port openings or sudo tokens if not refreshed.
- **Failover & Watchdog**: Execute service restart or fallback trigger upon heartbeat timeout.

## Available MCP Tools

| Tool | Purpose | Key Arguments |
|---|---|---|
| `arm_deadman_switch` | Arm new countdown with trigger action | `id`, `timeoutSeconds`, `actionType`, `actionPayload`, `name` |
| `ping_deadman_switch` | Reset countdown timer (heartbeat) | `id` (optional, omits = ping all) |
| `status_deadman_switch` | Inspect active countdowns and state | `id` (optional) |
| `cancel_deadman_switch` | Disarm and cancel switch | `id` |
| `trip_deadman_switch` | Manually fire switch (drill / emergency) | `id` |

## Supported Action Types

1. **`command`**: Executes shell script or CLI (e.g. `docker-compose restart`, rollback script, git restore).
2. **`webhook`**: Sends HTTP POST with switch metadata and timestamps to external URL (Slack, Discord, PagerDuty, n8n).
3. **`notification`**: Triggers desktop notification (`notify-send` on Linux / `osascript` on macOS).
4. **`note`**: Appends alert incident to `~/www/my-note/WORK REPORT/deadman_alerts.md`.

## Standard Patterns

### Pattern 1: Safe Migration Rollback
```json
// 1. Arm switch before migration
arm_deadman_switch({
  id: "migration-guard-01",
  name: "DB Migration Rollback Guard",
  timeoutSeconds: 300,
  actionType: "command",
  actionPayload: "npm run migrate:rollback && notify-send 'DB Migration' 'Auto-rollback executed due to timeout'"
})

// 2. Run migration steps...
// 3. Ping if long-running
ping_deadman_switch({ id: "migration-guard-01" })

// 4. Disarm when migration succeeds
cancel_deadman_switch({ id: "migration-guard-01" })
```

### Pattern 2: Agent Task Watchdog
```json
arm_deadman_switch({
  id: "subagent-watchdog",
  name: "Long ETL Task Monitor",
  timeoutSeconds: 1800,
  actionType: "notification",
  actionPayload: "ETL task timed out without completion heartbeat!"
})
```
