# Michael Olszewski

**PharmD, BCPS, BCCCP · DevOps Engineer · Clinical AI Developer · Charlotte, NC**
molszewski423@gmail.com

---

Twenty years in critical care and infectious disease gives you a particular perspective on clinical data  -  you learn quickly that the difference between a real safety signal and statistical noise can matter enormously. That experience is what led me to build my own clinical AI tooling rather than wait for commercial platforms to catch up. I led my organization to IDSA Antimicrobial Stewardship Center of Excellence designation, and somewhere along the way, a decades-long passion for Linux became the infrastructure that runs it all.

Everything here  -  the clinical AI platforms, the homelab infrastructure, the DevOps and cloud work  -  has been built entirely in my own time, outside of my traditional clinical pharmacist role. That is precisely the point. This is what I do because I find it genuinely compelling, and it is why I am actively looking for consulting and freelance opportunities where these skills can be put to use.

---

## Clinical AI

The pharmacovigilance and stewardship platforms below aren't side projects. They're tools I built to solve problems I ran into during clinical practice, running entirely on local hardware so patient data never leaves the machine.

**[pv-workbench](https://gitlab.com/molszewski423/pv-workbench)** brings the full pharmacovigilance workflow into a single local platform  -  PRR/ROR disproportionality analysis against FDA FAERS using the Evans criteria, RAG-powered querying over ICH/FDA/EMA regulatory guidelines, MedDRA coding assistance, ICSR narrative generation, and automated literature monitoring. The statistical methods are production-grade: continuity correction for sparse FAERS data, Yates' chi-squared correction, and artifact exclusion to prevent administrative PTs from drowning out real signals. Clinical interpretation runs through a locally-hosted 26B parameter model  -  no cloud API calls, no data leaving the machine.

**[ams-intelligence](https://gitlab.com/molszewski423/ams-intelligence)** applies the same disproportionality methodology to antimicrobial stewardship, aggregating resistance surveillance data from NHSN, WHONET, FAERS, PubMed, and the Pfizer ATLAS dataset. One design decision worth noting: confounding by indication is explicitly modeled throughout. Last-resort antibiotics treat the sickest patients in the building  -  a naive statistical read of the mortality signal will always mislead, and the platform is built to catch that. Includes an authenticated gateway portal for clinical team access.

---

## Infrastructure & Homelab

My homelab has been running in one form or another for decades. The current setup spans five machines networked via Tailscale with zero open inbound ports  -  all public traffic routes through a Cloudflare Tunnel.

### Hardware

| Machine | Role | Hardware | OS |
|---|---|---|---|
| **MikePC** | GPU workstation · Ollama inference | Ryzen 7 7800X3D · RTX 5060 Ti 16GB | Debian 13 |
| **archbox** | 24/7 server · 25-container agency pod | Intel i3-4130T · Alienware Alpha | Arch Linux |
| **MikeInspiron** | Dev laptop | Dell Inspiron | Debian 13 · Hyprland |
| **ThinkPad** | Incoming dev machine |  -  | Debian 13 (pending) |
| **debianbook** | Portable node | Samsung Chromebook Pro (Skylake) | Debian 13 · Sway |

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
 │  Ollama  :11434  (0.0.0.0  -  accessible to all tailnet nodes)       │
 │    └── qwen2.5:14b   ← agency orchestrator LLM fallback #2        │
 │    └── gemma4:26b    ← pv-workbench reasoning engine               │
 │    └── llama3.1:8b   ← fast inference tasks                        │
 │                                                                     │
 │  Nightly 2 AM  -  video generation pipeline                          │
 │    archbox triggers GPU job → MikePC renders YouTube Shorts        │
 │    25 niche rotation · auto-upload · 7-day cleanup                 │
 └─────────────────────────────────────────────────────────────────────┘

 LLM Routing Chain (agency orchestrator):
   1. Gemini 2.5 Flash  (primary  -  cloud, fast)
   2. Ollama qwen2.5:14b  (fallback #1  -  MikePC GPU via Tailscale)
   3. Groq llama-3.3-70b  (fallback #2  -  cloud)
   4. Groq llama-3.1-8b   (fallback #3  -  cloud, fastest)
```

### Where It's Heading

The homelab is mid-migration toward a hybrid cloud architecture. Public-facing RingCatch services are moving to AWS EC2 while internal automation stays on archbox  -  Tailscale bridges the two without VPC peering or open ports. Infrastructure is managed with Terraform, bootstrap automation with Ansible.

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

A k3s cluster across MikePC, archbox, and MikeInspiron is also in progress  -  GPU workloads pinned to MikePC via node labels, long-running services on archbox. All container workloads are designed Kubernetes-ready from day one so the same manifests will eventually target AWS EKS.

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
 │  Future: AWS EKS  -  same manifests, cloud node pool added       │
 └─────────────────────────────────────────────────────────────────┘
```

**[homelab-infra](https://gitlab.com/molszewski423/homelab-infra)** contains the full IaC  -  25 Podman quadlets, Ansible bootstrap playbook, Terraform for AWS provisioning, and network security configs for AdGuard, CrowdSec, and nftables.

**[chromebook-linux](https://gitlab.com/molszewski423/chromebook-linux)** documents the full Linux conversion of a Samsung Chromebook Pro, including hardware write protection removal, MrChromebox UEFI firmware flashing, Skylake audio DSP fixes, and the DRM atomic sleep workaround.

---

## Dotfiles & Desktop

**[dotfiles](https://gitlab.com/molszewski423/dotfiles)** covers the full Wayland desktop setup across Hyprland and Sway on Debian 13  -  Waybar, Mako, Wofi, Kitty, Fish, and everything needed for a working environment. An `install.sh` script sets up a new machine in one command, which has come in handy more than once.

---

## Linux Notes

**[linux-notes](https://gitlab.com/molszewski423/linux-notes)** is a running log of things worth remembering  -  distro trade-offs, driver fixes, networking gotchas, and the kind of hard-won knowledge that only comes from actually running into the problem.

Covers NixOS, Arch Linux, Fedora Kinoite, Debian 13, and the ChromeOS-to-Linux conversion path. Some entries worth reading: the RTX 5060 Ti Blackwell driver situation on both NixOS and Debian, why NixOS is excellent for stable servers but friction for AI/ML dev workstations, and the Tailscale/nftables conflict on Arch that took longer than it should have to diagnose.

---

## Stack

`Python` · `LangChain` · `ChromaDB` · `Ollama` · `Streamlit` · `FastAPI`
`Podman` · `Ansible` · `Terraform` · `k3s` · `Tailscale` · `nftables` · `CrowdSec`
`GitLab CI` · `Cloudflare` · `AWS EC2` · `Debian` · `Arch Linux` · `NixOS`
