# Michael Olszewski

**PharmD, BCPS, BCCCP · DevOps Engineer · AI Systems Developer · Charlotte, NC**
molszewski423@gmail.com

*20 years in critical care and infectious disease. Led my organization to IDSA Antimicrobial Stewardship Center of Excellence designation. Linux since before the web existed. Building clinical AI infrastructure that I can actually trust because I understand both sides of it.*

---

## Clinical AI Projects

These aren't demo projects — they solve real problems from 20 years of clinical practice.

**[pv-workbench](https://gitlab.com/molszewski423/pv-workbench)** — Pharmacovigilance signal intelligence platform built for PV consulting work. PRR/ROR disproportionality analysis with Evans criteria against FDA FAERS, RAG over ICH/FDA/EMA regulatory guidelines, MedDRA coding, ICSR narrative generation, and literature monitoring. Runs entirely local on an RTX 5060 Ti — no data leaves the machine.

**[ams-intelligence](https://gitlab.com/molszewski423/ams-intelligence)** — Clinical AI for antimicrobial stewardship programs, built on 20 years of ID/AMS practice and the experience of leading an organization to IDSA Antimicrobial Stewardship Center of Excellence designation. Aggregates NHSN, WHONET, FAERS, PubMed, and ATLAS. Detects resistance trends and adverse drug event signals with PRR signal detection. Confounding by indication is explicitly modeled — because last-resort antibiotics treat the sickest patients and a naive statistical read will always mislead.

**[ams-gateway](https://gitlab.com/molszewski423/ams-gateway)** — Authenticated portal for AMS Intelligence. Role-based access for clinical teams.

---

## Infrastructure & Homelab

### Hardware

| Machine | Role | Hardware | OS |
|---|---|---|---|
| **MikePC** | GPU workstation · Ollama inference server | Ryzen 7 7800X3D · RTX 5060 Ti 8GB | Debian 13 |
| **archbox** | 24/7 home server · 25-container agency pod | Intel i3-4130T · Alienware Alpha | Arch Linux |
| **MikeInspiron** | Dev laptop | Dell Inspiron | Debian 13 · Hyprland |
| **ThinkPad** | Incoming dev machine | — | Debian 13 (pending) |
| **debianbook** | Portable node | Samsung Chromebook Pro (Skylake) | Debian 13 · Sway |

All machines networked via **Tailscale** mesh (WireGuard). Zero open inbound ports on the home router — all public ingress via **Cloudflare Tunnel** on archbox.

---

### Current Architecture

```
 ┌─────────────────────────────────────────────────────────────────────┐
 │                        PUBLIC INTERNET                              │
 └──────────────────────────┬──────────────────────────────────────────┘
                            │ HTTPS
                    ┌───────▼───────┐
                    │  Cloudflare   │
                    │   Network     │
                    └───────┬───────┘
                            │ Tunnel (outbound-only)
 ┌──────────────────────────▼──────────────────────────────────────────┐
 │  archbox  ·  Arch Linux  ·  Intel i3-4130T  ·  24/7                │
 │                                                                     │
 │  ┌─────────────────────────────────────────────────────────────┐   │
 │  │  agency-pod  (Podman, rootless, systemd quadlets)           │   │
 │  │                                                             │   │
 │  │  agency-landing :8090  ←── ringcatch.io                    │   │
 │  │  agency-outreach :8080  ←── /api/chat  (Alex chatbot)      │   │
 │  │  agency-orchestrator :8109  ←── AI brain · 22 tools        │   │
 │  │  agency-scraper :8079   ←── Google Maps lead scraper       │   │
 │  │  agency-billing :8082   ←── Stripe webhooks                │   │
 │  │  agency-command :8100   ←── dashboard.ringcatch.io         │   │
 │  │  agency-discord :8103   ←── Discord bot bridge             │   │
 │  │  agency-n8n     :5678   ←── workflow automation            │   │
 │  │  + 17 more services                                         │   │
 │  └──────────────────┬──────────────────────────────────────────┘   │
 │                     │                                               │
 │  AdGuard Home :53   │  DNS filtering · network-wide ad/tracker     │
 │  CrowdSec           │  Intrusion detection · SSH brute force ban   │
 │  nftables           │  Default-drop firewall · CrowdSec ban sets   │
 └─────────────────────┼───────────────────────────────────────────────┘
                       │ Tailscale (WireGuard mesh)
 ┌─────────────────────▼───────────────────────────────────────────────┐
 │  MikePC  ·  Debian 13  ·  Ryzen 7 7800X3D  ·  RTX 5060 Ti        │
 │                                                                     │
 │  Ollama  :11434  (0.0.0.0 — accessible to all tailnet nodes)       │
 │    └── qwen2.5:14b   ← agency orchestrator LLM fallback #2        │
 │    └── gemma4:26b    ← pv-workbench reasoning engine               │
 │    └── llama3.1:8b   ← fast inference tasks                        │
 │                                                                     │
 │  Nightly 2 AM — video generation pipeline                          │
 │    archbox triggers GPU job → MikePC renders YouTube Shorts        │
 │    25 niche rotation · auto-upload · 7-day cleanup                 │
 └─────────────────────────────────────────────────────────────────────┘

 LLM Routing Chain (agency orchestrator):
   1. Gemini 2.5 Flash  (primary — cloud, fast)
   2. Ollama qwen2.5:14b  (fallback #1 — MikePC GPU via Tailscale)
   3. Groq llama-3.3-70b  (fallback #2 — cloud)
   4. Groq llama-3.1-8b   (fallback #3 — cloud, fastest)
```

---

### AWS Hybrid Migration (In Progress)

Moving public-facing sales services to AWS EC2 while keeping internal automation on archbox. Tailscale bridges home and cloud — no VPC peering, no open ports.

```
 ┌──────────────────────────────────────────────────────────────────┐
 │                       PUBLIC INTERNET                            │
 └──────────────────────────┬───────────────────────────────────────┘
                            │
                    ┌───────▼───────┐
                    │  Cloudflare   │
                    └───────┬───────┘
                            │ Tunnel
 ┌──────────────────────────▼───────────────────────────┐
 │  AWS EC2  ·  t3.small  ·  Ubuntu 22.04               │
 │                                                       │
 │  agency-landing   (nginx static)                      │
 │  agency-outreach  (chatbot + email engine)            │
 │  agency-billing   (Stripe webhooks)                   │
 │  agency-postgres  (PostgreSQL 16)                     │
 │  cloudflared      (Cloudflare Tunnel daemon)          │
 │                                                       │
 │  Infrastructure as Code: Terraform (homelab-infra)    │
 └──────────────────────┬───────────────────────────────┘
                        │ Tailscale
 ┌──────────────────────▼───────────────────────────────┐
 │  archbox  (internal services stay home)               │
 │  orchestrator · scraper · n8n · discord · sales …    │
 └──────────────────────┬───────────────────────────────┘
                        │ Tailscale
 ┌──────────────────────▼───────────────────────────────┐
 │  MikePC  ·  Ollama GPU inference                      │
 └───────────────────────────────────────────────────────┘
```

---

### Kubernetes Target (k3s — This Weekend)

```
 ┌─────────────────────────────────────────────────────────────────┐
 │                    k3s Cluster                                  │
 │                                                                 │
 │  ┌─────────────────┐  ┌──────────────┐  ┌──────────────────┐  │
 │  │     MikePC      │  │   archbox    │  │  MikeInspiron    │  │
 │  │ control plane   │  │   worker     │  │    worker        │  │
 │  │   + worker      │  │              │  │                  │  │
 │  │  RTX 5060 Ti    │  │  24/7 uptime │  │  24/7 capable    │  │
 │  │  gpu=true label │  │  always-on   │  │  low power       │  │
 │  └─────────────────┘  └──────────────┘  └──────────────────┘  │
 │                                                                 │
 │  Node labels drive workload placement:                          │
 │    gpu=true        → AI inference pods land on MikePC          │
 │    always-on=true  → Long-running services land on archbox     │
 │                                                                 │
 │  Networking: Tailscale between nodes (existing mesh)           │
 │  Future: AWS EKS — same manifests, cloud node pool added       │
 └─────────────────────────────────────────────────────────────────┘
```

All container workloads designed to be Kubernetes-ready from day one — environment variables only, no shared filesystems, health checks on every service, resource limits set. The Podman Compose files translate directly to Kubernetes manifests.

---

**[homelab-infra](https://gitlab.com/molszewski423/homelab-infra)** — Full IaC: 25 Podman quadlets, Ansible bootstrap playbook, Terraform AWS provisioning, AdGuard/CrowdSec/nftables network security configs.

**[chromebook-linux](https://gitlab.com/molszewski423/chromebook-linux)** — Full Linux conversion of a Samsung Chromebook Pro: hardware write protection removal, MrChromebox UEFI firmware flash, Skylake audio DSP kernel parameter fix, DRM atomic sleep workaround.

---

## Dotfiles & Desktop

**[dotfiles](https://gitlab.com/molszewski423/dotfiles)** — Complete Wayland desktop config for Debian 13. Hyprland + Sway + Waybar + Mako + Wofi + Fish. Includes `install.sh` for one-command setup on a new machine.

---

## Linux Notes

**[linux-notes](https://gitlab.com/molszewski423/linux-notes)** — A running log of distros, problems solved, and things worth remembering.

Distros used: **NixOS** · **Arch Linux** · **Fedora Kinoite** · **Debian 13** · **ChromeOS → Linux**

Notable entries:
- RTX 5060 Ti (Blackwell) — `open = true` is mandatory, closed module doesn't support the architecture
- Tailscale exit node + nftables on Arch — kernel module conflicts and fixes
- Fedora Kinoite rootless Podman — SELinux `:Z` volume label gotcha
- NixOS trade-off analysis — excellent for servers, friction for AI/ML dev workstations
- Hybrid cloud architecture — Podman homelab to AWS EC2 via Tailscale bridge

---

## Stack

`Python` · `LangChain` · `ChromaDB` · `Ollama` · `Streamlit` · `FastAPI`
`Podman` · `Ansible` · `Terraform` · `k3s` · `Tailscale` · `nftables` · `CrowdSec`
`GitLab CI` · `Cloudflare` · `AWS EC2` · `Debian` · `Arch Linux` · `NixOS`
