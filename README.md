# Lustre Engineering

Open engineering journey toward Lustre filesystem development, HPC infrastructure, Linux kernel, and storage systems.

## Purpose

This repository is a practical engineering lab and learning record. It connects Linux systems administration with storage internals, Lustre, LNet, HPC scheduling, performance engineering, and eventually upstream open-source contribution.

The guiding workflow is:

```text
Concept
  ↓
Read real code
  ↓
Build / run
  ↓
Break it
  ↓
Debug it
  ↓
Fix it
  ↓
Test it
  ↓
Document it
  ↓
Automate it
```

## Current focus

- Rocky Linux and Linux fundamentals
- C, system calls, file I/O, processes, threads, and memory
- Linux kernel and VFS internals
- Lustre and LNet architecture
- HPC cluster administration and Slurm
- Storage and performance testing
- Reproducible lab automation with Ansible
- Open-source engineering practices and upstream contribution

## Lab philosophy

Manual first. Automation second.

Every important change should have evidence:

```text
Before → Change → Build → Test → After
```

## Status

This repository is being built incrementally as part of a hands-on engineering mentorship. Early content will favor small, reproducible labs over large collections of notes.
## Development Environment

This repository is developed and tested from a Rocky Linux engineering lab.
