# Changelog — owl-engine-proxy-installer

All notable changes to the **proxy installer** (not upstream `owl-dns-synergy` / `owl-engine`). Format: Keep a Changelog + SemVer.

Upstream logs: `~/aiworkspace/owl-engine` (`git log --oneline`), tags `v1.0.0` + engine `9.0.0`.

---

## [1.0.0] - 2026-09-05

### Added
- Initial standalone installer repo — same instruction set as [`heretic-installer`](https://github.com/marktantongco/heretic-installer) (WIIFY welcome, beginner table, 12-phase logs)
- `install.sh` — **v4.0.0** unified installer (828 lines, copied from `~/aiworkspace/owl-engine/install.sh:1`), patched for standalone `curl | bash`:
  - Header: `curl -fsSL https://raw.githubusercontent.com/marktantongco/owl-engine-proxy-installer/main/install.sh | bash`
  - PEP 668-aware note — venv-first (`~/.owl-dns-synergy/venv`) is already compliant; added `uv`/`pipx` fallback docs
  - 12 phases: `1/12` prerequisites (Python 3.10+, pip, git, Go 1.21+, Node 18+, Rust) → `2/12` dirs → `3/12` venv+deps (`-e .`, httpx/dnslib/circuitbreaker) → `4/12` AutoClaw (flask/gunicorn/eventlet) → `5/12` `LLM_PROXY_KEY` → `6/12` `AUTOCLAW_TOKEN_KEY` → `7/12` `config.json` → `8/12` `.env` (600) → `9/12` examples → `10/12` systemd → `11/12` verify (10 critical + 3 optional imports) → `12/12` summary
- `README.md` — WIIFY (7-channel cascade, Fernet, Circuit Breaker), Quick Start (curl / clone / full-repo), What You Need, Step-by-Step (7 steps), What It Does (12 phases), Troubleshooting (PEP 668 → use venv), What's Inside, Integration, Uninstall
- `config.json.example` (387B) + `.env.example` (template) extracted from `install.sh:397`
- `LICENSE` (MIT) + `.gitignore` (Python + Owl)

### Changed
- `install.sh:10` `REPO_DIR` now handles standalone curl (falls back to `pip install owl-dns-synergy` if `pyproject.toml` absent)
- README one-liner points to `https://raw.githubusercontent.com/marktantongco/owl-engine-proxy-installer/main/install.sh`

---

## Upstream timeline (context)

| Date | Upstream | Installer | Note |
|------|----------|-----------|------|
| 2026-08-06 | **owl-dns-synergy v1.0.0** `a13e2c2` | — | Unified Dual-Channel Resilient Access Engine (OWL-AGENT v4.2 + LLM-DNS-Proxy + 5 aux, 7-channel cascade, Fernet, CB, Prometheus) |
| 2026-08-24 | — | **Installer 4.0.0** `dbb18a2` | Production-grade: Pages, CI (`pages.yml`, `ci.yml`), worktree docs, keysync |
| 2026-08-24 | — | **Installer 4.0.0** `3c6af54` + `20a9393` | Harden `set -e` arithmetic, xargs traps, quoting + `Pages index.html` |
| 2026-08-29 | **owl-engine 9.0.0** `6add1da` | — | Lazy imports, dead code removal, full test suite (`pyproject.toml: version = "9.0.0"`) |
| 2026-09-05 | — | **Proxy-Installer 1.0.0** `912da7b` | This repo — standalone, PEP 668-aware header, WIIFY, same instruction set as `heretic-installer` 1.1.0 |

---

## Roadmap

- [ ] `install.sh --with-uv` flag to use `uv venv` instead of `python3 -m venv` (faster)
- [ ] Auto-detect `OPENAI_API_KEY` from `~/heretic` or `opencode` config
- [ ] CI: test `curl | bash` on `ubuntu:24.04` + `python:3.14-slim`
