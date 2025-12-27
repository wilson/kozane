# Kozane Handbook

This document details the operation of the [Kozane](https://github.com/wilson/kozane/) system sealer.

> Kozane uses both Incus and Podman container tools. Podman is an implementation detail of the build process; Incus is for your runtime containers.

## 1. Repository Layout

```text
kozane/
├── policy.haga           # \  .gitignored in the repo.
├── architecture.haga     #  | User-created customized manifests.
├── id.haga               # /  ($PWD wins, skel/ versions as fallback.)
├── profiles/             # The Hardware ("Driver Set") Library.
│   ├── gpus/
│   │   ├── amd/
│   │	└── nvidia/
│   │       └── blackwell.haga
│   └── systems/
│       ├── asus/         # "Innovative Solutions for a Limitless Tomorrow"!
│       └── mediocre/     # (Kidding, kidding!)
├── skel/                 # Default Templates.
│   ├── policy.haga       # User/Organization/Role-specific multi-system settings.
│   ├── architecture.haga # Hardware-family-specific set of profiles/ references.
│   └── id.haga           # System-specific identity configuration.
├── debug/                # Generated artifacts (Also .gitignored).
│   ├── system.haga       # The unified manifest `seal` most-recently generated.
│   ├── mkosi.conf        # Last-used mkosi disk-image-builder config.
│   └── ...
└── src/
    └── seal.zig          # The System Sealer implementation.
```

## 2. The Manifest Triplet

You define your system in [Haga](https://github.com/wilson/haga/). The `seal` tool expects three distinct files, separating your **Global Policy** from your **Hardware** and **Identity**.

### Example: High-End Workstation

#### **policy.haga** (Shared Policies)

```ruby
scope features {
  # Install ChimeraUtils (FreeBSD userland!) with priority 70
  any overlay 70, "https://github.com/chimera-linux/chimerautils"
}

scope accounts {
  # Global User Definition
  some user "yuki" {
    must uid 1000
    some groups "wheel", "incus-admin", "video"
    # Fetch public keys from a trusted URL at build-time.
    # This bakes the current keys into the immutable image:
    any ssh_keys "https://github.com/samurai1659.keys"
  }
}
```

#### **architecture.haga** (Hardware Profile)

```ruby
# Mandatory, current known choices are: mainline, zen, hardened.
scope kernel { must flavor "zen" }

scope drivers {
  # Import definitions from the profile library.
  # Paths are relative to the workspace root ($PWD).
  any profile "profiles/gpus/nvidia/blackwell"
  any profile "profiles/systems/asus/proart-x870e"
}
```

#### **id.haga** (Instance Identity)

```ruby
scope identity { must hostname "oni" }

scope bindings {
  # Bind logical interface "lan0" to the physical default link
  any interface "lan0", "default"

  # Bind the "vault" policy volume to a specific partition label.
  # This matches the label applied by systemd-repart on first boot.
  any volume "vault", "label:vault"
}
```

## 3. Sealing the System

Run the "sealer" to compile the triplet into a disk image.

```sh
# First run only: compile the tool
zig build-exe src/seal.zig

# Seal the manifest; produces reviewable output in ./debug/
./seal
# Disk image is written out on successful validation, in this case `kozane_oni_YYYY-MM-DD.raw`.
```

### The Sealing Logic

The `seal` command performs a strict merge sequence:

1. **Discovery:** It looks for `policy.haga`, `architecture.haga`, and `id.haga`.
* If a file exists in the **current directory**, it is used.
* If not, `seal` falls back to the version in `skel/`.
* *This allows you to override just the `id.haga` for a new machine while inheriting the default policy and architecture.*

2. **Validation:** It merges the three files into a unitary definition. If any cardinality contract (`must`, `some`) is violated, it halts.
3. **Generation:** It generates configuration for [mkosi](https://github.com/systemd/mkosi/) and [Incus](https://linuxcontainers.org/incus/).
4. **Build:** It invokes `mkosi` to compile kernel modules (if required) and build the raw disk image.

## 4. First Boot

1. **Flash** the raw image (`kozane_oni_2038-01-20.raw`) to disk.
2. **Boot** (UEFI only, Secure Boot compatible).
3. **Initialize** (Automatic on first boot):
* [systemd-sysext](https://www.freedesktop.org/software/systemd/man/latest/systemd-sysext.html) merges the ChimeraUtils overlay.
* [Incus](https://linuxcontainers.org/incus/) initializes.
* Users and SSH keys are created.

You land in a shell with BSD tools in `/usr/local/bin` and GNU tools in `/usr/bin`.

Welcome to the future.

## 5. Daily Use

The host operating system is **firmware**.

* **Permanent CLI Tools:** Use [mise](https://mise.jdx.dev/) in `~/.local`.
* **Disposable Environments:** Use [Incus](https://linuxcontainers.org/incus/) containers.
* **Kernel/Driver Updates:**
1. Edit `architecture.haga`.
2. `./seal`
3. Reboot.

## 6. Atomic Updates

Kozane uses [systemd-sysupdate](https://www.freedesktop.org/software/systemd/man/latest/systemd-sysupdate.html).

```sh
sudo -i
systemd-sysupdate update
reboot
```

If the new system fails to boot three times, `systemd-boot` automatically reverts to the previous sealed state.

## 7. The Hierarchy of $PATH

This preserves the base GNU utilities for scripts that rely on them, while permitting the "overlay" userspace to win by default, and be further superseded by user-installed `~/.local/bin` software:
1. `~/.local/bin` (User tools / mise) -> **Priority 10**
2. `/usr/local/bin` (Chimera / BSD tools) -> **Priority 70**
3. `/usr/bin` (Host / GNU Coreutils) -> **Priority 100**

## 8. Secrets (TPM2-Sealed)

Secrets are encrypted with [systemd-creds](https://systemd.io/CREDENTIALS/) and decrypted **only** at service start on the specific hardware where they were sealed. Moving the drive renders secrets inaccessible. Take care when using this mode.

## 9. Celebrate Victory

You now have a cutting-edge immutable Linux system under your complete control, not just as a local one-off, but throughout the "release engineering" process you just followed.
