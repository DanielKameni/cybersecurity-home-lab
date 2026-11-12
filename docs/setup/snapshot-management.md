# Snapshot Management & Recovery Strategy

## 1. Overview

This document describes the snapshot management strategy implemented in the Cybersecurity Home Lab.

Snapshots are used to preserve known virtual machine states and support reproducible security testing.

The lab currently contains two baseline snapshots:

- Baseline-Kali
- Baseline-Metasploitable

These snapshots provide recovery points for the attacker and target virtual machines.

## 2. Snapshot Inventory

| Virtual Machine | Snapshot Name | Purpose | Status |
|---|---|---|---|
| Kali Linux | Baseline-Kali | Preserve attacker workstation configuration | Created |
| Metasploitable 2 | Baseline-Metasploitable | Preserve vulnerable target configuration | Created |

Both snapshots were created using Oracle VirtualBox.

## 3. Snapshot Creation

Before creating the snapshots, both virtual machines were shut down.

The running VM list was checked using:

```bash
VBoxManage list runningvms
```

No running virtual machines were reported.

### 3.1 Kali Linux Snapshot

```bash
VBoxManage snapshot "kali-linux-2026.2-virtualbox-amd64" take "Baseline-Kali" \
  --description "Baseline configuration before security assessment activities"
```

### 3.2 Metasploitable Snapshot

```bash
VBoxManage snapshot "Metasploitable2" take "Baseline-Metasploitable" \
  --description "Isolated vulnerable target baseline before penetration testing"
```

## 4. Snapshot Verification

Snapshot availability was verified using the VirtualBox command-line interface.

### Kali Linux

```bash
VBoxManage snapshot "kali-linux-2026.2-virtualbox-amd64" list
```

**Result:** Baseline-Kali exists.

### Metasploitable 2

```bash
VBoxManage snapshot "Metasploitable2" list
```

**Result:** Baseline-Metasploitable exists.

Both snapshot existence checks were successful.

## 5. Recovery Procedure

Snapshots can be used to restore a virtual machine to a previously saved state.

**Warning:** Restoring a snapshot discards the current VM state and returns the machine to the selected snapshot state. Any changes made after the snapshot may be lost.

### 5.1 Restore Kali Linux

First, shut down the virtual machine.

Then execute:

```bash
VBoxManage snapshot "kali-linux-2026.2-virtualbox-amd64" restore "Baseline-Kali"
```

### 5.2 Restore Metasploitable

First, shut down the virtual machine.

Then execute:

```bash
VBoxManage snapshot "Metasploitable2" restore "Baseline-Metasploitable"
```

These commands are documented for future recovery operations.

They have not yet been executed as part of a recovery verification test.

## 6. Snapshot Management Guidelines

The following practices are recommended:

1. Create baseline snapshots before security testing.
2. Use descriptive snapshot names.
3. Record significant configuration changes.
4. Restore target machines when a clean testing environment is required.
5. Monitor snapshot storage consumption.
6. Maintain independent backups of important VM configurations and project files.

## 7. Limitations

VirtualBox snapshots are not equivalent to independent backups.

Snapshots depend on the underlying virtual machine files and storage infrastructure.

A successful snapshot creation does not automatically guarantee successful recovery.

Recovery testing should therefore be performed before relying on snapshots for critical restoration procedures.

## 8. Conclusion

Baseline snapshots have been created and verified for both virtual machines.

The snapshot management strategy improves the reproducibility and maintainability of the Cybersecurity Home Lab.

A practical recovery verification test remains a future improvement.
