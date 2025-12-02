# Kozane Handbook

This document explains how to forge and maintain your own Kozane system.

## 1. Repository Layout

```
kozane/
├── schema.cue          # Rigid type definitions (do not edit directly)
├── system.cue          # <- Your configuration lives here
├── exports/            # Generated files (.gitignore these)
│   ├── mkosi.conf
│   ├── incus-preseed.yaml
│   └── userland.sh
└── scripts/
├── build.sh        # Forge the image
└── update.sh       # Apply atomic updates
```

## 2. Your Configuration (system.cue)

Define your system in [CUE](https://cuelang.org/) format.
All valid fields and constraints are enforced by `schema.cue`.

Example minimal laptop configuration:

```cue
package kozane

#System: {
    hostname: "shaguma"
    kernel:   "bazzite-fsync"
    gpu:      "intel" | "amd" | "nvidia"

    users: {
        "gonnojo": {
            uid:    1000
            groups: ["wheel", "incus-admin", "video", "input"]
            ssh_keys: [
                "ssh-ed25519 AAAAC3... user@host",
            ]
        }
    }

    overlays: chimera: enabled: true
}
```

Example high-end nVidia gaming/creative rig:

```cue
gpu: "nvidia"
kernel: "bazzite-fsync"
overlays: chimera: enabled: true
```

## 3. Forging the Image

Build with [mkosi](https://github.com/systemd/mkosi):
```sh
./scripts/build.sh
```

This command:
1. Validates `system.cue` against the schema
2. Generates `exports/mkosi.conf`, `incus-preseed.yaml`, etc.
3. Builds a raw disk image (`kozane_<hostname>_<version>.raw`)
   - Injects [Bazzite](https://bazzite.gg/) kernel + matching nVidia `akmod` drivers (compiled during build)
   - Builds ChimeraUtils system extension overlay (BSD userland in /usr/local)
   - Pre-bakes all configuration
   - Signs with `dm-verity`

## 4. First Boot

1. Flash the raw image to USB or disk (dd, gnome-disks, or [balenaEtcher](https://etcher.balena.io/))
2. Boot (UEFI only, Secure Boot compatible)
3. On first boot:
   - `systemd-sysext` merges the ChimeraUtils overlay
   - [Incus](https://linuxcontainers.org/incus/) initializes from the preseed
   - Your user is created with specified SSH keys
   - Root partition mounts read-only

You land in a shell with BSD tools in `/usr/local/bin` and GNU tools still in `/usr/bin`.

## 5. Daily Use

- Permanent CLI tools -> [mise](https://mise.jdx.dev/) into `~/.local`
- Throw-away environments -> `incus launch images:debian/13 dev && incus exec dev -- su - $USER`
- Services (Tailscale, Syncthing, etc.) -> define in `system.cue` -> become systemd-managed *Podman quadlets*
- Updates -> edit `system.cue` -> `./scripts/build.sh` anywhere → upload new image → reboot

## 6. Atomic Updates

Kozane uses `systemd-sysupdate` + `OSTree`.

After you’ve uploaded the new image:
```sh
sudo -i
systemd-sysupdate update
reboot
```

Three failed boots -> `systemd-boot` automatically rolls back to the previous slot.

## 7. Why GNU and BSD Tools Coexist Peacefully

- GNU coreutils -> priority **100** -> `/usr/bin`
- ChimeraUtils (BSD) -> priority **70** -> `/usr/local/bin`
- `~/.local/bin` -> priority **10**

`$PATH` is ordered, so interactive use prefers BSD tools, but any script that hard-codes `/usr/bin/ls` keeps working.

## 8. Secrets (TPM2-Sealed)

Encrypt secrets with `systemd-creds` on your workstation and reference them in `system.cue`. They are only decrypted at service start, and only on the exact machine whose trusted platform module you targeted.

## Done.

You now wield an immutable, verifiable, high-performance Linux workstation that can be regenerated from a single ~200-line CUE file at any moment.

Edit -> forge -> reboot -> repeat.
