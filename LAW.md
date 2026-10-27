# Law — podmesh-meshfs

> *"Look at what the modern Linux kernel already offers natively and harness it without intermediaries."* — Shaper OS Law

The following architectural invariants govern `podmesh-meshfs` (v2.3.0) across its Rust daemon, FUSE filesystem, REST API, and P2P replication engine. Every change must preserve these laws without exception.

## 1. 4 MB FastCDC / SHA-256 Content-Addressable Verification
- All stored payloads are segmented into bounded chunks (up to 4 MiB / `4 * 1024 * 1024` bytes).
- Every chunk is addressed exclusively by its lowercase hexadecimal SHA-256 digest (`<sha256>.chunk`).
- Any chunk read from disk or received from a remote peer must match its expected SHA-256 digest before being committed to the CAS pool or served to a reader. Mismatched chunks are rejected immediately.

## 2. Atomic Temp-to-Final Rename
- No chunk, manifest, or metadata file is ever written directly in-place to its canonical path when concurrent readers or crash recovery could observe a partial write.
- Staging writes must complete and flush to temporary files on the same filesystem before atomic POSIX `rename()` into the canonical CAS or metadata directory.

## 3. Zero Data Corruption & Merkle Root Integrity
- Every logical file, volume, or snapshot is sealed by a cryptographic SHA-256 Merkle root over its ordered sequence of chunk digests and metadata.
- Snapshot creation, replication, and volume assembly are bit-exact: reassembling a file or volume from its Merkle manifest must yield a byte-for-byte identical stream with zero silent drift.

## 4. Strict POSIX Attribute Fidelity
- When mounted via FUSE or materialized into local volume trees, `podmesh-meshfs` preserves POSIX file modes, ownership (`uid`/`gid`), timestamps, file sizes, and directory hierarchies accurately.
- Truncation, unlinking, atomic rename, and `fsync` requests from guest VMs or containers are honored deterministically before reporting completion to the caller.

## 5. P2P Gossip & Cluster Convergence
- In P2P mode (`meshfs-p2p`) and clustered OpenRaft mode (`meshfs`), chunk dissemination is idempotent because chunks are immutable and content-addressed.
- Any peer missing a chunk referenced by a valid Merkle manifest can pull that chunk from any available peer holding it, converging on identical CAS state without a central point of failure.
- Public code never hardcodes secret credentials; peer authentication relies on externally provisioned Ed25519 SSH keys or scoped cluster tokens.
