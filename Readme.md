# README

## Overview

This repository contains my lab records for the **"Computer Systems: A Programmer's Perspective" (CSAPP)** course. The labs cover various aspects of computer systems, including debugging, reverse engineering, computer architecture, and cache optimization.

---

## Table of Contents

1. [GDB](#gdb)
2. [BombLab](#bomblab)
3. [AttackLab](#attacklab)
4. [ArchLab](#archlab)
5. [CacheLab](#cachelab)

---

## GDB

### Description

This section provides an overview of **GDB**, the GNU Project debugger, and its usage for debugging programs. It includes essential commands and practical examples for using GDB effectively.

### Files

- [GDB.md](GDB/GDB.md): Documentation and steps for using GDB.

### Key Points

- Installing and setting up GDB.
- Basic and advanced GDB commands.
- Debugging programs and analyzing core dumps.

---

## BombLab

### Description

**BombLab** is a reverse engineering and debugging exercise where the goal is to defuse a binary bomb by analyzing its assembly code and providing correct inputs for each phase.

### Files

- [Bomblab.md](Bomblab/Bomblab.md): Detailed documentation and steps for completing the BombLab.

### Key Points

- Analyzing assembly code to understand program logic.
- Using GDB to debug and step through the bomb's phases.
- Providing correct inputs to defuse the bomb.

---

## AttackLab

### Description

**AttackLab** teaches buffer overflow and return-oriented programming (ROP) attacks. The lab involves exploiting vulnerabilities in provided target programs to execute arbitrary code.

### Files

- [Attack Lab.md](Attacklab/Attack%20Lab.md): Detailed documentation and steps for completing the AttackLab.

### Key Points

- Understanding buffer overflow vulnerabilities.
- Constructing attack strings using provided utilities.
- Performing ROP attacks using gadgets.

---

## ArchLab

### Description

**ArchLab** focuses on understanding the Y86-64 instruction set architecture and implementing various functions in Y86-64 assembly language. It involves writing and simulating Y86-64 programs and optimizing assembly code.

### Files

- [Archlab.md](ArchLab/Archlab.md): Documentation and steps for completing the ArchLab.

### Key Points

- Writing Y86-64 assembly programs.
- Extending the SEQ processor to support new instructions.
- Optimizing assembly code for performance.

---

## CacheLab

### Description

**CacheLab** focuses on understanding the impact of cache memory on program performance. The lab involves implementing a cache simulator and optimizing a matrix transpose function to minimize cache misses.

### Files

- [CacheLab.md](Cachelab/CacheLab.md): Documentation and steps for completing the CacheLab.

### Key Points

- Simulating cache behavior and analyzing cache performance.
- Implementing the Least Recently Used (LRU) replacement policy.
- Optimizing matrix transpose to reduce cache misses.

---

## Acknowledgments

- The CSAPP course and its authors for providing the lab materials.
- The open-source community for tools and resources used in these labs.
