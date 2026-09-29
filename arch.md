# macfit Architecture

## Purpose

macfit keeps Mac config files in one encrypted store and restores them on any Mac. macOS only.

## System Summary

macfit registers live files, captures their content and mode as versions in one sealed store file, and restores them on any Mac that can open the store. It writes nothing but the store file, the remembered-path pointer, and a last-save record.

## Current Platform

- Go

## Major Components

- `cmd/macfit/main.go`: help, flags, store-path resolution, and the store commands
- `key.go`: `key show`, `key restore`, `key rm`, and `key passphrase`
- `set.go`: entry binding and mode changes
- `render.go`: `render` and `cat`
- `github.com/queone/gkit/lockbox`: the encrypted store, the key store, the manifest, and the save log

## Core Files

- `AGENTS.md`: base governance contract
- `plan.md`: prioritized roadmap and approved direction
- `build.sh`: self-contained build / release-prep / release script (Bash 3.2+, no external tools)
- `govna/development-cycle.md`: workflow from roadmap through release
- `govna/ac-template.md`: acceptance-criteria template for new work
- `govna/build-release.md`: build, test, and release rules

## Data And Control Flow

Each command resolves the store path (`-s`, then `MACFIT_STORE`, then the remembered path, then `$XDG_DATA_HOME/macfit/macfit.store`), reads the key from the login Keychain, and decrypts the store into memory. It runs, then seals the store again and replaces the file atomically.

## AC Lifecycle Control Flow

The governed change path is `Draft → Audit → Refine → Implement → Ratify → Package`. Draft creates the AC; Audit, Refine, Implement, and Ratify are the four AC phases; Package is post-Ratify release preparation and is not a fifth phase.

Integrated audit adoption is the only command-mediated phase exception. It can advance one emitted adoption AC through immediate Audit and no-edit Refine, but it cannot enter Implement. Every unpackaged AC with implementation in the unreleased state enters the pending release batch, including work awaiting Ratify. A private pre-Implement calculation prevents that complete batch from growing beyond one 80-byte prefix-plus-summary message. Package requires every member to be Ratified, rejects excluded implemented work, and rechecks the complete batch before prep. A named request such as `Package AC70+AC71` establishes a fitting multi-AC batch; a standalone Package alias reuses the complete batch already established in the active session.

## Architecture Notes

- record stable system decisions here
- prefer durable structure and interfaces over transient implementation detail
- The store is a SQLite database sealed with XChaCha20-Poly1305. Its file starts with lockbox's `MACFIT` format tag, which must not change.
- The data key lives in the login Keychain under service `macfit`. A passphrase-wrapped copy (Argon2id) sits in the store header, so another Mac unlocks it with the passphrase once.
- Targets are stored as `$XDG_*`, `$CLAUDE_CONFIG_DIR`, or `~` templates, so a pulled file lands wherever those variables point on that Mac.
- Only whole-file atomic saves reach the store, which keeps a synced folder safe.
- Every save adds a row to the store's save log and records it in `$XDG_STATE_HOME/macfit/last-save-KEYID`, so a Mac notices when another Mac's copy replaced its save.
- `pull` only plans unless given `-f`, and `pull`, `push`, and `diff` refuse symlinks.
- lockbox comes from `github.com/queone/gkit` at a pinned version. Raise that version deliberately.

## Conventions

- update this document when architecture or major workflow changes materially
- keep implementation detail in code and stable architecture here
