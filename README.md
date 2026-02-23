# Arch Linux Systems Design

An opinionated, architecture-focused approach to building a resilient Arch Linux system using Btrfs, Snapper, and window-manager-based graphical stacks.

This project contains a single, well-reasoned system design intended for long-term personal use. It emphasizes deterministic recovery, layered snapshot management, and minimal graphical environments over desktop abstractions.

This is not a beginner’s introduction to Arch.
It is a structured system architecture for users who want to understand and control their system’s behavior.

---

## Project Goals

- Design a resilient Arch system centered on Btrfs snapshotting
- Implement a layered recovery model (internal rollback + external replication)
- Favor deterministic recovery over convenience
- Use window-manager-based graphical stacks (i3 or sway) instead of full desktop environments
- Maintain long-term relevance independent of hardware trends

---

## Hardware Context

The design documented here was developed and tested on legacy BIOS-based laptop hardware (e.g., ThinkPad T400-era systems).

---

## Core Design Principles

- **Single coherent path** - This project does not present multiple competing installation strategies. It documents one architecture
- **Snapshot-driven system state** - System integrity is managed through structured snapshot creation and retention
- **Layered recovery architecture** - Internal rollback and external backup are treated as distinct layers
- **Minimal and transparent system design** – Only essential components are included. System behavior is visible and understandable rather than abstracted behind desktop layers
- **Intentional operation** – Updating and maintaining the system follows a structured operational model designed to preserve system integrity

---

## Repository Structure

- [01 – Philosophy](docs/01-philosophy.md)
- [02 – Base Installation](docs/02-base-installation.md)
- [03 – System Recovery Model](docs/03-system-recovery-model.md)
- [04 – Graphical Stack: Xorg + i3](docs/04-graphical-stack-xorg-i3.md)
- [05 – Graphical Stack: Wayland + sway](docs/05-graphical-stack-wayland-sway.md)
- [06 – Operational Discipline](docs/06-operational-discipline.md)
- [07 – Extensions and Enhancements](docs/07-extensions-and-enhancements.md)
- [08 – Troubleshooting](docs/08-troubleshooting.md)

Each document represents a distinct layer of the system architecture.
The core of the design is the **System Recovery Model**.

---

## Intended Audience

This project is intended for intermediate Linux users who:

- Have experience using Arch or another Linux distribution
- Want to move away from full desktop environments
- Want to understand and control their system’s recovery behavior
- Value a structured, opinionated architecture over broad option lists

---

## Status

Active development.

The repository structure is scaffolded to reflect the intended system architecture.
Sections are incrementally refined and merged into `main` upon reaching stable milestones.

---

## Scope

This repository does not intend to replace the [Arch Wiki](https://wiki.archlinux.org/title/Main_page).
It documents a specific system design built on top of Arch.

Readers are encouraged to consult the [Arch Wiki](https://wiki.archlinux.org/title/Main_page) for detailed reference documentation.
