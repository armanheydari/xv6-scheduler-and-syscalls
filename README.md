# xv6-scheduler-and-syscalls

> Operating-systems coursework in C: a priority scheduler and new system calls built into the xv6 kernel, plus Linux labs on processes, threads, pipes and sockets.

[![C](https://img.shields.io/badge/language-C-00599C.svg)](https://en.cppreference.com/w/c) [![xv6](https://img.shields.io/badge/kernel-xv6%20%28x86%29-lightgrey.svg)](https://github.com/mit-pdos/xv6-public) [![QEMU](https://img.shields.io/badge/runs%20on-QEMU-orange.svg)](https://www.qemu.org/)

## 📌 Overview

This repo holds my work for the Operating Systems course at Iran University of Science and Technology (2021). It has two parts. The first is **kernel work on MIT's xv6**, a small teaching OS. The second is a set of **Linux userland labs** that use the POSIX process, thread and IPC APIs directly.

The xv6 projects involve real kernel changes. They add fields to `struct proc`, hook the timer interrupt to count run and I/O ticks, and add system calls through the whole path (`syscall.h` → `syscall.c` → `sysproc.c` → `usys.S` → `user.h`). They also replace xv6's round-robin scheduler with a priority-based one. I did the scheduler project with Mohadeseh Jafari.

## ✨ Key Features

* **Priority scheduler for xv6:** Replaces round robin. On each pass it runs the `RUNNABLE` process with the lowest priority value (range 0–100, default 60). A new `set_priority(priority, pid)` syscall returns the old priority and calls `yield()` to force a reschedule. A `cps` syscall prints the process table with priorities.
* **Per-process time accounting + `waitx`:** Adds `stime`, `etime`, `rtime` and `iotime` to `struct proc`. `trap.c` updates `rtime` and `iotime` on every timer tick. `waitx(&wtime, &rtime)` works like `wait()` and also reports the child's waiting and running ticks.
* **`proc_dump` syscall:** Copies the PID and memory size of every `RUNNING` or `RUNNABLE` process into a user buffer, sorted by memory size, while holding the process-table lock. `userTestProcDump N` forks N children with different heap sizes to demonstrate it.
* **Thread-per-connection HTTP server:** Turns a one-client-at-a-time socket server on port 8090 into a concurrent one by calling `pthread_create` for each accepted connection.
* **Pipe-and-filter pipeline:** Rebuilds `ps -A -T | grep chrome` in C with `pipe()`, `fork()`, `dup()` and `execlp()`.
* **Concurrency experiments:** Threads in one process race to write a shared global array (compared with threads in separate processes). A semaphore built only from a mutex shows why that approach deadlocks once the count reaches zero.

## 🛠️ Tech Stack

* **Core Languages:** C, x86 assembly (xv6 syscall stubs)
* **Frameworks & Libraries:** POSIX — pthreads, `fork`/`exec`, `pipe`/`dup`, BSD sockets, System V shared memory
* **Infrastructure / Data:** xv6 (x86), QEMU, GCC, Make

## 🚀 Getting Started

### Prerequisites

Linux with GCC (32-bit support for xv6), Make, Perl and QEMU.

### Installation

```bash
git clone https://github.com/armanheydari/xv6-kernel-and-posix-labs.git
cd xv6-kernel-and-posix-labs
sudo apt install build-essential gcc-multilib qemu-system-x86   # Debian/Ubuntu
# one-time fixes to build xv6 with a modern toolchain
chmod +x scheduling-xv6/xv6-public/*.pl
sed -i 's/-Werror//' scheduling-xv6/xv6-public/Makefile
```

## 💻 Usage

**Boot xv6 with the priority scheduler and change a process's priority:**

```bash
cd scheduling-xv6/xv6-public
cp ../q2/proc.c proc.c        # use the Part 2 priority scheduler
make qemu-nox CPUS=1          # quit QEMU with Ctrl-A then X
$ set_priority_test 10 1      # (inside xv6) set PID 1's priority to 10
```

```
Name    PID  State     Priority
init    1    SLEEPING  60
...
Previous: 60
Name    PID  State     Priority
init    1    SLEEPING  10
```

**Userland labs:**

```bash
cd 3/q1 && make && ./server              # then: curl localhost:8090 from several terminals
gcc 5/q1.c -o pipeline && ./pipeline     # ps -A -T | grep chrome
gcc 3/q2.c -o threads -pthread && ./threads
```

## 📁 Project Structure

```
.
├── 1/                      # Linked list: insert, bubble sort by value, remove by name + value
├── 2/                      # Redirect.c: run a program with popen() and save its output to a file
├── 3/
│   ├── q1/                 # Multithreaded HTTP server (server.c + Makefile)
│   └── q2.c                # fork + pthreads: thread return values, nested forks, shared-array race
├── 4/                      # Semaphore built on a mutex, showing why it deadlocks at zero
├── 5/
│   ├── q1.c                # ps -A -T | grep chrome via pipe/dup/execlp
│   └── q2 - *.c            # Shared-memory writer/reader with a shift cipher (draft, doesn't compile)
├── phase1/
│   └── xv6-changed-files/  # proc_dump syscall: drop-in files for upstream xv6
└── scheduling-xv6/
    ├── q2/                 # Part 2: priority scheduler, waitx, set_priority, cps
    └── xv6-public/         # Full xv6 tree with timing fields + Part 3 multilevel-queue scheduler (WIP)
```

Each folder also has the original assignment brief (PDF, in Persian) and my write-up (`.docx`). xv6 is © MIT and distributed under its own license (`scheduling-xv6/xv6-public/LICENSE`).
