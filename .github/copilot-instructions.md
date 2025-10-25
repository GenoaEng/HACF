## Quick orientation

This repository is a Home Assistant configuration for a cultivation facility (HACF). It uses YAML-based configuration with top-level includes and local blueprints. An AI coding agent should treat this like a Home Assistant config repo (not an application codebase).

Key file layout to reference:

- `configuration.yaml` — root config; includes `automations.yaml`, `scripts.yaml`, `scenes.yaml`, themes and Lovelace resources (see `lovelace:` section).
- `automations.yaml` — primary automations file (YAML automations are used directly).
- `scripts.yaml`, `scenes.yaml` — included by `configuration.yaml` (scripts file may be empty).
- `blueprints/automation/homeassistant/` — stores automation blueprints (example: `motion_light.yaml`).
- `zigbee2mqtt/configuration.yaml` — Zigbee2MQTT adapter config; integrates with MQTT broker `core-mosquitto` (sensitive secrets appear here).
- `secrets.yaml` — central place for credentials. Avoid exposing secrets in responses.

## Big-picture architecture notes

- This is a Home Assistant config repository: components communicate via Home Assistant entities and services. External integration example: Zigbee devices talk to Home Assistant via `zigbee2mqtt` -> `mqtt` -> `core-mosquitto`.
- Lovelace UI is configured in YAML (`lovelace.mode: yaml`) and references HACS resources (e.g., `decluttering-card`, `button-card`) listed under `resources:` in `configuration.yaml`.
- Blueprints follow Home Assistant blueprint structure; many automation patterns reuse inputs and device_class filters (see `blueprints/automation/homeassistant/motion_light.yaml`).

## Project-specific conventions and patterns

- Single-file includes: top-level `configuration.yaml` uses `!include` to include `automations.yaml`, `scripts.yaml`, `scenes.yaml`. When adding a new automation prefer adding it to `automations.yaml` or creating a new file and including it from `configuration.yaml`.
- Blueprints are stored under `blueprints/automation/homeassistant/` and may be upstream copies—preserve upstream structure and comments where possible.
- Lovelace: UI resources are managed by HACS and referenced in `configuration.yaml`; do not hard-delete HACS entries without verifying HACS state.
- Secrets: Use `secrets.yaml` for passwords/keys. This repo currently contains a password-like string inside `zigbee2mqtt/configuration.yaml` — treat such data as secrets and avoid printing them in generated responses.

## Typical edit / validation workflow (discoverable from repo)

- Edit YAML (automations, scenes, scripts, blueprints) in this repo and push.
- Validate before applying to a running Home Assistant instance using the Home Assistant config checker (not present in repo; common tools include `ha core check` or `python -m homeassistant --script check_config` in a HA environment). If you can't run the HA CLI locally, do careful syntax checks (YAML lint) and small, incremental changes.

## Examples to follow when writing automations

- Motion-activated light (blueprint example): uses a state trigger on a motion sensor switching from `off` to `on`, calls `light.turn_on` with a `target:` block, waits for motion to go `off`, delays by `no_motion_wait`, then `light.turn_off`. Follow the same `target:` usage and `mode: restart` pattern for similar automations.

## Integration and cross-component notes

- Zigbee2MQTT uses the `mqtt` section in `zigbee2mqtt/configuration.yaml` and points at `mqtt://core-mosquitto:1883`. Service-level changes to MQTT or Mosquitto will impact the Zigbee device feed.
- Frontend resources are loaded from `/hacsfiles/*` paths — HACS-managed files are expected outside this repo; do not remove `resources:` entries unless you confirm HACS changes.

## Safety and secure handling

- Do not output or copy any secret values (passwords, network keys) into chat or generated PRs. When producing examples, redact secrets (e.g. `password: <REDACTED>`).

## When to open a PR vs quick edit

- Small changes to automations or scenes can be edited directly if you have CI or a staging Home Assistant to validate; otherwise create a PR with a short description and a `how to validate` section explaining how to reload automations or test the change.

## Where to look for more examples

- `blueprints/automation/homeassistant/motion_light.yaml` — canonical example of a blueprint/automation structure.
- `configuration.yaml` — authoritative list of included files and Lovelace resource references.

If any section here is unclear or you'd like me to expand specific examples (e.g., how to add a new automation that uses MQTT or how to safely update Zigbee network keys), tell me which area and I'll update the file.
