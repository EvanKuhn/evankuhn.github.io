---
layout: post
title: "Linux Signals and Process State"
author: Evan Kuhn
date: 2016-06-26 19:22:00 -0700
categories: linux
description: An overview of Linux signals, signal handling, and process states. And zombies!
---

## Intro to Signals

Signals are one of the oldest forms of **interprocess communication** in Linux.  They are used to to signal asynchronous events to one or more processes.  They can be sent to a user-space process either from the kernel, or from another user process.

There are restrictions regarding which processes a given process can signal:

- The kernel can signal any process.
- Superusers can signal any process.
- A process may signal another process with the same uid and gid.
- A process may signal another process in the same process group.

To view a list of signals, use `kill -l`:

```bash
$ kill -l
 1) SIGHUP      2) SIGINT      3) SIGQUIT     4) SIGILL
 5) SIGTRAP     6) SIGABRT     7) SIGEMT      8) SIGFPE
 9) SIGKILL    10) SIGBUS     11) SIGSEGV    12) SIGSYS
13) SIGPIPE    14) SIGALRM    15) SIGTERM    16) SIGURG
17) SIGSTOP    18) SIGTSTP    19) SIGCONT    20) SIGCHLD
21) SIGTTIN    22) SIGTTOU    23) SIGIO      24) SIGXCPU
25) SIGXFSZ    26) SIGVTALRM  27) SIGPROF    28) SIGWINCH
29) SIGINFO    30) SIGUSR1    31) SIGUSR2
```

## Sending and Receiving

You may send a signal either in code, or from the command line:

```c
int kill(pid_t pid, int sig);   # From C code
kill [-s signal_name] pid       # From the command line
```

To receive a signal, the process defines a **signal handler,** which is a function that is run when the signal is received:

```c
# Use this to register a signal handler
typedef void (*sighandler_t)(int);
sighandler_t signal(int sig, sighandler_t handler);
```

Each signal has a **default action.**  This could be: create a core image, stop the process, or terminate the process.  For a list of default actions, see `man kill`.

You may set a signal to be **ignored**, in which case no handler code is run.

Note that `SIGKILL (9)` and `SIGSTOP (17)` cannot be caught or ignored.

## Context Switching

When a signal arrives, the operating system stops the process, saves all registers and process state, and runs the interrupt handler code.  Thus, a signal causes a **context switch.**

When the signal handler returns, the OS performs another context switch to continue running the process where it left off.  This is done via the `sigreturn` system call, which the OS executes.

## Signal Handler Tips

Your signal handling routine is called each time a signal is received.  Because a signal can be received multiple times, your function should be **reentrant.**  Furthermore, because the process can be interrupted at any time, you should be careful about reading or writing any global state.  

Ideally, your signal handler should be as **simple** as possible.  Either set a flag and return, or clean up and exit the program immediately.

## Zombie Processes, wait, and SIGCHLD

When a process exits, its exit status is recorded in the operating system's process table.  This entry is kept around until the parent process reads it via one of the 'wait' system calls.

