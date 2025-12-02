# FORGE.md — Maintainer Notes

This file is private to the forgemaster. Everything here is allowed to be messy, aspirational, or half-baked.  

## Current Status

- CUE -> mkosi.conf generation: schema drafted
- CUE -> incus preseed generation: schema drafted
- ChimeraUtils sysext build: outlined (priority 70 for now)
- Bazzite kernel + akmod-nvidia compiled at image build time: TBD
- dm-verity root + A/B systemd-sysupdate: TBD
- First-boot user creation + SSH keys: TBD

## Remaining Work

[ ] Finish proper TPM2-sealed credential injection via mkosi extra trees
[ ] Generate Podman quadlets from CUE (right now only incus preseed exists)
[ ] Add example quadlets (tailscale, syncthing, jellyfin, audiobookshelf, etc.)
[ ] CI pipeline that builds the image on push and publishes to GitHub Releases
[ ] Automatic semantic version bump when schema or kernel changes
[ ] Signed system extensions once systemd 257 lands
[ ] Write "how to encrypt a secret for a specific machine" guide
[ ] Optional ostree-native-container output mode for the rpm-ostree crowd

## Open Decisions

- Bake `mise` into the image or keep it as user-installed?
- Chimera overlay priority: stay at 70 or drop to 50? (needs more real-world script testing)
- Rope in some other AMD GPUs for testing?

## v1.0 Testing Checklist

[ ] NVIDIA RTX 4070 Ti Super + Secure Boot + driver rebuild works flawlessly
[ ] Intel iGPU laptop (any recent generation) works
[ ] AMDGPU desktop works
[ ] Automatic rollback after three failed boots works
[ ] Incus profile or storage changes require only a rebuild, no manual steps
[ ] ChimeraUtils overlay never breaks a script that hard-codes /usr/bin/*
