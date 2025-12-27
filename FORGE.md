# FORGE.md — Maintainer Notes

This file is private to the forgemaster. Everything here is allowed to be messy, aspirational, or half-baked.  

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
