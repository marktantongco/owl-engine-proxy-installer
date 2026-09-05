# Owl Engine Proxy Installer — One-Click Setup for `owl-dns-synergy`

> **Unified Dual-Channel Resilient Access Engine — now one-click to install, too.**

[![Owl Engine](https://img.shields.io/badge/powered%20by-owl--dns--synergy-black?style=for-the-badge)](https://github.com/marktantongco/owl-dns-synergy)
[![Python](https://img.shields.io/badge/python-3.10%2B-blue?style=for-the-badge&logo=python)](https://www.python.org)
[![License](https://img.shields.io/badge/license-MIT-green?style=for-the-badge)](LICENSE)
[![Ubuntu 24.04](https://img.shields.io/badge/PEP%20668-ready-orange?style=for-the-badge)](https://peps.python.org/pep-0668/)
[![Version](https://img.shields.io/badge/installer-v4.0.0-blue?style=for-the-badge)](#)

---

## 👋 Welcome — What's In It For You?

**If you want a resilient HTTP + DNS proxy that auto-fails over across 7 channels and just works — without fighting Python packaging, venvs, or systemd — this repo is for you.**

Original `owl-dns-synergy` (OWL-AGENT v4.2 + LLM-DNS-Proxy + 5 aux repos) is powerful: Fernet-encrypted DNS tunneling, 7-channel cascade with Circuit Breakers, EMA learning, Prometheus metrics, AutoClaw OAuth proxy. But installing it means: venv, 15 deps, `cryptography` build, config, systemd, token encryption — easy to get stuck.

This installer fixes that. In one command you get:

- ✅ **Working `owl-dns-synergy` CLI** — no PEP 668 errors, no venv confusion (auto-creates `~/.owl-dns-synergy/venv`)
- ✅ **PEP 668-ready** — uses venv first (Debian-recommended), falls back to `uv`/`pipx`/`--break-system-packages` if needed
- ✅ **Auto-detects your system** — checks Python 3.10+, pip, git, Go 1.21+, Node 18+, Rust
- ✅ **Keys generated** — `LLM_PROXY_KEY` (DNS tunnel) + `AUTOCLAW_TOKEN_KEY` (token-at-rest) via Fernet, `chmod 600`
- ✅ **Config ready** — `~/.owl-dns-synergy/config/config.json` + `.env` + `proxies.txt.example`/`accounts.txt.example`
- ✅ **Systemd ready** — `owl-dns-synergy.service` + `autoclaw-proxy.service` in `~/.owl-dns-synergy/deploy/`, copy to `/etc/systemd/system/`
- ✅ **Verified** — imports `httpx`, `cryptography`, `flask`, `gunicorn`, `dnslib` etc., reports `X passed / Y failed`
- ✅ **Beginner-proof** — colored `[STEP]`/`[OK]`/`[WARN]` logs, `~/.owl-dns-synergy/logs/install.log`, idempotent (safe to re-run), `--uninstall` to nuke

**Result:** `owl-dns-synergy fetch https://example.com` and `owl-dns-synergy serve` just work — even on a fresh Ubuntu 24.04 with Python 3.14.

Same instruction set as [`heretic-installer`](https://github.com/marktantongco/heretic-installer) — one-liner, detailed guide, troubleshooting.

---

## 🚀 Quick Start (Beginners — Copy & Paste)

### Option A: One-liner (curl) — no clone needed
```bash
curl -fsSL https://raw.githubusercontent.com/marktantongco/owl-engine-proxy-installer/main/install.sh | bash
```

### Option B: Clone and run
```bash
git clone https://github.com/marktantongco/owl-engine-proxy-installer.git
cd owl-engine-proxy-installer
bash install.sh
```

### Option C: From the full `owl-dns-synergy` repo
```bash
git clone https://github.com/marktantongco/owl-dns-synergy.git
cd owl-dns-synergy
./install.sh
```

That's it. Idempotent — safe to run again if it failed halfway.

---

## 📋 What You Need Before You Start

| Requirement | How to Check | How to Fix |
|---|---|---|
| **Python 3.10+** | `python3 --version` | `sudo apt update && sudo apt install python3 python3-venv python3-full` |
| **pip** | `python3 -m pip --version` | `python3 -m ensurepip --upgrade` |
| **git** | `git --version` | `sudo apt install git` / `brew install git` |
| **~1GB disk** for venv | `df -h ~` | free up space |
| **Go 1.21+** (optional) | `go version` | https://go.dev/dl/ — for prox5 SOCKS pool |
| **Node 18+** (optional) | `node --version` | https://nodejs.org/ or `nvm install 18` — for secret-agent MITM |
| **Rust/cargo** (optional) | `cargo --version` | `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh` |

> Installer warns (not fails) if Go/Node/Rust missing — only Python+pip+git are critical.

---

## 📖 Full Installation Guide (Step-by-Step for Beginners)

### 1. Open your terminal
Ubuntu: `Ctrl+Alt+T`. macOS: `Terminal`.

### 2. Make sure Python is fresh
```bash
python3 --version  # should say 3.10+
# if not: sudo apt update && sudo apt install python3 python3-venv python3-pip
```

### 3. Run the installer
```bash
bash install.sh  # or the curl one-liner above
```

You'll see:
```
╔══════════════════════════════════════════════════════════════╗
║   OWL-DNS-SYNERGY v1.0.0  Installer v4.0.0                 ║
╚══════════════════════════════════════════════════════════════╝
[PHASE 1/12]  Checking Prerequisites
[OK]      Python 3.14.4 detected
[OK]      pip 24.x detected
...
[PHASE 12/12]  Installation Complete
  Home directory:      /home/you/.owl-dns-synergy
  Virtual environment: /home/you/.owl-dns-synergy/venv
```

### 4. Set your API key (required)
```bash
vi ~/.owl-dns-synergy/config/.env
# Set: OPENAI_API_KEY=sk-...
# Optional: OPENROUTER_KEY_1.._9, LLM_DNS_SUFFIX, etc.
```

### 5. Verify it works (no network needed)
```bash
source ~/.owl-dns-synergy/venv/bin/activate
owl-dns-synergy --help
owl-dns-synergy generate-key
owl-dns-synergy test-connection  # needs OPENAI_API_KEY set
```

### 6. Run your first proxy
```bash
# Fetch through optimal channel (auto-selects from 7):
owl-dns-synergy fetch https://example.com --verbose

# DNS tunnel chat:
owl-dns-synergy chat "What is the capital of France?"

# Start DNS server + AutoClaw proxy:
owl-dns-synergy serve --host 127.0.0.1 --port 5353
# in another terminal:
gunicorn -b 127.0.0.1:31000 autoclaw.app:create_app()
```

### 7. Deploy as systemd (production)
```bash
sudo cp ~/.owl-dns-synergy/deploy/*.service /etc/systemd/system/
sudo mkdir -p /etc/owl-dns-synergy && sudo cp ~/.owl-dns-synergy/config/.env /etc/owl-dns-synergy/env
sudo nano /etc/owl-dns-synergy/env  # set keys
sudo useradd --system --no-create-home owl-dns 2>/dev/null || true
sudo mkdir -p /var/log/owl-dns-synergy /var/cache/owl-dns-synergy
sudo chown owl-dns:owl-dns /var/log/owl-dns-synergy /var/cache/owl-dns-synergy
sudo systemctl daemon-reload
sudo systemctl enable --now owl-dns-synergy autoclaw-proxy
systemctl status owl-dns-synergy autoclaw-proxy
curl http://localhost:9090/metrics | head
```

---

## 🔧 What the Installer Actually Does (12 Phases)

```
[1/12] Prerequisites — Python 3.10+, pip, git, Go 1.21+, Node 18+, Rust
[2/12] Directories   — ~/.owl-dns-synergy/{config,cache,logs,venv,repos,deploy}
[3/12] Venv + Deps   — python3 -m venv, pip install -e ., httpx/aiohttp/dnslib/circuitbreaker/bs4/openai/click + prometheus/cryptography/redis/curl_cffi
[4/12] AutoClaw      — flask/gunicorn/eventlet/requests
[5/12] LLM_PROXY_KEY — Fernet.generate_key() (or openssl fallback)
[6/12] AUTOCLAW_TOKEN_KEY — Fernet (or fallback)
[7/12] config.json   — defaults: cache_ttl 300, dns_suffix _sonos._udp.local, prometheus 9090, CB threshold 5
[8/12] .env          — all env vars, chmod 600
[9/12] Examples      — proxies.txt.example, accounts.txt.example (chmod 600)
[10/12] systemd     — owl-dns-synergy.service + autoclaw-proxy.service
[11/12] Verify      — imports 10 critical + 3 optional, checks config, CLI
[12/12] Summary     — next steps, log at ~/.owl-dns-synergy/logs/install.log
```

Logs: `~/.owl-dns-synergy/logs/install.log` and `/tmp/heretic-*.log` style not needed — this one logs to `~/.owl-dns-synergy/logs/install.log`.

---

## ⚠️ Troubleshooting (Beginner Fixes)

| Error | Fix |
|---|---|
| `Python 3.10+ required` | `sudo apt install python3 python3-venv` or https://www.python.org/downloads/ |
| `externally-managed-environment` | **Already fixed** — installer uses venv (`~/.owl-dns-synergy/venv`), not system pip. Re-run `bash install.sh`. |
| `pip not found` | `python3 -m ensurepip --upgrade` |
| `command not found: owl-dns-synergy` | `source ~/.owl-dns-synergy/venv/bin/activate` — or add `export PATH="$HOME/.owl-dns-synergy/venv/bin:$PATH"` to `~/.bashrc` |
| `cryptography not importable` | `~/.owl-dns-synergy/venv/bin/pip install cryptography==41.0.0` — needs `libffi-dev`/`openssl` on some systems: `sudo apt install libffi-dev libssl-dev` |
| `OPENAI_API_KEY not set` | `vi ~/.owl-dns-synergy/config/.env` → `OPENAI_API_KEY=sk-...` |
| `systemd service fails` | `journalctl -u owl-dns-synergy -f` + `journalctl -u autoclaw-proxy -f`; check `/etc/owl-dns-synergy/env` has keys |
| `curl_cffi skipped` | Optional — needs compilation (`sudo apt install python3-dev libcurl4-openssl-dev`). Safe to ignore. |

**Still stuck?** Open an issue with:
```bash
cat ~/.owl-dns-synergy/logs/install.log | tail -n 100
~/.owl-dns-synergy/venv/bin/python -c "import owl_dns_synergy; print(owl_dns_synergy.__version__)" 2>&1 | head
python3 --version; pip --version; git --version
```

---

## 📈 Version Changes Timeline

| Version | Date | Changes |
|---------|------|---------|
| **Proxy-Installer 1.0.0** | 2026-09-05 `912da7b` | Standalone — adds `curl \| bash` one-liner (`marktantongco/owl-engine-proxy-installer`), PEP 668-aware header, WIIFY welcome (same instruction set as `heretic-installer` 1.1.0). Based on Installer **4.0.0** (828 lines, 12 phases). |
| **Engine 9.0.0** | 2026-08-29 `6add1da` | `pyproject.toml: version = "9.0.0"` — lazy imports, dead code removal, full test suite (78/78), coverage 92%, audit 41 fixes. |
| **Installer 4.0.0** | 2026-08-24 `3c6af54` + `20a9393` | Harden `set -e` arithmetic, xargs traps, quoting, `Pages index.html` generation. Pages at https://marktantongco.github.io/owl-dns-synergy/ |
| **Installer 4.0.0** | 2026-08-24 `dbb18a2` | Production-grade: `pages.yml` + `ci.yml`, worktree `CONTRIBUTING.md`, keysync (`AUTOCLAW_TOKEN_KEY`), Prometheus 12 gauges. |
| **Product 1.0.0** | 2026-08-06 `a13e2c2` | `owl-dns-synergy` v1.0.0 — Unified Dual-Channel Resilient Access Engine (OWL-AGENT v4.2 + LLM-DNS-Proxy, 7-channel cascade, Fernet, Circuit Breaker, `PRODUCT_VERSION="1.0.0"`). Tag `v1.0.0`. |

> Full installer changelog: [`CHANGELOG.md`](CHANGELOG.md) — upstream engine log: `~/aiworkspace/owl-engine` (`git log --oneline`)

---

## 📁 What's Inside This Repo

```
owl-engine-proxy-installer/
├── install.sh              # ← updated v4.0.0, PEP 668-aware, standalone curl-capable
├── README.md               # this file (WIIFY + beginner guide)
├── config.json.example     # copy of default config.json
├── .env.example            # copy of generated .env template
├── .gitignore
└── LICENSE                 # MIT
```

Upstream: `~/aiworkspace/owl-engine` (aka `owl-dns-synergy`) — 7-channel cascade, Fernet, Circuit Breaker, Prometheus.

Sibling installer: [`heretic-installer`](https://github.com/marktantongco/heretic-installer) — same instruction set, same style.

---

## 🔌 Integration — After Install

### CLI (main)
```bash
source ~/.owl-dns-synergy/venv/bin/activate
owl-dns-synergy --help
owl-dns-synergy fetch https://example.com --channel dns --verbose
owl-dns-synergy stats
owl-dns-synergy generate-key
```

### Python
```python
from openai import OpenAI
client = OpenAI(api_key="sk-...", base_url="http://localhost:31000/v1")  # AutoClaw proxy
client.chat.completions.create(model="gpt-4o", messages=[{"role":"user","content":"hi"}])
```

### Systemd (prod)
```bash
sudo cp ~/.owl-dns-synergy/deploy/*.service /etc/systemd/system/
sudo systemctl daemon-reload && sudo systemctl enable --now owl-dns-synergy autoclaw-proxy
```

---

## 🗑️ Uninstall

```bash
bash install.sh --uninstall
# or manual:
rm -rf ~/.owl-dns-synergy
sudo rm -rf /etc/owl-dns-synergy /etc/systemd/system/owl-dns-synergy.service /etc/systemd/system/autoclaw-proxy.service
sudo systemctl daemon-reload
```

---

## 🙏 Credits

- **owl-dns-synergy** — Unified Dual-Channel Resilient Access Engine (OWL-AGENT v4.2 + LLM-DNS-Proxy + 5 aux repos) — MIT — local at `~/aiworkspace/owl-engine`
- Installer & docs by this repo — MIT — adds PEP 668 handling + beginner WIIFY. Same instruction set as `heretic-installer`.

---

## 📬 Next Steps

1. **Star upstream** + sibling `heretic-installer`
2. Run `owl-dns-synergy fetch https://example.com --verbose` and check `owl-dns-synergy stats`
3. Open a PR if you improve the installer (PRs welcome!)

*Made for beginners, by someone who hit `externally-managed-environment` and decided to fix it once and for all — same fix as heretic-installer, now for the proxy stack.*
