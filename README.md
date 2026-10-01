# Shayan Jewani

*Support Engineer → IAM / Security Engineering*

[homelab-infra](https://github.com/shay-devops/homelab-infra) · [msp-it-automation](https://github.com/shay-devops/msp-it-automation) · [LinkedIn](https://www.linkedin.com/in/shayan-jewani/)

---

> **Portfolio note**
>
> This homelab is a single, continuously evolving production-grade environment, not a set of isolated labs. Every piece below is a real, running system with a git history behind it: identity federation across four distinct auth mechanisms, least-privilege access control, network segmentation and intrusion detection, and full observability, all under infrastructure-as-code discipline.

**Focus areas:** identity federation (OIDC/SSO) · secrets management and least privilege · network security and detection · infrastructure as code · observability

**The build at a glance:** five VMs on a Proxmox host (HashiCorp Vault, a two-node k3s cluster, an Ansible/Terraform control node, and a documentation wiki with a password manager), behind a FortiGate 60E firewall, with a managed switch and a Raspberry Pi providing DNS and intrusion detection.

---

## Identity & Access Management

| Capability | What it demonstrates | Stack |
|---|---|---|
| [**Okta SSO federation across 4 services**](https://github.com/shay-devops/homelab-infra/blob/main/docs/okta-federation.md) | Four genuinely different integration mechanisms — config-file patch, environment variables, DB-registered via CLI, and a secrets manager's own native OIDC backend — not the same integration copy-pasted four times | Okta, OIDC, Grafana, BookStack, Gitea, Vault |
| [**Vault least-privilege ACL design**](https://github.com/shay-devops/homelab-infra/blob/main/docs/vault-access-control.md) | Human vs. machine identity separation (OIDC role vs. AppRole), explicit-deny hardening on rekey/raw-storage/root-policy paths, a deliberate decision to run no standing viewer tier on a secrets manager | HashiCorp Vault, HCL |
| **RBAC-scoped cluster access** | Kubeconfig built from a dedicated ServiceAccount + ClusterRole (get/list/watch only, secrets excluded) — boundary verified by testing a denied action, not just configuring one | Kubernetes RBAC |
| [**Vault dynamic secrets brokering**](https://github.com/shay-devops/homelab-infra/blob/main/docs/vault-access-control.md) | AppRole-based machine identity for Terraform/Ansible, scoped per-tool by blast radius, zero static secrets in any tracked file | Vault, AppRole, Terraform, Ansible |

A deliberate exclusion: the Vaultwarden password manager is kept out of SSO, because its encryption key is derived client-side from the master password, which is fundamentally incompatible with federation that grants access by login.

## Infrastructure & Platform

| Capability | What it demonstrates | Stack |
|---|---|---|
| **IaC provisioning pipeline** | Every VM provisioned via Terraform, every service configured via Ansible, secrets sourced from Vault — zero manual server builds | Terraform, Proxmox, Ansible |
| **k3s cluster + Helm/Kustomize deployments** | Vendored-and-patched manifests where drift-detection matters, live Helm releases where CRD lifecycle matters — the deployment pattern is chosen per workload, not applied uniformly | k3s, Helm, Kustomize |
| **Internal PKI (cert-manager)** | Self-signed root CA + ClusterIssuer chain, auto-issued/renewed leaf certs across every internal service — built specifically because Okta rejects non-HTTPS redirect URIs | cert-manager, ECDSA |
| **Full observability stack** | Metrics, logs (container + systemd journal), and uptime monitoring — every component resource-sized from real unpatched-default incidents, not guessed | Prometheus, Grafana, Loki, Alloy, Uptime Kuma |

## Network Security & Detection

| Capability | What it demonstrates | Stack |
|---|---|---|
| **Network segmentation** | Dedicated firewall appliance keeping the lab on its own routed network, separate from the home network, validated by confirming traffic from the home Wi-Fi side cannot reach the lab gateway | FortiGate 60E |
| **Network intrusion detection** | Suricata (ET Open rules) on a dedicated capture NIC fed by a switch port mirror, verified end to end with a generated test alert and zero dropped packets; Pi-hole provides LAN-wide DNS. Both are configured through Ansible roles, and the Suricata role validates its config before any restart | Suricata, Pi-hole, TP-Link managed switch, Ansible |

## Production MSP troubleshooting

[**msp-it-automation**](https://github.com/shay-devops/msp-it-automation) — real fixes written on the job, across a multi-tenant MSP environment (client identifiers scrubbed throughout). Different flavor of evidence than the homelab: this is operating inside existing, live client infrastructure under real constraints, not building a system from scratch.

- **Exchange Online DDL regression workaround** — an Azure Automation runbook with Managed Identity, stamping a custom attribute nightly to route around a domain-filter bug
- **Windows tray-icon pinning fix** — reverse-engineered the undocumented per-user pinning mechanism on both Windows 10 (binary registry blob) and Windows 11 (NotifyIconSettings) to keep an RMM agent icon visible fleet-wide
- **Driver-install failure fix** — root-caused a kernel driver service that silently never got created during a DNS filtering client install, deployed as an RMM script across the client base
- **OS-aware Teams deployment fix** — auto-detects Windows 10/11 vs. Server 2019/RDS to route around a bootstrapper install method that silently fails on server hosts

## On the roadmap

- SIEM (Wazuh) as the single correlation point for Suricata alerts and FortiGate logs (Suricata is already running)
- AI-assisted alert triage (Ollama, local inference)
- Off-site backup (AWS S3)

---

### A few real incidents from this build

- Diagnosed a Vault OIDC login failure to a hostname mismatch between what was registered in Okta and what was actually served — traced through a CLI-vs-UI redirect flow and a `localhost` loopback-listener/SSH mismatch before landing on the real root cause.
- Root-caused a cluster-wide DNS resolution failure for internal hostnames by reading the live Kubernetes ConfigMap directly rather than trusting the managed one, and fixed it with a scoped override that didn't touch default resolution behavior.
- Found and fixed a silent log-ingestion failure by verifying against a component's actual metric output instead of its reported health status — traced to an upstream default that silently restricted what it could read.
- Diagnosed Kubernetes control-plane distress (failing health checks, API timeouts) after a policy-engine teardown left dangling admission-webhook registrations pointing at a deleted service; removing them dropped the load average from about 3.5 to 0.2.
- Found a FortiGate DNS zone that the GUI showed as authoritative but the running config did not, confirmed it with the CLI, a packet sniffer and the DNS proxy debug trace, and fixed it from the CLI.

---

*Full commit history and technical detail behind every claim above: [homelab-infra](https://github.com/shay-devops/homelab-infra)*
