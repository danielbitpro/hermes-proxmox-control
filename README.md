# Hermes Agent Skill: Proxmox VE Control

A skill for [Hermes Agent](https://github.com/nousresearch/hermes-agent) that enables autonomous control of Proxmox VE hypervisors via the REST API.

**Capabilities depend on the Proxmox user/token permissions.** The skill uses whatever the authenticated user is allowed to do — you control what Hermes can and cannot do by configuring the Proxmox token role.

## What This Skill Can Do

### ✅ Currently Working (tested)

| Capability | Details |
|------------|---------|
| **VM status monitoring** | List all VMs and containers, check running/stopped status, CPU, memory |
| **VM power management** | Start, stop, graceful shutdown, reboot VMs |
| **System power management** | Start/stop the hypervisor node itself (requires appropriate permissions) |

### 🔄 Planned / Permission-Dependent

| Capability | Status |
|------------|--------|
| Snapshot creation, rollback, cleanup | Planned — requires snapshot permissions on the Proxmox token |
| Resource monitoring (CPU, disk I/O) | Available if the token has `PVEAuditor` or broader role |
| Storage management | Available if the token has storage permissions |
| Template cloning | Available if the token has VM.Config permissions |

> **Key point:** Proxmox tokens are scoped. A token with only `PVEVMAdmin` can manage VMs but not snapshots or storage. A token with `PVEAdmin` has full access. Configure the token role on your Proxmox host to match what you want Hermes to be able to do.

## Quick Start

### 1. Install the Skill

```bash
hermes skills install danielbitpro/hermes-proxmox-control
```

Or download and place in `~/.hermes/skills/proxmox-control/`.

### 2. Install Dependencies

```bash
pip install proxmoxer
```

### 3. Configure Authentication

Add credentials to `~/.hermes/.env`:

**Option A: Token-based (Recommended)**

```bash
PROXMOX_HOST=192.168.1.100
PROXMOX_PORT=8006
PROXMOX_TOKEN_ID=youruser@realm!tokenname
PROXMOX_TOKEN_SECRET=your-token-secret
```

**Option B: Username/Password**

```bash
PROXMOX_HOST=192.168.1.100
PROXMOX_PORT=8006
PROXMOX_USER=youruser@pve
PROXMOX_PASSWORD=yourpassword
```

### 4. Create API Token (if using Option A)

On your Proxmox host:

```bash
# Full admin access (can do everything)
pvesh create /access/tokens -userid root@pam -privsep 0 -description "Hermes"
pveum aclmod / -u root@pam -role PVEAdmin

# Or create a VM-only token (limited scope)
pvesh create /access/tokens -userid root@pam -privsep 0 -description "Hermes"
pveum aclmod / -u root@pam -role PVEVMAdmin
```

## Usage Examples

### Monitor All VMs

> "Check the status of all my VMs"

The skill discovers node names dynamically and lists all VMs with their status, CPU, and memory usage.

### Start/Stop a VM

> "Shut down VM 103 gracefully" or "Start VM 104"

Performs power operations with safety checks — confirms with user first and monitors task completion.

### Resource Monitoring

> "What's using the most memory on my Proxmox node?"

Lists all VMs with CPU and memory metrics. Works if the token has `PVEAuditor` role or broader.

## Safety Features

- ⚠️ **Confirmation required** before any power operations (VM or system)
- 📋 **Status check** before shutdown/reboot
- ⏱️ **Task monitoring** — waits for operations to complete
- 🚫 **LXC protection** — doesn't modify containers unless explicitly requested

## Token Permissions Guide

| Role | What Hermes Can Do |
|------|-------------------|
| `PVEAdmin` | Everything: VMs, containers, snapshots, storage, node management |
| `PVEVMAdmin` | VM power management, status, config — but not snapshots or storage |
| `PVEAuditor` | Read-only: status, monitoring, storage info — no modifications |
| `PVEDBSuperuser` | Backup/restore operations (if configured) |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| 401 Unauthorized | Try different realm (`@pam` vs `@pve`) or check token role assignments |
| Token secret fails | Editing a token in Proxmox UI regenerates the secret — copy the new one |
| Node name errors | Never hardcode `'pve'` — always discover dynamically via `proxmox.nodes.get()` |
| Connection refused | Verify `PROXMOX_HOST` is reachable on port 8006 |
| "permission denied" on operations | Check the token's role in Proxmox UI → Permissions → ACLs |

## Architecture

```
Hermes Agent
    ↓ (skill: proxmox-control)
Python script (proxmoxer library)
    ↓ (HTTPS, port 8006)
Proxmox VE REST API
    ↓
VMs / Containers / Storage / Snapshots
    (operations limited by token role)
```

## License

MIT
