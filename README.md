# podmesh-meshfs (v2.4.0)

> *"Look at what the modern Linux kernel already offers natively and harness it without intermediaries."* — Shaper OS Law  
> **Universal Multi-Platform (Proxmox VE + TrueNAS + Synology + Windows + Podman) 4 MB Content-Addressable & Distributed Storage Engine.**  
> *Sovereign alternative to CephFS: atomic chunk reception, FastCDC / BLAKE3, Merkle Copy-on-Write snapshots, FUSE POSIX mounting, and WireGuard Dual-Hub P2P replication without Single Point of Failure (SPOF).*

---

## Technical Documentation & Guides

- **[🤖 AI-First Installation & Configuration (`AI_INSTALL_AND_CONFIG.md`)](AI_INSTALL_AND_CONFIG.md)**: Deterministic headless installation, cluster bootstrap, Unix socket IPC, and 4 MB CAS operations for AI agents (zero web UI required).
- **[Genesis Story](GENESIS.md)**: How solving immutable container backup in PMBS birthed MeshFS.
- **[Use Cases & Benchmarks](USE_CASES.md)**: FUSE Workstation mount, Rootless Podman, LXC containers, and live NVMe/WireGuard benchmarks.
- **[Proxmox VE Integration](PROXMOX.md)**: Complete guide to using MeshFS as a shared storage plugin for KVM virtual machines and LXC on Proxmox VE 8.x/9.x.

---

## Overview

`podmesh-meshfs` is the dedicated distributed storage and CAS filesystem engine of the PodMesh and SHAPER PHOENIX ecosystem. Transitioning from the experimental v2.3.0 demonstrator to **v2.4.0 universal multi-storage architecture**, it operates across heterogeneous storage fabrics including Proxmox VE, TrueNAS, Synology, Windows, and Linux/Podman container hosts.

### Key Capabilities

- **Universal Multi-Platform Support**: Seamless integration across Proxmox VE hypervisors, TrueNAS CORE/SCALE, Synology DSM, Windows Server / Hyper-V, and bare-metal Linux Podman nodes.
- **Atomic Chunk Ingestion & FastCDC / BLAKE3**: Content-Defined Chunking (FastCDC) with BLAKE3 cryptographic verification and atomic staging, eliminating write tearing and achieving wire-speed throughput.
- **4 MB Content-Addressable Storage (CAS)**: Automatic chunk-level deduplication across volumes, virtual machine disks, container rootfs trees, and backups.
- **Cryptographic Merkle Tree Snapshots**: Instantaneous Copy-on-Write (CoW) snapshots sealed by a cryptographic Merkle root guaranteeing bit-exact integrity.
- **Dual Execution Modes**:
  1. **Native Rust Daemon (`meshfs`)**: High-performance Rust binary (< 8 MB RAM) providing a FUSE POSIX filesystem (`fuser`), OpenRaft consensus (`openraft` + `rusqlite`), kernel `FICLONE` zero-copy reflinks, Unix Domain Socket IPC, and an HTTP REST API on port `7443`.
  2. **Pure Peer-to-Peer Engine (`bin/meshfs-p2p`)**: Zero-central-server P2P chunk replication over mutually authenticated WireGuard Dual-Hub mesh topology without SPOF.
