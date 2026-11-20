# RISE Build Farm & CI Infrastructure — Current Status & Server Reference

> **Last Updated:** 2026-11-20
> **Bi-Weekly Progress Log:** [`2026_weekly_progress.md`](2026_weekly_progress.md)
> **Sister Hardware Testbed (OSU-OSL):** [RISC-V Developer Cloud (`riseproject-dev/developer-cloud`)](https://github.com/riseproject-dev/developer-cloud) — see its [`current_status.md`](../developer-cloud/current_status.md) and [`2026_weekly_progress.md`](../developer-cloud/2026_weekly_progress.md)
>
> **Internal details:** Addresses, budgets, contracts and meeting recordings are not published here. RISE members can find them in the RISE shared drive or ask the WG maintainers. Build Farm host details are in `RISE Build Farm Guide for Administrators (Hetzner Instances)`.

This living document is the single source of truth for the current server inventory, active CI workloads, runner architecture, and supporting services of the **RISE Build Farm** and **Developer Tooling Working Group** (which consolidated the **Developer Infrastructure** and **Simulator / Emulator** workstreams in 2026).

---

## 1. Quick Links, Dashboards & Repositories

| Resource / Project | URL | Purpose |
| :--- | :--- | :--- |
| **WG Charter & Docs Repo** | [github.com/riseproject-dev/developer-infrastructure-wg](https://github.com/riseproject-dev/developer-infrastructure-wg) | Build Farm documentation, server inventory, and bi-weekly WG progress log |
| **RISC-V GitHub Runner Fleet** | [github.com/riseproject-dev/riscv-runner](https://github.com/riseproject-dev/riscv-runner) | Kubernetes control plane, Ansible worker playbooks, and RISC-V SoC device plugin for native `riscv64` runners |
| **RISC-V Developer Cloud (Board Farm)** | [github.com/riseproject-dev/developer-cloud](https://github.com/riseproject-dev/developer-cloud) | Physical RISC-V board farm at OSU-OSL (Alibaba Zhihe A210, SpacemiT K3) + [Live Telemetry Dashboard](https://riseproject-dev.github.io/developer-cloud/hub/static/) |
| **GCC Post-Commit CI & Dashboard** | [github.com/riseproject-dev/gcc-postcommit-ci](https://github.com/riseproject-dev/gcc-postcommit-ci) | Automated GCC post-merge CI & [Live Status Dashboard](https://riseproject-dev.github.io/gcc-postcommit-ci/) |
| **GCC Pre-Commit CI** | [github.com/riseproject-dev/gcc-precommit-ci](https://github.com/riseproject-dev/gcc-precommit-ci) | Patchwork pre-merge trybot for `gcc-patches` (`RISC-V`) with mirrored toolchain sources |
| **RISC-V GNU Toolchain CI** | [github.com/riseproject-dev/riscv-gnu-toolchain-ci](https://github.com/riseproject-dev/riscv-gnu-toolchain-ci) | Upstream `riscv-gnu-toolchain` Patchwork polling and PR validation |
| **Compiler Fuzzers (GCC & LLVM)** | [github.com/patrick-rivos/compiler-fuzz-ci](https://github.com/patrick-rivos/compiler-fuzz-ci) | Continuous Csmith & YARPGen random program fuzzing (migrating to `riseproject-dev`) |
| **Linux Kernel RISC-V CI** | [github.com/linux-riscv](https://github.com/linux-riscv) | Pre-commit ([Patchwork `linux-riscv`](https://patchwork.kernel.org/project/linux-riscv/list/)) and post-commit kernel build + QEMU boot CI |
| **Python RISC-V Wheels** | [github.com/riseproject-dev/python-wheels](https://github.com/riseproject-dev/python-wheels) | Pre-built, tested `riscv64` Python binary wheels (`NumPy`, `SciPy`, `PyTorch`, etc.) & [Python Versions](https://github.com/riseproject-dev/python-versions) |
| **Yocto `meta-riscv` Hardware CI** | [github.com/riscv/meta-riscv](https://github.com/riscv/meta-riscv) | Upstream Yocto layer built on `rise-python-ci-1` and boot-tested on OSU-OSL hardware |
| **Software Ecosystem Readiness** | [github.com/riseproject-dev/sw-ecosystem](https://github.com/riseproject-dev/sw-ecosystem) | Ecosystem readiness reports (`V8`, `Valgrind`, `tinygrad`, `SwiftShader`, `tcmalloc`, `tokenizers`, `Traefik`) |
| **RISE Grafana Cloud** | [riscv.grafana.net](https://riscv.grafana.net/) | Build Farm and runner utilization telemetry dashboards |

---

## 2. Hetzner Build Farm — 19× `CCX63` Dedicated x86 Server Inventory

The RISE Build Farm operates **19 dedicated `CCX63` instances** on Hetzner Cloud in the **Helsinki (`hel1-dc2`)** datacenter:
* **Per-Server Specification:** **48 Dedicated vCPUs** (AMD EPYC), **192 GB ECC RAM**, **960 GB NVMe SSD**, Ubuntu LTS (`x86_64`).
* **Fleet Capacity:** **912 Dedicated vCPUs**, **3,648 GB (3.65 TB) RAM**, **18.24 TB NVMe SSD**.
* **Grandfathered Contract Policy (Confirmed `2026-11-05`):** All 19 `CCX63` instances are billed under a grandfathered legacy contract rate that is less than half of current retail pricing for newly created or resized `CCX63` instances. **Operational Rule:** Never delete and recreate an idle `CCX63` server; always reprovision and repurpose existing instances in-place *(details in a separate Google Doc)*.

### 2.1 Server Allocation & Audit Status (as of November 2026)

*(Hosts are listed by short name; the full DNS name of each host is `<short-name>.di.riseproject.dev`. Public/private IPv4 and IPv6 addresses and Hetzner server IDs are recorded in `RISE Build Farm Guide for Administrators (Hetzner Instances)`.)*

| Server Group (Count) | Hosts (Short Logical Names) | Primary Workload & WG | Project Leads / Maintainers | Oct/Nov 2026 Audit Status & Active Migration Actions |
| :--- | :--- | :--- | :--- | :--- |
| **GCC Post-Commit CI (`7`)** | `rise-gcc-ci-05` .. `rise-gcc-ci-11` | **GCC Post-Commit CI**<br>([`gcc-postcommit-ci`](https://github.com/riseproject-dev/gcc-postcommit-ci))<br>*Toolchains WG* | Jiawei (ISCAS),<br>Kito Cheng (SiFive),<br>Jeff Law (Qualcomm),<br>Edwin Lu & Patrick O'Neill | • **14 runners** across 7 hosts.<br>• `05`, `06`, `08`–`11` actively run daily 4–8h multilib GCC post-commit builds; `07` was idle after a reboot (runners were not under `systemd`).<br>• **In Progress:** Re-registering runners from `patrick-rivos/gcc-postcommit-ci` to standalone [`riseproject-dev/gcc-postcommit-ci`](https://github.com/riseproject-dev/gcc-postcommit-ci) under `systemd` (pilot started on `rise-gcc-ci-07` with Jiawei) and consolidating multiple runners per host. |
| **GCC Pre-Commit CI (`4`)** | `rise-gcc-ci-01` .. `rise-gcc-ci-04` | **GCC Pre-Commit CI**<br>([`gcc-precommit-ci`](https://github.com/riseproject-dev/gcc-precommit-ci))<br>*Toolchains WG* | Jiawei (ISCAS),<br>Kito Cheng (SiFive),<br>Edwin Lu & Patrick O'Neill | • **9 runners** across 4 hosts (`precommit-ping` poller on `01` + 8 build runners).<br>• Legacy runners on `ewlu/gcc-precommit-ci` stalled on a broken upstream `newlib` submodule URL that has already been fixed in [`riseproject-dev/gcc-precommit-ci`](https://github.com/riseproject-dev/gcc-precommit-ci) via mirrored toolchain sources and checkout retries.<br>• **In Progress:** Re-registering `01`–`04` to `riseproject-dev/gcc-precommit-ci` and [`riseproject-dev/riscv-gnu-toolchain-ci`](https://github.com/riseproject-dev/riscv-gnu-toolchain-ci) with container isolation. |
| **Compiler Fuzzers (`2`)** | `rise-compiler-fuzzer-ci-1` .. `rise-compiler-fuzzer-ci-2` | **GCC & LLVM Fuzzers**<br>([`compiler-fuzz-ci`](https://github.com/patrick-rivos/compiler-fuzz-ci))<br>*Toolchains WG* | Min Hsu (SiFive),<br>Kito Cheng (SiFive),<br>Edwin Lu & Patrick O'Neill | • Discovered **99+ GCC bugs** and **56+ LLVM bugs** using Csmith & YARPGen.<br>• Both Hetzner hosts were found idle in the Oct 27 audit (`ci-1` stopped receiving jobs on `2026-07-13` when GitHub auto-disabled scheduled workflows in `patrick-rivos/compiler-fuzz-ci` after 60 days of repo inactivity; `ci-2` stopped after a `2026-03-01` reboot without `systemd`). Non-latency-sensitive fuzzing was also expanded onto preemptible GCP Spot instances.<br>• **Next Action:** Coordinate with Toolchains WG (Min Hsu / Kito Cheng) to re-point `ci-1` and `ci-2` to `riseproject-dev` under `systemd` or repurpose for LLVM pre/post-commit CI. |
| **Linux Kernel CI (`5`)** | `rise-linux-kernel-ci-1` .. `rise-linux-kernel-ci-5` | **Linux Kernel Pre & Post-Commit CI**<br>([`linux-riscv`](https://github.com/linux-riscv))<br>*Kernel & Virtualization WG* | Björn Töpel (Rivos) | • **Active (5 hosts):** Scrapes [`patchwork.kernel.org/project/linux-riscv`](https://patchwork.kernel.org/project/linux-riscv/list/) (~20–30 patches/day, 10 check scripts per patch) and runs post-commit builds across **120 kernel configurations** and **743 QEMU boot tests** (`riscv-for-next`, `riscv-fixes`, `linus-master`, `bpf`, `bpf-next`). |
| **Yocto `meta-riscv` x86 Builder (`1`)** *(formerly Python CI)* | `rise-python-ci-1` | **Yocto `meta-riscv` Dedicated x86 Builder**<br>([`riscv/meta-riscv`](https://github.com/riscv/meta-riscv))<br>*Distro Integration WG* | Valentina Fernandez &<br>Trevor Gamblin (BayLibre),<br>Puneetha Ramachandra,<br>Hai Bi | • **Repurposed (`2026-11-05`):** Originally provisioned for the Python Wheel Builder (which now runs via [`riseproject-dev/python-wheels`](https://github.com/riseproject-dev/python-wheels) and `riscv-runner`).<br>• Reprovisioned in-place as a dedicated x86 Yocto CI builder with persistent `sstate-cache` and `downloads` directories (cutting `meta-riscv` build times from ~2–7 hours to minutes), paired with physical hardware boot testing on the OSU-OSL Developer Cloud (`pool=a210-yocto`). |

---

## 3. Native RISC-V Runner Fleet (`riseproject-dev/riscv-runner`) & GitHub Larger Runners

In addition to the Hetzner `x86_64` cross-compilation and QEMU test fleet, RISE operates a multi-provider Kubernetes runner infrastructure ([`riseproject-dev/riscv-runner`](https://github.com/riseproject-dev/riscv-runner), led by Ludovic Henry) for native `riscv64` compilation and testing.

### 3.1 Native RISC-V Kubernetes Fleet (`riscv-runner`)
* **Control Plane & Provisioning:** Managed declaratively via modular Ansible playbooks (`ansible-runner-setup`) with SHA256 binary verification, Ubuntu 24.04/26.04 RVA23 container images, cross-entity fairness scheduling, and Prometheus `node_exporter` enrollment.
* **Hardware Worker Pools & Providers (`runner/device-plugin`):**
  * **Scaleway (`scaleway-em-rv1`, EMEA / Paris):** Alibaba T-Head `TH1520` (4-core XuanTie C910, 16 GB RAM, 128 GB SSD; mainline Linux 7.2 support upstreamed via RISE's engagement with Root Commit / Michael Opdenacker).
  * **ISCAS, Cloud-V / 10xEngineers, Mengzhuo & openKylin Pools:** Enrolls `spacemit-k1`, `spacemit-k3`, `spacemit-v100`, and openKylin `ultrarisc-dp-1000` nodes with automatic SoC signature detection (`mvendorid` / `marchid` / `mimpid`).
  * **OSU-OSL RISC-V Developer Cloud Integration (`Zhihe A210` & `SpacemiT K3`):** Physical boards at OSU-OSL (ratified RVV 1.0 on `zhihe-a210`; RVA23 + 1024-bit RVV 1.0 on `spacemit-k3`) are supported in the `riscv-runner` device plugin (`riseproject.dev/board=zhihe-a210` and `spacemit-k3`), with a tiered storage policy adding 128 GB SSDs to the subset of A210 boards dedicated to container-heavy `riscv-runner` pods.
* **Supported Upstream & RISE Organizations:** Serves external upstream organizations (`pytorch`, `ggml-org` / `llama.cpp`, `conda-forge`, `numpy`, `RuyiAI-Stack`) as well as `riseproject-dev` repositories ([`python-wheels`](https://github.com/riseproject-dev/python-wheels), [`python-versions`](https://github.com/riseproject-dev/python-versions), [`pytorch-ci`](https://github.com/riseproject-dev/pytorch-ci), `executorch`, `OpenBLAS`).

### 3.2 Transition to Native GitHub-Hosted Larger Runners for x86 Workflows (`2026-11-12`)
* On **`2026-11-12`**, the RISE Governing Board approved reallocating funds from discontinued custom-managed x86 cloud runners to native **GitHub-Hosted Larger Runners** (`32 vCPU / 128 GB RAM` class).
* **Rationale:** For bursty x86 compilation workloads averaging `<40%` duty cycle, GitHub-Hosted Larger Runners are cost-equivalent to dedicated cloud VMs while requiring zero OS/runner maintenance ("all batteries included"). The reallocation is budget-neutral *(details in a separate Google Doc)*.
* **Scaleway x86 Builder for Android:** A Scaleway x86 runner, `rise-aosp-builder-0` (`32 vCPU / 128 GB RAM`), runs the nightly Android 17, Kleaf kernel, and Mesa builds for the `riseproject-dev/android_*` repositories.

---

## 4. Supporting Infrastructure Services & Secret Management

| Service | Provider / Platform | Current Configuration & Operational Notes |
| :--- | :--- | :--- |
| **Primary Domain & DNS** | **AWS Route53** (`riseproject.dev`) + **Rackspace Cloud DNS** (`di.riseproject.dev`) | The `di.riseproject.dev` subdomain is delegated from AWS Route53 to Rackspace Cloud DNS and holds the Build Farm host records *(zone administration notes are in `RISE Build Farm Guide for Administrators (Hetzner Instances)`)*. |
| **Artifact & Object Storage** | **Hetzner Object Storage** (`S3`-compatible) | Bucket `riscvsoftware`: available for long-term archival of CI artifacts; not currently in use. |
| **Telemetry & Monitoring** | **Grafana Cloud** ([`https://riscv.grafana.net/`](https://riscv.grafana.net/)) | Collects Prometheus `node_exporter` metrics across Build Farm and runner hosts. Organization admins include Puneetha Ramachandra and Hai Bi. |
| **Outbound Email / Alerts** | **Mailgun** | Sends automated CI failure notifications and Patchwork alerts. |
| **Secret & Token Management** | **Transitioning to Shared Secret Manager (`2026-11-06`)** | Moving RISC-V CI and GitHub Actions runner private keys/tokens out of single-admin personal vault storage into a shared multi-admin secret store (**Scaleway Secret Manager** under evaluation first, alongside OIDC Workload Identity Federation options). No secrets or credentials are stored in GitHub repositories. |

---

## 5. Simulator / Emulator (QEMU) & Developer Tools Workstreams

Following the 2026 consolidation of the **Developer Infrastructure WG** and **Simulator / Emulator WG** into the **RISE Developer Tooling WG** (co-chaired by Daniel Barboza, Puneetha Ramachandra, and Hai Bi), the working group also tracks upstream QEMU feature enablement and cross-distro developer images:

### 5.1 Upstream QEMU Enablement Status (`2026-11`)
* **Completed (`QEMU 11.2`):**
  * **`SE_01_022` — RISC-V Server SoC Reference Board:** Merged upstream and available in **QEMU 11.2**.
  * **QEMU `qemuriscv64` TCG TLB-Flush Optimization (`30–40%` Speedup):** Upstream patch by Trevor Gamblin and Daniel Barboza (`2026-09`) optimizing RISC-V TLB flushing in TCG emulation mode, reducing Yocto `qemuriscv64` `ptest` runtimes by **30–40%** (`coreutils` `ptest` improved by **38%**).
* **Under Review on `qemu-devel` Mailing List:**
  * **`SE_01_003` — WorldGuard Support:** Split into smaller patch series; upstream reviews ongoing.
  * **`SE_01_004` — IOPMP Support:** Patch series v11 under review.
  * **`SE_01_005` — PCIe Passthrough on x86 Hosts:** Patch series v1.
  * **`SE_01_018` — ACPI PPTT Support for RISC-V:** Patch series v1 under review.
  * **`SE_01_029` — P Extension Support:** Patch series v1 under review.

### 5.2 Cross-Distro Stages & Bootable Developer Images
* Building on Nathan Egge and Luca Barbato's Gentoo RISC-V stage images ([`docs/developer-tools.md`](../developer-tools.md)), the **Cross-Distro Stages** initiative was presented to the RISE Governing Board on **`2026-11-12`** (no objections; TSC review next) to produce automated, continuously tested bootable RISC-V images across **Ubuntu, Red Hat, Debian, and Gentoo** paired with board-specific U-Boot and OpenSBI for the RISC-V Developer Cloud and GitHub Actions runners *(details in a separate Google Doc)*.