If the parent process does not call 'wait' on a child process, that child process is said to be a **zombie** (or **defunct**).  A zombie process cannot be killed; rather, you must kill the parent process, at which point the child will be reparented under the init process (pid 1), which should immediately wait on the child, causing the OS to remove its process entry (though I've seen zombies under init).

Until the child process has been wait-ed on by its parent, it remains in the defunct/zombie process state.  The state transition looks like this:

1. Process is in RUNNABLE state
1. Process exits
1. Process moved to ZOMBIE state
1. Parent calls `wait` on child
1. Process entry is removed; process no longer exists!

The parent process is notified that a child has exited via the `SIGCHLD` signal.  In fact, this signal is used to notify the parent of any interesting state change (exit, crash, trap, stop, continue, etc).

By default, `SIGCHLD` signals are ignored.  However, you may explicitly ignore a `SIGCHLD` to prevent children from becoming zombies:

```c
signal(SIGCHLD, SIG_IGN);
```

To repeat, you must explicitly ignore the `SIGCHLD` signal to prevent zombies; it's not enough to leave the default ignore behavior.

## The Parent Death Signal

Linux allows you to set a signal that a process should receive when its parent dies.  This is recorded in the `pdeath_signal` variable, which is initialized to 0 (none) during a `fork`.

To be specific, the parent-death signal is sent when the thread in the parent process that performed the fork dies; the parent process may still be running.  Be careful of this.

The pdeath_signal variable can be read or written via `prctl`:

```c
prctl(PR_SET_PDEATHSIG, sig);     # Set the parent-death signal
prctl(PR_GET_PDEATHSIG, &sig);    # Get the parent-death signal
```

This is useful for services that fork a number of worker processes.  If the manager process dies, the workers can be notified and clean themselves up, rather than remaining as **orphans.**

> **Warning**: the "parent" in this case is considered to be the thread that created this process.  In other words, the signal will be sent when that thread terminates (via, for example, pthread_exit(3)), rather than after all of the threads in the parent process terminate."

(Source: [man prctl](http://man7.org/linux/man-pages/man2/prctl.2.html))

## Trapping Signals

You can use the `trap` command to handle signals within a script:

```bash
trap "command to run" <signum> [signum] ...
```

Once this is run, if the current script receives one of the signals listed, it will run the command.

For example, this will remove some tempfiles when `SIGHUP (1)` or `SIGINT (2)` are received:

```bash
trap "rm $WORKDIR/temp1 $WORKDIR/temp2; exit" 1 2
```

Note the call to `exit`.  This causes the script to exit after the command is run.  Without it, the script will continue to execute.

To ignore signals, use an empty command string:

```bash
trap '' 2 15
```

To reset signal handling, omit the command:

```bash
trap 2 15
```

## Tracing

Signals are used by OS facilities like `ptrace` to trace a process.  When a process is traced, it becomes a child of the 'tracer' process.  As a parent, it will be notified of state changes to the traced process via `SIGCHLD`.

Next, ptrace puts the traced process in the STOP state.  It then configures the OS to stop the process at various events or locations during execution.  Then, when one of these events occurs, the OS places the traced process in the STOP state and signals the parent.  To continue running the child, it moves it to the running state.

## Process States

```
D	Uninterruptible sleep (usually IO)
R	Running or runnable (on run queue)
S	Interruptible sleep (waiting for an event to complete)
T	Stopped, either by a job control signal or because it is being traced
Z	Defunct ("zombie") process, terminated but not reaped by its parent
W	Paging (not valid since the 2.6.xx kernel)
X	Dead (should never be seen)
```

## SIGSTOP and SIGCONT

A process can be stopped and restarted via the `SIGSTOP` and `SIGCONT` signals.  One way to stop a process is via `Ctrl-Z`, which sends a `SIGSTOP`:

```bash
$ sleep 100
^Z                            # Pressed Ctrl+Z
[1]+  Stopped

$ ps -o pid,state,command
  PID S COMMAND
13224 T sleep 100             # Process in stopped state (T)
```

Or you can use `kill` to send both `SIGSTOP` and `SIGCONT`:

```bash
$ sleep 100 &
[1] 41992

$ ps -o pid,state,command
  PID STAT COMMAND
41992 S	sleep 100             # Process in sleep state (S)

$ kill -STOP 41992
[1]+  Stopped

$ ps -o pid,state,command
  PID STAT COMMAND
41992 T	sleep 100             # Process in stopped state (T)

$ kill -CONT 41992

$ ps -o pid,state,command
  PID STAT COMMAND
41992 S	sleep 100             # Process back in sleep state (S)
```

## References

[1] Brouwer, Andries. ["The Linux Kernel", Chapter 5: Signals.](https://www.win.tue.nl/~aeb/linux/lk/lk-5.html) 01 Feb 2003. <br/>
[2] ["Unix - Signals and Traps."](http://www.tutorialspoint.com/unix/unix-signals-traps.htm) TutorialsPoint. <br/>
[3] Rusling, David A. ["The Linux Kernel", Chapter 5: Interprocess Communication Mechanisms.](http://www.tldp.org/LDP/tlk/ipc/ipc.html) <br/>
[4] ["Linux process states."](https://idea.popcount.org/2012-12-11-linux-process-states/) idea.popcount.org. 11 Dec 2012.<br/>