- **Native Proxmox VE & PMBS Integration**: Pairs seamlessly with [`podmesh-pve-plugin`](https://github.com/xavdp-pro/podmesh-pve-plugin) for shared KVM/LXC storage and with PodMesh Backup Server (PMBS) for zero-rechunking incremental backups.

---

## Architecture

```text
+-----------------------------------------------------------------------+
|                       podmesh-meshfs (v2.4.0)                         |
+-----------------------------------+-----------------------------------+
|     Pure Rust Daemon (meshfs)     |    Standalone P2P (meshfs-p2p)    |
|                                   |                                   |
|  - FUSE POSIX Mount (src/fuse.rs) |  - WireGuard Dual-Hub (No SPOF)   |
|  - REST API :7443 (src/server.rs) |  - FastCDC / BLAKE3 & 4MB CAS     |
|  - OpenRaft Log (src/cluster.rs)  |  - Atomic Chunk Reception Engine  |
|  - Merkle CoW (src/merkle.rs)     |  - Multi-Platform Storage Fabrics |
|  - FICLONE Reflink (src/engine.rs)|  - Podman Volume Export           |
+-----------------------------------+-----------------------------------+
                                    |
                                    v
+-----------------------------------------------------------------------+
|             Canonical 4 MB SHA-256 CAS Pool & Merkle Store            |
|          /var/lib/meshfs/chunks/<sha256>.chunk & meta/<root>.json     |
+-----------------------------------------------------------------------+
```

### Source Layout

| Path | Description |
| :--- | :--- |
| `Cargo.toml` | Rust package manifest (`meshfs` v2.3.0) |
| `src/main.rs` | Binary entrypoint and runtime dispatcher |
| `src/cli.rs` | Command-line interface parser (`clap`) |
| `src/engine.rs` | Core storage engine, atomic writes, and `FICLONE` reflink execution |
| `src/chunk.rs` | 4 MB content-addressable chunking and SHA-256 verification |
| `src/merkle.rs` | Cryptographic Merkle tree construction and snapshot verification |
| `src/fuse.rs` | FUSE POSIX filesystem implementation (`fuser` ABI 7.31) |
| `src/cluster.rs` | OpenRaft distributed consensus and peer replication |
| `src/server.rs` | Axum HTTP REST API server (`:7443`) |
| `src/control.rs` | Unix Domain Socket (`meshfs.sock`) control plane |
| `src/live.rs` | Live migration and streaming volume synchronization |
| `src/storage.rs` | Local disk layout and SQLite metadata persistence |
| `src/volume.rs` | Logical volume lifecycle and quota management |
| `bin/meshfs-p2p` | Standalone Peer-to-Peer CAS replication engine for LAN clusters & Podman |
| `packaging/build-deb-meshfs.sh` | Debian package builder (`podmesh-meshfs_2.3.0_amd64.deb`) |

---

## Building & Installation

### 1. Compile the Rust Engine

```bash
cargo build --release
```

The compiled binary is generated at `target/release/meshfs`.

### 2. Build the Debian Package (`.deb`)

```bash
./packaging/build-deb-meshfs.sh
sudo dpkg -i dist/podmesh-meshfs_2.3.0_amd64.deb
```

This installs `/usr/bin/meshfs`, `/etc/meshfs/config.json`, and enables the `meshfs.service` systemd unit listening on port `7443`.

---

## CLI Usage

### Native Daemon (`meshfs`)

```bash
# Start the MeshFS daemon
meshfs daemon --port 7443 --data-dir /var/lib/meshfs --socket /var/lib/meshfs/meshfs.sock

# Mount a MeshFS volume via FUSE
meshfs mount --data-dir /var/lib/meshfs --mountpoint /mnt/pve/meshfs

# Inspect cluster & storage status
meshfs status --socket /var/lib/meshfs/meshfs.sock
```

### Standalone Peer-to-Peer Mode (`bin/meshfs-p2p`)

`meshfs-p2p` enables direct machine-to-machine CAS replication across a LAN mesh without any central server:

```bash
# Install on each peer node
sudo install -m 0755 bin/meshfs-p2p /usr/local/bin/meshfs-p2p

# Ingest a file into 4 MB CAS chunks and replicate in parallel to all peers
meshfs-p2p write /path/to/payload.raw volumes/vm-100-disk-0.raw

# Reassemble a file on any peer from its SHA-256 Merkle root (pulling missing chunks via P2P)
meshfs-p2p read <merkle_root_sha256> /path/to/restored.raw

# Mount CAS chunks directly inside a Podman container
podman run --rm -v /var/lib/meshfs-pool/cas_chunks:/data:ro alpine ls -la /data
```

---

## HTTP REST API (`:7443`)

The MeshFS daemon exposes a JSON REST API consumed by `podmesh`, [`podmesh-pve-plugin`](https://github.com/xavdp-pro/podmesh-pve-plugin), and `podmesh-pmbs`:

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/api/v1/meshfs/status` | Returns node health, cluster state, total/free/used bytes, and chunk statistics. |
| `GET` | `/api/v1/meshfs/volumes` | Lists registered logical volumes and their active Merkle roots. |
| `POST` | `/api/v1/meshfs/volumes` | Declares or updates a logical volume in the MeshFS store. |
| `GET` | `/api/v1/meshfs/snapshots` | Lists immutable Merkle CoW snapshots across volumes. |
| `POST` | `/api/v1/meshfs/snapshots` | Creates an instantaneous Merkle tree snapshot for a volume. |
| `POST` | `/api/v1/meshfs/snapshots/delete` | Deletes a snapshot and garbage-collects unreferenced chunks. |
| `GET` | `/api/v1/meshfs/chunks/:sha256` | Retrieves and verifies a 4 MB CAS chunk by its SHA-256 digest. |
| `PUT` | `/api/v1/meshfs/chunks/:sha256` | Uploads a 4 MB CAS chunk with mandatory SHA-256 verification. |

---

## Governance & Companion Repositories

- **Intent & Mandate**: See [`INTENT.md`](INTENT.md)
- **Architectural Invariants**: See [`LAW.md`](LAW.md)
- **Extraction Lineage**: See [`LINEAGE.md`](LINEAGE.md)
- **Human–AI Tandem Instructions**: See [`AGENTS.md`](AGENTS.md)
- **Dedication**: See [`DEDICATION.md`](DEDICATION.md)
- **Qualification Lab**: [`xavdp-pro/podmesh-meshfs-lab`](https://github.com/xavdp-pro/podmesh-meshfs-lab) (private)

## License

Licensed under the Apache License, Version 2.0. See [`LICENSE`](LICENSE) for details.
