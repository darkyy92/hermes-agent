# Local Hermes patches

This directory contains local patch overlays that must survive `hermes update`.

Hermes is installed from upstream `NousResearch/hermes-agent`, but this machine intentionally runs a few product defaults and safety fixes before they are upstream. `hermes update` resets the source checkout to upstream and then reapplies these overlays.

## Patch order

1. `001-gateway-steer-default.patch`
   - Gateway busy messages default to `steer`.
   - `/steer` remains available, but normal Messenger messages during active work steer the current task.
   - `agent.gateway_notify_interval` default is 300 seconds.
   - Adds the local patch overlay hook to `hermes update`.

2. `002-gateway-self-management-guard.patch`
   - Blocks Gateway sessions from hard-stopping or manually launchctl-managing their own live Gateway.
   - Allows safe `hermes gateway restart` and `hermes update`.

3. `003-honcho-context-tokens-zero.patch`
   - Treats Honcho `contextTokens: 0` as no injected context / tools-only recall, not an invalid `tokens=0` API call.

4. `004-media-path-escaped-newline.patch`
   - Cleans escaped trailing `\n`, `\r`, `\t` from `MEDIA:` paths before attachment delivery.

## Verify overlays

```bash
cd ~/.hermes/hermes-agent
for p in ~/.hermes/local-patches/00*.patch; do
  git apply --reverse --check "$p" && echo "applied: $(basename "$p")"
done
```

## Expected production config per profile

```yaml
agent:
  gateway_notify_interval: 300

display:
  busy_input_mode: steer

gateway:
  default_busy_message_mode: steer
```

Expected env overrides per profile:

```dotenv
HERMES_GATEWAY_BUSY_INPUT_MODE=steer
HERMES_GATEWAY_DEFAULT_BUSY_MESSAGE_MODE=steer
```

Do not store secrets in this directory.
