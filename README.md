# Hermes Agent Skill: Proxmox VE Control

A skill for [Hermes Agent](https://github.com/nousresearch/hermes-agent) that enables autonomous control of Proxmox VE hypervisors via the REST API.

## Features

- ✅ VM/Container status monitoring
- ✅ Power management (start, stop, shutdown, reboot)
- ✅ Snapshot creation, rollback, and cleanup
- ✅ Resource monitoring (CPU, memory, disk)
- ✅ Storage management
- ✅ Template cloning and conversion

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
pvesh create /access/tokens -userid youruser@pve -privsep 0 -description "Hermes"
```

## Usage Examples

### Monitor All VMs

> "Check the status of all my VMs"

The skill discovers node names dynamically and lists all VMs with their status, CPU, and memory usage.

### Create a Snapshot

> "Create a snapshot of VM 100 named 'before_update'"

Creates a named snapshot. Always create snapshots before risky operations.

### Start/Stop a VM

> "Shut down VM 103 gracefully" or "Start VM 104"

Performs power operations with safety checks — confirms with user first and monitors task completion.

### Resource Monitoring

> "What's using the most memory on my Proxmox node?"

Lists all VMs sorted by memory usage with CPU and disk metrics.

## Safety Features

- ⚠️ **Confirmation required** before any power operations
- 📋 **Status check** before shutdown/reboot
- 📸 **Snapshot recommendation** before risky operations
- ⏱️ **Task monitoring** — waits for operations to complete
- 🚫 **LXC protection** — doesn't modify containers unless explicitly requested

## Troubleshooting

| Problem | Solution |
|---------|----------|
| 401 Unauthorized | Try different realm (`@pam` vs `@pve`) or check token role assignments |
| Token secret fails | Editing a token in Proxmox UI regenerates the secret — copy the new one |
| Node name errors | Never hardcode `'pve'` — always discover dynamically via `proxmox.nodes.get()` |
| Connection refused | Verify `PROXMOX_HOST` is reachable on port 8006 |

## Architecture

```
Hermes Agent
    ↓ (skill: proxmox-control)
Python script (proxmoxer library)
    ↓ (HTTPS, port 8006)
Proxmox VE REST API
    ↓
VMs / Containers / Storage / Snapshots
```

## License

MIT
