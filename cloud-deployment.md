---
title: Cloud Deployment (Trusted Agents)
layout: default
nav_order: 9
---

# Cloud Deployment (Trusted Agents)

This guide provides a practical starting point to run a trusted compute agent in common cloud environments with a focus on security, reproducibility, and low operational overhead.

The network operates a single community AI service; trusted agents help keep it reliable. If you prefer to participate permissionlessly, see the [Compute Agent](/compute-agent) page instead.

---

## Baseline Requirements

- Hardened OS image with automatic security updates
- Outbound-only network access (ingress restricted)
- Authenticated channel to orchestrators
- Signed agent descriptor and minimal configuration
- Observability: metrics and logs shipped to your account

---

## Common Cloud Patterns

The details vary by provider, but the pattern is consistent:

1. Create a minimal instance profile/role for the agent (no broad admin rights)
2. Provision a VM or container with a hardened base image
3. Inject agent credentials (short-lived where possible)
4. Run the agent as a non-root user with resource limits
5. Expose only required egress; deny inbound by default
6. Enable automatic upgrades and restarts

---

## Example: Generic VM (Ubuntu/Debian)

1. System prep:
   - Enable unattended-upgrades
   - Create user `fa-agent` (no-login shell)
   - Configure a firewall to deny inbound, allow required egress
2. Install agent:
   - Download the signed release binary
   - Verify signature and checksum
   - Place binary in `/usr/local/bin/compute-agent`
3. Configure:
   - Write `/etc/compute-agent/config.yaml` with resource limits and allowed operators
   - Add issued credentials to `/etc/compute-agent/credentials.json` (600 perms)
4. Service unit:
   - Create a systemd unit running as `fa-agent`, with `MemoryMax`, `CPUQuota`, and restart policy
5. Observability:
   - Configure journald forwarding or a lightweight log shipper

---

## Example: Kubernetes

- Run as a Deployment with a single replica per node (DaemonSet if desired)
- Use a restricted PodSecurityContext (drop capabilities, read-only rootfs)
- Mount a Secret for credentials; rotate it periodically
- Apply CPU/memory requests and limits
- Use a NetworkPolicy to restrict egress to orchestrator endpoints

---

## Credentials & Rotation

- Bind credentials to your operator identity
- Prefer short-lived tokens; rotate on a schedule and on incident
- On credential misuse, revoke trust and recycle the node

---

## Upgrades

- Use canary upgrades across your fleet
- Keep two known-good versions available for fast rollback
- After upgrade, run a self-check operator set sanity test

---

## Security Checklist

- [ ] Non-root execution
- [ ] Ingress disabled by default
- [ ] Egress restricted to orchestrators and monitoring
- [ ] Signed binary verified at boot/deploy
- [ ] Secrets at rest with least privilege access
- [ ] Automatic security updates enabled

---

## Next Steps

- Complete the [Trusted Registration](/trust-and-validation#registration-initial-proposal)
- Deploy a small pilot and monitor validation rates
- Reach out to the community for peer review of your setup
