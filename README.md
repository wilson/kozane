# Kozane (小札) · "koh-zah-neh"

**The Way of Immutable Linux.**

> *Small, layered scales of samurai armor.*

Kozane are the individual iron-and-leather scales that were lacquered and laced together to form the flexible, near-impenetrable ō-yoroi.  
In the same spirit, Kozane is a desktop Linux architecture built from small, immutable, verifiable layers that compose into a resilient whole.

This is not a traditional distribution.  
It is a forge.

Kozane is not trying to be anyone’s first Linux-based OS, only their last.

Only a kernel security fix should ever require a hard reboot.

Inspired by [ParticleOS](https://github.com/systemd/particleos).  
All user intent lives in a single, rigorously validated [CUE](https://cuelang.org/) file.  
Updates are full-image atomic replacements. The host has no package manager.

## Materials
- [mkosi](https://github.com/systemd/mkosi) — the forge  
- [systemd](https://systemd.io/) + sysupdate/sysext — the lacing  
- [CUE](https://cuelang.org/) — the blueprint  
- [OSTree](https://ostreedev.github.io/ostree/) — the memory  
- [Bazzite](https://bazzite.gg/) kernel — fsync/futex2 steel  
- [ChimeraUtils](https://github.com/chimera-linux/chimerautils) overlay — BSD userland without breaking GNU scripts  
- [Podman](https://podman.io) + [Incus](https://linuxcontainers.org/incus/) — containers for everything else

Clone -> `$EDITOR system.cue` -> forge -> defend.

The armor is yours.  

> "It is bad when one thing becomes two. One should not look for anything else in the Way of the Samurai."  
> — Hagakure
