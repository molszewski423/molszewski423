# Michael Olszewski

**PharmD, BCPS, BCCCP · DevOps Engineer · Clinical AI Developer · Charlotte, NC**
molszewski423@gmail.com

---

Twenty years in critical care and infectious disease gives you a particular perspective on clinical data - you learn quickly that the difference between a real safety signal and statistical noise can matter enormously. That experience is what led me to build my own clinical AI tooling rather than wait for commercial platforms to catch up. I led my organization to IDSA Antimicrobial Stewardship Center of Excellence designation, and somewhere along the way, a decades-long passion for Linux became the infrastructure that runs it all.

Everything here - the clinical AI platforms, the homelab infrastructure, the DevOps and cloud work - has been built entirely in my own time, outside of my traditional clinical pharmacist role. That is precisely the point. This is what I do because I find it genuinely compelling, and it is why I am actively looking for consulting and freelance opportunities where these skills can be put to use.

The technical foundation goes back further than most. At 10 years old I was writing BASIC, cracking open computers to learn how the hardware worked, and dialing into BBS services on a 286 running MS-DOS - years before the web existed. The command line, the hardware, the problem-solving at the system level - none of it has ever felt foreign. What has changed is that four decades of accumulated instinct now has a real and productive outlet.

---

## Clinical AI

### [pv-workbench](https://gitlab.com/molszewski423/pv-workbench)

Pharmacovigilance platform for signal detection, MedDRA coding, ICSR narrative drafting, and literature monitoring. Built around a tiered LLM stack: `gemma4:26b` for regulatory Q&A and signal interpretation, `gemma4:e4b` for ICSR narrative generation, and `qwen2.5:7b` for fast intent routing. PRR/ROR disproportionality analysis against FDA FAERS using Evans criteria - with continuity correction for sparse data, Yates' chi-squared correction, and artifact exclusion. RAG over ICH/FDA/EMA guidelines via ChromaDB. All inference runs on a local RTX 5060 Ti via Ollama - no patient data leaves the machine. Deployed as a k3s pod at `http://pv.lan`.

