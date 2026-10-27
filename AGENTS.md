# Agents — podmesh-meshfs

> **🤖 AI-First Installation & Configuration Mandate**:
> `podmesh-meshfs` is 100% AI-First and headless-native. Zero web UI is required to install, cluster, mount, snapshot, or audit storage.
> 👉 **Autonomous AI Agents: Read [`AI_INSTALL_AND_CONFIG.md`](AI_INSTALL_AND_CONFIG.md) first for deterministic CLI, Unix Socket (`/var/lib/meshfs/meshfs.sock`), and systemd configuration protocols.**

`podmesh-meshfs` lives in two repositories and one vault under the SHAPER PHOENIX governance model. Every human–AI tandem (Claude Code, Codex, Cursor, Antigravity, or any other agent) follows this strict separation. Dialogue with the operator may be in French; everything committed into either repository must be written in English (`en-US`).

| Where | What belongs there | What never goes there |
| --- | --- | --- |
| **`xavdp-pro/podmesh-meshfs`** (this repository, public) | The product: Rust crate (`src/`, `Cargo.toml`), standalone P2P engine (`bin/meshfs-p2p`), Debian packaging (`packaging/`), architecture and CLI/API documentation, canonical governance (`INTENT.md`, `LAW.md`, `LINEAGE.md`, `AGENTS.md`, `DEDICATION.md`) | Laboratory campaign narratives, hardware qualification logs, private hostnames/IPs as mandatory defaults, secrets or private keys |
| **`xavdp-pro/podmesh-meshfs-lab`** (private companion) | The maintainers' workshop: bare-metal and VM qualification reports, P2P replication proofs, PMBS/MeshFS invariance benchmarks, cycle records (`meshfs-*`), lab topology maps | Signing keys, tokens, credentials, large binary disk images |
| **The vault** | Signing keys, SSH private keys, cluster tokens, credentials | — |

## Mandatory Rules for Agents

1. **Preserve `LAW.md` Invariants**: Never weaken 4 MB SHA-256 chunk verification, atomic temp-to-final file writes, Merkle root verification, POSIX attribute fidelity, or P2P convergence.
2. **Public vs. Lab Split**: Product code and generic documentation go here (`xavdp-pro/podmesh-meshfs`). Qualification evidence, timing benchmarks, and lab-specific incident/cycle reports go to `xavdp-pro/podmesh-meshfs-lab`.
3. **Build & Verification**:
   - Check Rust compilation with `cargo check` and `cargo test`.
   - Build release binary with `cargo build --release`.
   - Build Debian package with `./packaging/build-deb-meshfs.sh`.
4. **Cross-Component Alignment**: When changing the HTTP REST API (`/api/v1/meshfs/*`) or snapshot semantics, verify compatibility with [`xavdp-pro/podmesh-pve-plugin`](https://github.com/xavdp-pro/podmesh-pve-plugin) and `podmesh-pmbs`.
