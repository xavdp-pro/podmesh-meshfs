# AI-First Installation, Cluster Join & 4MB CAS Configuration Protocol — `podmesh-meshfs` (v2.3.0)

> **Core Directive for Autonomous AI Agents**:
> `podmesh-meshfs` is a **100% headless, AI-First distributed 4 MB SHA-256 Content-Addressable Storage (CAS) engine**.
> It requires **zero web UI** and zero Ceph/Java/Erlang dependencies.
> AI agents deploy, configure, cluster, mount, snapshot, and audit `meshfs` exclusively via CLI subcommands, JSON membership manifests, Unix Domain Sockets (`/var/lib/meshfs/meshfs.sock`), and HTTP JSON endpoints (`127.0.0.1:7443`).

---

## 1. Operational Invariants for AI Agents

1. **Rule 0G (Zero Mocks)**: Every volume, chunk, and snapshot operation must execute against the real `meshfs` binary and `/var/lib/meshfs` datastore.
2. **Rule 38 (Bounded Retry $k \le 2$)**: At most 2 automated retry attempts on transient network/mount errors; halt fail-closed with JSON diagnostics on the second failure.
3. **4 MB CAS Law (`LAW.md`)**: Every ingested file is split into up to 4 MB chunks (`4,194,304` bytes), hashed with SHA-256, and written atomically via temp-file + `rename` under `/var/lib/meshfs/chunks/<prefix>/<sha256>.chunk`.
4. **Shared CAS Pool with PMBS**: When deployed alongside `podmesh-pmbs`, both engines share the identical 4 MB SHA-256 chunk format, enabling Zero-Transit backups.

---

## 2. Step-by-Step AI Installation

### 2.1 System Prerequisites (Debian 13 / Proxmox VE 9.x)

```bash
export DEBIAN_FRONTEND=noninteractive
apt-get update -qq
apt-get install -y -qq fuse3 curl jq ca-certificates python3
```

### 2.2 Build & Install Binary

From the `podmesh-meshfs` repository root:

```bash
cargo build --release
install -m 0755 target/release/meshfs /usr/local/bin/meshfs
install -m 0755 bin/meshfs-p2p /usr/local/bin/meshfs-p2p
meshfs --version
```

*(Alternatively, build and install the Debian package via `./packaging/build-deb-meshfs.sh` and `dpkg -i`.)*

---

## 3. Declarative Daemon & Cluster Configuration

### 3.1 Directory Layout

```bash
install -d -m 0700 /var/lib/meshfs
install -d -m 0700 /var/lib/meshfs/chunks
install -d -m 0700 /var/lib/meshfs/volumes
install -d -m 0750 /etc/meshfs
```

### 3.2 Standalone Mode vs. Fixed Cluster Membership

#### Mode A — Standalone Single-Node Operation
No external config file is needed. The daemon starts with defaults:
- Data directory: `/var/lib/meshfs`
- Control Unix Socket: `/var/lib/meshfs/meshfs.sock`
- HTTP REST API: `127.0.0.1:7443`

#### Mode B — Multi-Node Cluster Membership (`/etc/meshfs/cluster.json`)
For multi-node replication (e.g., across Proxmox VE or PodMesh nodes over LAN/WireGuard), write `/etc/meshfs/cluster.json` (`chmod 0600`):

```json
{
  "cluster_name": "meshfs-cluster",
  "self_node_id": "node-a",
  "self_address": "10.10.10.210:7443",
  "shared_token": "REPLACE_WITH_32BYTE_HEX_SECRET",
  "nodes": [
    { "id": "node-a", "address": "10.10.10.210:7443" },
    { "id": "node-b", "address": "10.10.10.211:7443" }
  ]
}
```

### 3.3 Systemd Service (`/etc/systemd/system/meshfs.service`)

```ini
[Unit]
Description=PodMesh MeshFS 4MB CAS Distributed Storage Daemon (v2.3.0)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/meshfs daemon \
  --port 7443 \
  --data-dir /var/lib/meshfs \
  --cluster meshfs-cluster \
  --socket /var/lib/meshfs/meshfs.sock
Restart=on-failure
RestartSec=2
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
systemctl daemon-reload
systemctl enable --now meshfs.service
```

---

## 4. AI-Driven CLI & Socket Operations (JSON Output)

All `meshfs` CLI commands return structured JSON to `stdout` and use exit code `0` on success.

### 4.1 Health & Status Probes

```bash
# Fast IPC ping via Unix Domain Socket (/var/lib/meshfs/meshfs.sock)
meshfs ping

# Full node & cluster status (chunk count, physical CAS bytes, logical bytes, dedup ratio)
meshfs status
```

### 4.2 Volume Provisioning

```bash
# Create a logical volume (e.g. for Proxmox VE or Podman workloads)
meshfs volume create pve-shared

# List all volumes
meshfs volume list
```

### 4.3 Storing, Retrieving & Snapshotting Data

```bash
# Ingest a file into 4MB SHA-256 CAS chunks
meshfs put --volume pve-shared vm-9501-disk-0.raw /path/to/local/file.raw

# Create an O(1) immutable Merkle CoW snapshot
meshfs snapshot --volume pve-shared --name snap-pre-upgrade

# Retrieve and verify a file from CAS chunks
meshfs get --volume pve-shared vm-9501-disk-0.raw --output /tmp/restored.raw
```

### 4.4 Deep Cryptographic Bit-Rot Audit (`inspect`)

AI agents should run `meshfs inspect --deep` periodically or prior to critical migrations to re-hash every 4 MB chunk on disk against its SHA-256 filename and Merkle manifest:

```bash
meshfs inspect --deep
```

Expected healthy JSON output:

```json
{
  "status": "HEALTHY",
  "corrupted_chunks": 0,
  "missing_chunks": 0,
  "integrity_verified": true
}
```

### 4.5 Joining a Peer Node Dynamically

```bash
meshfs join 10.10.10.210:7443 --token "$MESHFS_CLUSTER_TOKEN"
```

---

## 5. Active-Active P2P Directory Mirror (`meshfs-p2p`)

When mounting a shared POSIX directory across hypervisor nodes (such as `/mnt/pve/shared-meshfs` on Proxmox VE):

```bash
# Start P2P active-active CAS mirror daemon between node-a and node-b
/usr/local/bin/meshfs-p2p \
  --watch-dir /mnt/pve/shared-meshfs \
  --cas-dir /var/lib/meshfs/cas \
  --bind 0.0.0.0:7444 \
  --peers 10.10.10.211:7444
```

---

## 6. HTTP REST API Contract (For Local Adapters & Plugins)

| Method & Endpoint | Purpose |
| :--- | :--- |
| `GET /api/v1/meshfs/ping` | Liveness check (`{"pong": true, "engine": "meshfs-rust"}`) |
| `GET /api/v1/meshfs/status` | Full storage metrics, CAS chunk count, dedup ratio, peer list |
| `GET /api/v1/meshfs/inspect` | Deep SHA-256 chunk and Merkle tree verification |
| `GET /api/v1/meshfs/volumes` | List volumes and byte usage |
| `POST /api/v1/meshfs/volumes` | Create volume (`{"name": "..."}`) |
| `POST /api/v1/meshfs/snapshots` | Create instant Merkle CoW snapshot (`{"volume": "...", "name": "..."}`) |
| `POST /api/v1/meshfs/snapshots/delete` | Delete snapshot (`{"volume": "...", "snapshot": "..."}`) |
