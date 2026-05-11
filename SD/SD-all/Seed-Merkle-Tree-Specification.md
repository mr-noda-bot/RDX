# =============================================================
# Seed Merkle Tree Specification (v1)
# =============================================================
# Purpose:
#   Define exactly which artifacts constitute the Seed's
#   structural identity and how the Merkle root is computed.
#   This root represents lineage, not meaning or authority.
# =============================================================

version: 1

# -----------------------------
# 1. Design Principles
# -----------------------------
principles:
  - minimal_surface: only artifacts required for origin identity
  - determinism: identical inputs must yield identical roots
  - immutability: all inputs are read-only after computation
  - opacity: contents are not interpreted by downstream layers
  - forward_compatibility: future nodes may be appended, never reordered

# -----------------------------
# 2. Hash Function
# -----------------------------
hash:
  algorithm: sha256
  encoding: hex

# -----------------------------
# 3. Canonical Ordering Rules
# -----------------------------
ordering:
  - all leaf nodes sorted lexicographically by node_id
  - no timestamps influence ordering
  - byte-for-byte file content hashing (no normalization)

# -----------------------------
# 4. Merkle Node Types
# -----------------------------
nodes:
  # ---- Core Identity Artifacts ----
  - node_id: seed_binary
    type: file
    path: /opt/seed/seed.gforth
    required: true
    description: "Exact gforth seed script executed"

  - node_id: seed_binary_hash
    type: value
    source: seed_binary
    description: "Hash of seed.gforth for redundancy"

  - node_id: hardware_identity
    type: file
    path: /var/lib/seed/identity.bin
    required: true
    description: "Hardware signing output"

  - node_id: seed_provenance
    type: file
    path: /var/lib/seed/provenance.seed
    required: true
    description: "Seed execution provenance"

  # ---- Constitutional Constraints ----
  - node_id: seed_constitution
    type: file
    path: /opt/seed/constitution.yaml
    required: false
    description: "Seed constitution text, if externalized"

  # ---- Environment Snapshot ----
  - node_id: os_fingerprint
    type: value
    description: "OS identifier string (e.g. ubuntu-22.04)"
    source: /etc/os-release

  # ---- Explicit Exclusions ----
exclusions:
  - cloud-init artifacts
  - core bootstrap script
  - network-derived data
  - timestamps beyond seed execution record
  - runtime-generated files

# -----------------------------
# 5. Leaf Hash Construction
# -----------------------------
leaf_hash:
  file:
    method: sha256(file_bytes)
  value:
    method: sha256(utf8_bytes)

# -----------------------------
# 6. Internal Node Construction
# -----------------------------
internal_node:
  method: sha256(left_child_hash || right_child_hash)

# -----------------------------
# 7. Merkle Root Semantics
# -----------------------------
root:
  name: SEED_MERKLE_ROOT
  meaning:
    - represents complete seed state at handoff
    - invariant under replication of identical seeds
    - changes if and only if a constituent artifact changes
  prohibitions:
    - must not be recomputed by Core or later layers
    - must not be used as policy input
    - must not be interpreted semantically

# -----------------------------
# 8. Evolution Rules
# -----------------------------
evolution:
  allowed:
    - append new optional nodes with new node_id
  forbidden:
    - reordering existing nodes
    - removing existing nodes
    - changing hash algorithms

# -----------------------------
# 9. Failure Semantics
# -----------------------------
failures:
  missing_required_node:
    effect: "Seed aborts before handoff"
  hash_mismatch:
    effect: "Seed aborts before handoff"

# -----------------------------
# 10. Invariant Statement
# -----------------------------
invariant:
  statement: >
    The Seed Merkle Root uniquely identifies the origin state of the system.
    It is a commitment to structure, not intent, policy, or meaning.