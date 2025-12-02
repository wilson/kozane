# Kozane (小札) ("koh-zah-neh")

**The Way of Immutable Linux.**

> *Small, layered scales of samurai armor.*

In the physical realm, **Kozane** are the individual scales of iron and leather, lacquered against the elements and laced together to form the *O-yoroi* (Great Armor). They provide flexibility through multiplicity and strength through isolation.

In the digital realm, **Kozane** is an operating system architecture where individual, immutable components (kernel, configuration, userland, and applications) are layered to form a resilient, verifiable whole.

A synthesis of ancient aesthetic philosophy and bleeding-edge systems engineering.  
We reject the messy, mutable reality of the "distribution" in favor of the disciplined, composed reality of the "image."  

Inspired by [ParticleOS](https://github.com/systemd/particleos).  
Always take everything to its logical extreme.  

All "user intent" is captured in [CUE](https://cuelang.org/) format.  
CUE is full of deep ideas, it may take some time to absorb.  

Kozane is not trying to be anyone's first Linux-based OS, only their last.

## The Materials (Dependencies)

Kozane is not "installed"; it is forged. We rely on a strict chain-of-trust and powerful tools to build the system image.
*   **[mkosi](https://github.com/systemd/mkosi)** (Make OS Image) — The forge. Generates the immutable system images with hermetic reproducibility.
*   **[systemd](https://systemd.io/)** — The lacing (*odoshi*). Provides the `sysupdate`, `repart`, and `creds` mechanisms that bind the system together and secure it at rest.
*   **[CUE](https://cuelang.org/)** — The blueprint. A unified, rigorously validated configuration language that generates the system state before a single byte is written to disk.
*   **[OSTree](https://ostreedev.github.io/ostree/)** — The memory. Provides content-addressed versioning for the system binaries, ensuring atomic transitions and distinct history.
*   **[Podman](https://podman.io/)** & **[Incus](https://linuxcontainers.org/incus/)** — The capacity. Strictly confined containers for all applications and "pet" workspaces, keeping the host pure.
*   **[Bazzite Linux]([https://bazzite.gg/](https://bazzite.gg/))** — The proprietary steel additives. We inherit high-performance kernel patches (fsync/futex2) to ensure the armor does not hinder the warrior's movement.

## Architecture

1.  **The Core:** A read-only, verified boot image containing only the kernel, systemd, and hardware drivers.
2.  **The Configuration:** Generated entirely from CUE schemas at build time; `/etc` is transient or strictly managed.
3.  **The Userland:** Applications live in containers. CLI tools live in `~/.local` (via [mise](https://mise.jdx.dev/)) or disposable environments.
4.  **The Lifecycle:** Updates are atomic image swaps. There is no package manager on the host.

> *"It is bad when one thing becomes two. One should not look for anything else in the Way of the Samurai."*
