---
title: "DuoOS"
description: "A minimal bare-metal RISC-V OS built from scratch."
summary: "A minimal bare-metal RISC-V OS built from scratch for the Milk-V Duo."
showDate: false
showReadingTime: false
showAuthor: false
---

[DuoOS on GitHub](https://github.com/LeoriumDev/duo-os)

A minimal bare-metal RISC-V operating system built from scratch, targeting the [Milk-V Duo](https://milkv.io/duo).

## Overview

DuoOS is a personal systems project focused on learning operating systems from first principles by building one directly on bare metal. The goal is to understand the full path from bootstrapping and low-level hardware interaction to core kernel mechanisms such as memory management, traps, scheduling, and device support.

## Goals

- Build a small but real bare-metal OS from scratch
- Learn RISC-V systems programming deeply
- Understand kernel internals through implementation
- Document the process clearly through a long-form blog series

## Target Platform

- Architecture: 64-bit RISC-V
- Board: Milk-V Duo

## Current Status

DuoOS is currently in active development.

## Implemented

- Early project setup
- Initial bootstrapping
- Basic bring-up work

## Planned

- UART output
- Trap and interrupt handling
- Memory management
- Task scheduling
- Device support
- Filesystem exploration

## Writing

Related posts and development notes are collected in the [DuoOS series](/blog/duo-os/).

## Why this project exists

I built DuoOS to learn operating systems the hard way: by writing one. Instead of treating OS concepts as purely theoretical material, this project turns them into something concrete, inspectable, and incremental.

## Links

- Github Repository: [LeoriumDev/duo-os](https://github.com/LeoriumDev/duo-os)
- Blog Series: [/blog/duo-os/](/blog/duo-os/)