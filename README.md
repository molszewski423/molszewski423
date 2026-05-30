# Michael Olszewski

**DevOps Engineer · AI Systems Developer**
Troutman, NC (Charlotte metro) · molszewski423@gmail.com

---

I've been tinkering with computers since MS-DOS and BBS days, long before the modern internet existed. Decades of Linux use as a passion finally found a productive outlet in DevOps and AI infrastructure — the homelab isn't a portfolio project, it's just how I've always worked.

---

## Infrastructure & Homelab

Self-hosted homelab running 24/7 across three machines — a dev workstation (RTX 5060 Ti), a low-power always-on server (i3-4130T), and a converted Chromebook. All networked via Tailscale mesh with zero open inbound ports.

**[homelab-infra](https://gitlab.com/molszewski423/homelab-infra)** — 25-service Podman pod (Arch Linux), AdGuard Home DNS filtering, CrowdSec intrusion detection with nftables bouncer, Cloudflare Tunnel, Ansible bootstrap playbook. Previously ran on a Raspberry Pi 4 before migrating to x86.

**[chromebook-linux](https://gitlab.com/molszewski423/chromebook-linux)** — Full Linux conversion of a Samsung Chromebook Pro: hardware write protection removal, MrChromebox UEFI firmware flash, Skylake audio DSP kernel parameter fix, DRM atomic sleep workaround. Sway + Tokyo Night desktop.

---

## AI Projects

**[pv-workbench](https://gitlab.com/molszewski423/pv-workbench)** — Pharmacovigilance signal intelligence platform. Pulls from FDA FAERS, WHO VigiBase, PubMed, and NHSN. Detects adverse event signals and drug resistance trends. Python + Streamlit + ChromaDB.

**[ams-intelligence](https://gitlab.com/molszewski423/ams-intelligence)** — Clinical AI for antimicrobial stewardship. Tracks resistance trends across NHSN, WHONET, FAERS, and ATLAS. Containerized, GitLab CI with lint + container build pipeline.

**[ams-gateway](https://gitlab.com/molszewski423/ams-gateway)** — Auth portal for AMS Intelligence. Role-based access, bcrypt credentials, Streamlit.

---

## Dotfiles & Desktop

**[dotfiles](https://gitlab.com/molszewski423/dotfiles)** — Complete Wayland desktop config for Debian 13. Hyprland + Sway + Waybar + Mako + Wofi + Fish. Includes `install.sh` for one-command setup on a new machine.

---

## Linux Notes

**[linux-notes](https://gitlab.com/molszewski423/linux-notes)** — A running log of distros, problems solved, and things worth remembering.

Distros used: **NixOS** · **Arch Linux** · **Fedora Kinoite** · **Debian 13** · **ChromeOS → Linux**

Notable entries:
- RTX 5060 Ti (Blackwell) driver setup on NixOS and Debian — `open = true` is mandatory, closed module doesn't support the architecture
- Tailscale exit node + nftables on Arch — kernel module conflicts and fixes
- Fedora Kinoite rootless Podman — SELinux `:Z` volume label gotcha
- NixOS trade-off analysis — why it's excellent for servers but friction for AI/ML dev workstations

---

## Stack

`Podman` · `Ansible` · `Tailscale` · `nftables` · `CrowdSec` · `AdGuard`
`Python` · `FastAPI` · `Streamlit` · `GitLab CI` · `Cloudflare`
`Hyprland` · `Sway` · `Fish` · `Debian` · `Arch Linux` · `NixOS`

---

*Kubernetes homelab cluster in progress — k3s across MikePC (RTX 5060 Ti) + archbox + MikeInspiron*
