---
Last Updated: 2026-04-13
Maintainer: Spencer Long
Related system: GenomeDepot (protect.qb3.berkeley.edu/genomedepot/)
Script path: /usr2/people/spencerlong/genomedepot_data/ops/genomedepot_watchdog.sh
---

# GenomeDepot — Active Issues & Operational Notes

## Status Summary

| # | Issue | Owner | Blocking analysis? | Status |
|---|-------|-------|--------------------|--------|
| 1 | Zombie `stopping` state caused 4-day outage (Apr 9–13) | Spencer Long | No (resolved) | ✅ Resolved |
| 2 | Watchdog v2 deployed with zombie recovery + Slack alerts | Spencer Long | — | ✅ Done |

---

## Resolved Items

### 1. gd-web Zombie Stopping State — April 9–13, 2026

**Plain language:** GenomeDepot was down for approximately 4 days (April 9 19:23 PDT through April 13 09:27 PDT), returning a 502 Bad Gateway from nginx. The watchdog script fired every 2 minutes throughout this period but was unable to recover the service automatically. The underlying cause was a Podman container stuck in a `stopping` state — the OS process had already died, but Podman's internal state database never received a clean exit signal and froze mid-shutdown. Because Podman's state machine only permits `start` from `Created` or `Stopped` states, every automated and manual restart attempt failed. The watchdog had no handler for this scenario.

**Technical detail:** `podman inspect gd-web` showed `"Status": "stopping"`, `"Running": false`, and `"FinishedAt": "0001-01-01T00:00:00Z"` (epoch zero — container never cleanly exited). PID 6192 referenced in Podman state was a ghost; `podman kill gd-web` failed with `open pidfd: No such process`. `podman start gd-web` failed with `container must be in Created or Stopped state: container state improper` (exit 125). `systemctl restart container-gd-web.service` also failed for the same reason. The watchdog cooldown (600s) further suppressed most retry attempts after the first failure on April 9. `gd-db` was healthy throughout; the database was not involved. Recovery required: `podman rm -f gd-web` to clear the stale state record, `podman run` with the original flags to recreate the container, and `systemctl --user start container-gd-web.service` to re-sync systemd ownership.

**Manual recovery reference (if this recurs before watchdog can act):**

```bash
# Step 1 — clear the zombie record
podman rm -f gd-web

# Step 2 — recreate the container (exact original run command)
podman run -d \
  --pod genomedepot-pod \
  --name gd-web \
  --env-file /usr2/people/spencerlong/genomedepot_data/.env \
  -v /usr2/people/spencerlong/genomedepot_data/static:/app_static \
  -v /usr2/people/spencerlong/genomedepot_data/logs:/app_logs \
  -v /usr2/people/spencerlong/genomedepot_data/appdata:/app_appdata \
  -v /usr2/people/spencerlong/genomedepot_data/tmp:/app_tmp \
  -v /usr2/people/spencerlong/genomedepot_data/configs.txt:/app/genomebrowser/configs.txt:ro \
  -v /usr2/people/spencerlong/GenomeDepot/ref_data:/app/ref_data:ro \
  localhost/genomedepot:web

# Step 3 — re-sync systemd
systemctl --user start container-gd-web.service
systemctl --user reset-failed container-gd-web.service
```

---

### 2. Watchdog v2 Upgrade — April 13, 2026

**Plain language:** Following the April 9–13 outage, the GenomeDepot watchdog script was upgraded from v1 to v2. The upgrade adds automatic detection and recovery of the zombie `stopping` state that caused the outage, so future occurrences self-heal without manual intervention. It also adds Slack alerting to `#protect-tools-alerts` in the PROTECT workspace, so any recovery event (success or failure) is visible to the team rather than silently failing for days.

**Technical detail:** v2 adds two new functions to the existing watchdog logic. `is_zombie_stopping()` inspects the container state and checks for `"Status": "stopping"` combined with a `FinishedAt` epoch-zero timestamp, confirming the OS process is dead but Podman state is frozen. `recover_zombie_container()` runs `podman rm -f gd-web` followed by `podman run` with the exact original flags (all 6 volume mounts, env-file, pod membership), then re-syncs systemd via `systemctl --user start`. This recovery path is inserted before the existing `systemctl restart` path in the systemd-owned branch. `send_slack_alert()` reads a bot token from `/usr2/people/spencerlong/genomedepot_data/ops/.slack_token` (chmod 600) and posts to channel `C0ASPHLJ2MS` (`#protect-tools-alerts`) via curl to the Slack API. Alerts fire on: successful auto-recovery, failed recovery requiring manual intervention, cooldown suppression while unhealthy, and `gd-db` down. The v1 script is preserved at `genomedepot_watchdog_v1_backup.sh`.

**Key file locations:**

| File | Path |
|------|------|
| Watchdog script (v2) | `/usr2/people/spencerlong/genomedepot_data/ops/genomedepot_watchdog.sh` |
| Watchdog script (v1 backup) | `/usr2/people/spencerlong/genomedepot_data/ops/genomedepot_watchdog_v1_backup.sh` |
| Slack token | `/usr2/people/spencerlong/genomedepot_data/ops/.slack_token` (chmod 600, never commit) |
| Cooldown file | `/usr2/people/spencerlong/genomedepot_data/ops/.gd_web_last_restart_epoch` |
| systemd timer | `genomedepot-watchdog.timer` (fires every ~2 minutes) |
| Slack channel | `#protect-tools-alerts` (ID: C0ASPHLJ2MS, protect-ihr3502 workspace) |

> **Note:** The Slack token file must never be committed to any repository. It is stored only on the server at the path above with owner-only read permissions (chmod 600).
