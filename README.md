# Kozane · "koh-zah-neh"
> *Small, layered scales of samurai armor.*

**Kozane** (小札) is a rigorous system-sealer for high-integrity Linux computing.

It treats the operating system as **firmware**. It does not "install" packages; it **seals** a declarative **manifest** into an immutable, cryptographically-verified disk image.

## The Philosophy

Inspired by [ParticleOS](https://github.com/systemd/particleos) and [Bazzite](https://bazzite.gg/), but taken to the extreme:
1. **Unified:** The kernel, userland, and configuration are a single atomic "build" unit.
2. **Total:** The **manifest triplet** is the strict boundary of the system. If it is not in the configuration, it does not exist.
3. **Finite:** The system state must be fully enumerated. There is no "undefined behavior," "drift," or "config rot."

Kozane is not trying to be anyone’s first Linux-based OS, only their last.

Only a kernel security fix (or proprietary GPU driver demand) should ever require a hard reboot.

## Architecture

Kozane is not a distribution; it is a **composition** of high-assurance components:

* **Kernel:** [Bazzite Kernel Project](https://github.com/bazzite-org/kernel-bazzite/).
* **Userland:** [ChimeraUtils](https://github.com/chimera-linux/chimerautils/) (FreeBSD userland overlay).
* **Mechanism:** [systemd-sysext](https://github.com/systemd/systemd/tree/main/src/sysext/) merging `/usr/local` at runtime.
* **Secrets:** Sealed to the hardware via TPM2 with [systemd-creds](https://github.com/systemd/systemd/tree/main/src/creds/).

## The Manifest Format

Kozane definitions are written in [Haga](https://github.com/wilson/haga/), a strict configuration language where every statement enforces its own cardinality.

Unlike standard config formats, Haga requires you to explicitly declare whether a resource is a singleton (`must`), an option (`may`), or a collection (`some`/`any`).

This permits a "mere" `grep must **/*.haga` command to enumerate all of your "exactly one" required properties.

The system definition is composed of a **triplet** of configuration files, which `seal` merges into a unitary system manifest at build time:

1.  **`policy.haga`**: The Shared Policy. Global software, security, and networking intent shared across all your machines.
2.  **`architecture.haga`**: The Hardware Profile. Driver definitions and kernel tuning for a specific class of hardware (e.g., "Zen 5 Workstations").
3.  **`id.haga`**: The Instance Identity. Unique settings for a single physical machine (Hostname, IP, Disk UUIDs).

## Commencing the Journey

### Build System Prerequisites

* [Zig](https://ziglang.org/): To compile the **Sealer**.
* [mkosi](https://github.com/systemd/mkosi/): To orchestrate the **hermetic compilation** of kernel modules and the final image.
* [podman](https://github.com/containers/podman/): To provide the **isolated sandbox** where `mkosi` builds the OS.

### Workflow

You do not run `apt` or `dnf`, or any package manager at all.

You edit the manifest and "reseal" the system:
```sh
KOZANE=git@github.com:wilson/kozane
git clone $KOZANE && cd kozane

# Compile the "Sealer" (first time only)
zig build-exe src/seal.zig

# 1. Customize your Identity (optional but recommended)
# The sealer begins by looking in the current directory for the three .haga files.
# If any of the three is not present, the default for it from the "baked-in" skel/ is used.
# The three filenames are conventional and not customizable on a `seal` invocation.
# You can use symlinks to "build" arbitrary combinations from a "library" of files if needed.
cp skel/id.haga .
$EDITOR ./id.haga

# 2. Verify and Seal
# In this example, this merges ./id.haga, skel/policy.haga, and skel/architecture.haga.
# (This merge is trivial, effectively just `cat`.)
# After merging, the process creates the resulting disk image and verifies it.
./seal
# 3. Done: See HANDBOOK.md for further details on the lifecycle of your new system.
```

> "It is bad when one thing becomes two. One should not look for anything else in the Way of the Samurai."
> -- Hagakure (*Hidden by Leaves*, 1709-1716 C.E.)
