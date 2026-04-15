# Home Assistant Config — skrinv

## Overview

This repo manages the Home Assistant configuration for the `skrinv` instance, running at `homeassistant` (port 8123). Configuration is version-controlled and auto-synced to the host on save via a VS Code `runonsave` task.

## File Structure

| File                 | Purpose                                                   |
| -------------------- | --------------------------------------------------------- |
| `configuration.yaml` | Main entry point; loads all other files via `!include`    |
| `automations.yaml`   | All automations (`automation: !include automations.yaml`) |
| `scripts.yaml`       | All scripts (`script: !include scripts.yaml`)             |
| `scenes.yaml`        | All scenes (`scene: !include scenes.yaml`)                |
| `secrets.yaml`       | Secret values — **gitignored, never commit**              |

Themes are loaded from a `themes/` folder via `!include_dir_merge_named themes`.

## Secrets

- Sensitive values live in `secrets.yaml` (gitignored).
- Reference them in config with `!secret <key>` (e.g. `!secret geopulse_bearer`).
- Never inline credentials directly in YAML files.

## Sync Workflow

The `.vscode/settings.json` configures `emeraldwalk.runonsave`:

- On save of any config YAML, the file is rsynced to `root@homeassistant:/config/` over SSH (key: `~/.ssh/id_ed25519`, port 22).
- Immediately after, a `POST /api/services/homeassistant/reload_all` is called using a token from the macOS Keychain item `homeassistant-api-token`.
- No manual deployment step is needed; saving triggers the full sync+reload cycle.

## Integrations in Use

- **`avanza_stock`** — Stock sensor for Avanza stock ID 155541 (USD).
- **`prometheus`** — Exports metrics under the `homeassistant` namespace; includes `sensor`, `binary_sensor`, `light`, `climate`, `person`, `device_tracker`, `fan`, `switch`.
- **`rest_command.send_gps_data`** — POSTs GPS/battery data for device `sm_s911b` to `http://geopulse.lan/api/homeassistant` using a Bearer token from secrets.
- **Trusted proxy** — Caddy reverse proxy on `192.168.1.0/24` is trusted via `http.trusted_proxies`.

## Device Tracking

The primary tracked device is `device_tracker.sm_s911b` (Samsung Galaxy S23). Automations read its `latitude`, `longitude`, `gps_accuracy`, `altitude`, `speed`, and `battery_level` attributes.

## Conventions

- Automation `alias` values should be short, human-readable descriptions (e.g. `Send GPS data to server`).
- All `rest_command` URLs pointing to internal services use `.lan` hostnames (e.g. `geopulse.lan`).
- Template expressions use `{{ }}` syntax; use `| default(0, true)` to guard against `None` attribute values.
- All YAML files use 2-space indentation.
