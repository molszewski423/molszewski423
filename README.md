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

Four-machine homelab networked via Tailscale mesh — all running Linux, all 24/7 capable:

| Machine | Role | Hardware |
|---|---|---|
| **MikePC** | GPU workstation, Ollama inference | Ryzen 7 7800X3D · RTX 5060 Ti · Debian 13 |
| **archbox** | 24/7 server, 25-container agency pod | Intel i3-4130T · Arch Linux |
| **MikeInspiron** | Dev laptop | Dell Inspiron · Debian 13 · Hyprland |
| **ThinkPad** | Incoming dev machine | Debian 13 (pending setup) |
| **debianbook** | Portable Linux node | Samsung Chromebook Pro · Debian 13 · Sway |

**[homelab-infra](https://gitlab.com/molszewski423/homelab-infra)** — 25-service Podman pod (Arch Linux), AdGuard Home DNS filtering, CrowdSec intrusion detection with nftables bouncer, Cloudflare Tunnel, Ansible bootstrap playbook, Terraform for AWS provisioning. Previously ran on a Raspberry Pi 4.

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
`Podman` · `Ansible` · `Terraform` · `Tailscale` · `nftables` · `CrowdSec`
`GitLab CI` · `Cloudflare` · `Debian` · `Arch Linux` · `NixOS`

---

*Kubernetes homelab cluster in progress — k3s across RTX 5060 Ti workstation + archbox + Dell Inspiron*