**[argus-bot](https://gitlab.com/molszewski423/pv-workbench)** - Discord bot (Argus#1432) providing mobile access to pv-workbench for signal queries, case lookups, and narrative review.

### [ams-intelligence](https://gitlab.com/molszewski423/ams-intelligence)

Antimicrobial stewardship platform surfacing resistance trends, utilization signals, and clinical decision support. Backed by two decades of hands-on AMS program leadership including IDSA Center of Excellence designation. Confounding by indication is explicitly modeled - last-resort antibiotics treat the sickest patients in the building; the platform is built to catch that bias. Deployed at `http://ams.lan` on the same k3s cluster.

---

## RingCatch Agency

### [ringcatch-agency](https://gitlab.com/molszewski423/ringcatch-agency) · [ringcatch.io](https://ringcatch.io)

AI chatbot agency serving US local SMBs - $450 setup + $89/month. Fully automated pipeline from lead scraping through booking, onboarding, and client success. Twenty-four services running in Kubernetes on archbox, migrated from Podman systemd quadlets on 2026-05-31.

**LLM routing:** Gemini 2.5 Flash → Ollama gemma4:26b (k3s GPU pod on MikePC) → Groq llama-3.3-70b → Groq llama-3.1-8b

Public via Cloudflare tunnel: `ringcatch.io`, `dashboard.ringcatch.io`

---

## Infrastructure & Homelab

### Hardware

| Machine | Role | Hardware | OS | LAN |
|---|---|---|---|---|
| **MikePC** | k3s control plane + GPU node | RTX 5060 Ti 16 GB | Debian 13 | 192.168.4.54 |
| **archbox** | k3s worker · 24/7 server | Intel i3-4130T | Arch Linux | 192.168.4.46 |
| **MikeInspiron** | k3s worker · 24/7 lid-closed | Dell Inspiron | Debian 13 | 192.168.4.33 |
| **ThinkPad** | Remote daily driver | - | Debian 13 | - |
| **debianbook** | Samsung Chromebook Pro | Skylake | Debian 13 · Sway | - |

### Current Architecture (deployed 2026-05-31)

```
                         INTERNET
                             |
                    [Cloudflare Tunnel]
                      (no open ports)
                             |
┌────────────────────────────────────────────────────────────────────┐
│  k3s CLUSTER  (LAN: 192.168.4.x, k3s v1.35)                       │
│                                                                     │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │  MikePC  192.168.4.54  RTX 5060 Ti  (control plane + GPU) │   │
│  │                                                             │   │
│  │  namespace: ai                                              │   │
│  │  ├─ ollama [GPU]  gemma4:26b · gemma4:e4b · qwen3:30b     │   │
│  │  │               qwen2.5:7b · nomic-embed-text             │   │
│  │  ├─ pv-workbench      → http://pv.lan                      │   │
│  │  ├─ ams-intelligence  → http://ams.lan                     │   │
│  │  ├─ argus-bot         → Discord (Argus#1432)               │   │
│  │  └─ traefik ingress                                         │   │
│  └────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │  archbox  192.168.4.46  i3-4130T  (24/7 worker)           │   │
│  │                                                             │   │
│  │  namespace: agency  (24 services)                          │   │
│  │  ├─ orchestrator · outreach · scraper · landing            │   │
│  │  ├─ command · discord · billing · legal · marketing        │   │
│  │  ├─ support · success · bi · sales · cfo                   │   │
│  │  ├─ inbox · delivery · video · dashboard                   │   │
│  │  ├─ n8n · calcom · kokoro · voice · tunnel                 │   │
│  │  └─ postgresql-16 (hostPath PVC)                           │   │
│  └────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │  MikeInspiron  192.168.4.33  (24/7 lid-closed worker)      │   │
│  └────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

LLM Routing (agency):
  Gemini 2.5 Flash → Ollama gemma4:26b (k3s GPU pod) → Groq llama-3.3-70b → Groq llama-3.1-8b

LLM Routing (pv-workbench):
  REASON_MODEL  gemma4:26b   signal detection, regulatory Q&A, MedDRA
  DRAFT_MODEL   gemma4:e4b   ICSR narratives, digests
  CHAT_MODEL    qwen2.5:7b   intent routing, Discord general chat
```

### Where It's Going

Cluster is production-stable. Next phase is AWS hybrid: stateless public-facing services (landing, chatbot, webhook handlers) move to AWS free-tier EC2 while stateful workloads, LLM inference, and clinical AI stay on-prem. Tailscale bridges the two without open ports or VPC peering. Same Kubernetes manifests, new node pool.

### [homelab-infra](https://gitlab.com/molszewski423/homelab-infra)

Full IaC for the cluster: k3s manifests for both namespaces (ai + agency), NVIDIA device plugin setup, Traefik ingress, GitLab CI/CD pipelines. Migrated 24 agency services from Podman quadlets to k3s on 2026-05-31. See **[k3s-homelab](https://gitlab.com/molszewski423/k3s-homelab)** for the full day-one build documentation including lessons learned.

---

## Dotfiles & Linux

**[dotfiles](https://gitlab.com/molszewski423/dotfiles)** - Fish, Hyprland, Kitty, Neovim, Starship, Waybar across Debian and Arch. `install.sh` sets up a new machine in one command.

**[linux-notes](https://gitlab.com/molszewski423/linux-notes)** - Running log of things worth remembering: hardware quirks, driver fixes, networking gotchas. RTX 5060 Ti Blackwell on Debian, Tailscale/nftables interaction on Arch, i915 firmware dimming, Chromebook UEFI.

---

## Stack

`Python` · `FastAPI` · `LangChain` · `ChromaDB` · `Ollama` · `Streamlit`
`Kubernetes / k3s` · `Traefik` · `GitLab CI/CD` · `Podman` (image builds)
`Cloudflare Tunnel` · `Tailscale` · `nftables` · `CrowdSec` · `AdGuard Home`
`PostgreSQL` · `n8n` · `Terraform` · `AWS EC2`
`Debian 13` · `Arch Linux` · `Hyprland` · `Fish`
