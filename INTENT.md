# Intent — podmesh-meshfs

Provide an autonomous, high-performance 4 MB Content-Addressable Storage (CAS) distributed and Peer-to-Peer (P2P) filesystem and storage engine for Podman containers, Proxmox VE hypervisors, and Linux hosts.

## Role and Sovereign Architecture

`podmesh-meshfs` exists to eliminate the operational complexity, memory overhead, and quorum fragility of traditional distributed storage systems (such as Ceph/CephFS) across single-node, two-node, and multi-node sovereign infrastructures.

It delivers two complementary execution planes:
1. **Pure Rust Native Engine (`meshfs`)**: A single-binary (< 8 MB RAM) daemon and CLI featuring 4 MB SHA-256 content-addressable chunking, Merkle tree Copy-on-Write (CoW) snapshots, FUSE POSIX mounting (`fuser`), OpenRaft consensus for clustered metadata, kernel `FICLONE` zero-copy reflinks, Unix Domain Socket control IPC, and an HTTP REST API (`:7443`) for hypervisor and backup integration.
2. **Pure Peer-to-Peer Engine (`bin/meshfs-p2p`)**: A zero-central-server P2P replication engine operating over mutually authenticated LAN/SSH mesh topologies. It slices payloads into 4 MB CAS chunks, computes cryptographic SHA-256 Merkle roots, gossips chunks in parallel across peers without any central coordinator, and exposes deduplicated volumes directly to rootless or rootful Podman containers.

## Interface and Authority

`podmesh-meshfs` operates strictly as a storage and filesystem substrate:
- **Zero Ambient Authority**: Storing, replicating, or mounting volumes confers neither SHAPER governor jurisdiction nor cluster orchestration authority.
- **Explicit Topology Inputs**: Node addresses, peer lists, ports, and data directories are provided via configuration or environment variables; laboratory addresses are never treated as implicit authority.
- **Standalone or Integrated**: Operates standalone on any Linux or Proxmox VE host, or integrates with `podmesh`, `podmesh-pve-plugin`, and `podmesh-pmbs` (PodMesh Backup Server) via its canonical Merkle and chunk interfaces.

## Shared Source Route and Custody

The human–AI tandem carries intention and mandate. Read the [common composition](https://github.com/xavdp-pro/shaper-ecosystem/blob/main/chapters/10-COMPOSITION.md) and [glossary](https://github.com/xavdp-pro/shaper-glossary/blob/main/TERMS.md) to distinguish component, logical space, class, specialization, and instance.

The public repository `xavdp-pro/podmesh-meshfs` owns the generic product code, CLI, daemon, P2P engine, packaging, and architectural contracts. The private companion repository `xavdp-pro/podmesh-meshfs-lab` owns hardware qualification reports, P2P and NVMe benchmark campaigns, invariance proofs, and laboratory records.
