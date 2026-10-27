# Lineage — podmesh-meshfs

## Extraction & Modular Separation — v2.3.0 (2026-10-27)

- **Extracted Version**: `2.3.0`
- **Extraction Date**: `2026-10-27`
- **Upstream Source**: Extracted from [`xavdp-pro/podmesh`](https://github.com/xavdp-pro/podmesh) (`crates/meshfs/` and `packaging/meshfs/`) together with the qualified standalone P2P engine (`bin/meshfs-p2p`) following the **SHAPER PHOENIX** modular separation architecture.
- **Companion Lab Repository**: [`xavdp-pro/podmesh-meshfs-lab`](https://github.com/xavdp-pro/podmesh-meshfs-lab) (private)
- **Related Ecosystem Components**:
  - [`xavdp-pro/podmesh`](https://github.com/xavdp-pro/podmesh) — Sovereign container and VM orchestration platform.
  - [`xavdp-pro/podmesh-pve-plugin`](https://github.com/xavdp-pro/podmesh-pve-plugin) — Native Proxmox VE storage plugin (`PVE::Storage::Custom::MeshFSPlugin`) backed by `podmesh-meshfs`.
  - [`xavdp-pro/podmesh-pmbs`](https://github.com/xavdp-pro/podmesh-pmbs) — PodMesh Backup Server sharing the 4 MB SHA-256 CAS chunk invariance contract.

## Historical Milestones

1. **v1.x — Initial CAS & Merkle Store**: Prototype content-addressable chunk store and snapshot engine inside the monolithic PodMesh tree.
2. **v2.1.0 — Pure Rust Daemon, OpenRaft & FUSE**: Full Rust implementation (`crates/meshfs`) combining `axum` HTTP API (`:7443`), `fuser` POSIX mount, `rusqlite` + `openraft` crash-durable state, and zero-copy `FICLONE` reflinks.
3. **v2.3.0 — SHAPER PHOENIX Autonomous Extraction & P2P Engine**:
   - Extracted into dedicated repository `xavdp-pro/podmesh-meshfs` (`v2.3.0`).
   - Integrated `bin/meshfs-p2p` for zero-central-server Peer-to-Peer 4 MB CAS replication and direct Podman volume consumption.
   - Separated Proxmox VE Perl storage integration into `xavdp-pro/podmesh-pve-plugin`.
