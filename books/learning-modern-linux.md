# Learning Modern Linux

## Chapter1. Introduction to Linux

### Modern Environment

- mobile device, cloud computing, IOT, diverse processor architecture

### What is Operating system?

- abstracts different hardware component
- provides clean and nicely designed API to handle them
- API: system calls, syscalls

### Linux Distributions

- distribution = bundling of kernel, related components. \
   including, package management, file system, init system, and a shell

### Resource Visibility

- resource = anything that can be used to execute software.
  - hardware, and its abstractions(=CPU, RAM), file systems, har disk drives, SSD, processes, networking-related stuff like devices or routing table, credential representing users
- visibility
  - traditionally: global view on resource
  - now: can be global, or local
- in docker/k8s environment, there can be many process with same pid

## Chapter2. The Linux Kernel

### Linux Architecture

- 3 classifications
  - Hardware
    - CPU, memory, peripheral devices such as keyboards, monitors
  - The kernel
    - between hardware and the user land
    - strictly speaking, init system and system services(networking etc.) are not part of linux kernel
  - User land
    - where apps are running.
    - including system components like shells
- Between the kernel and the user land is the interface called system calls
- kernel mode: fast execution with limited abstraction
- user mode: comparatively slower but safer, more convenient

### CPU Architectures

- The BIOS and UEFI
  - BIOS(Basic I/O System): used to bootstrap linux/unix
  - replaced by UEFI(Unified Extensible Firmware Interface)
- x86 architecture
  - developed by Intel, later licensed to AMD
  - x86: Intel 64 bit processor
  - x64: intel 32 bit processor
  - amd64: AMD 64 bit processor
  - powerful and widely available
  - not energy efficient
- ARM architecture
  - RISC(Reduced Instruction Set Computing)
  - portable devices. increasing usages: AWS Graviton(Data center)
  - fast, cheap, produces less heat
  - vulnerable
- RISC-V architecture
  - pronounce: risk five
  - implementations exists: Alibaba Group, Nvidia
  - relatively new and not widely used

### Kernel Components

- monolithic = one binary file
- function blocks
  - process management
  - memory management
  - networking
  - file system
  - management of character devices and device drivers

#### Process Management

- process: user-facing unit. based on executable program
- thread: unit of execution in the context of a process
- glossary
  - sessions
    - contains one or more process groups
    - represent high level user-facing unit
    - optionally tty attached \
       (tty: input device. physical/virtual. console, terminal. originated from teletypwrite)
    - identified by session id(SID)
  - process group
    - contains one or more processes
    - at most one process group in a session as the foreground
    - identified by process group id(PGID)
  - process
    - abstraction that group multiple resources(address space, threads, sockets, ...)
    - current process is exposed as `/proc/self`
    - identified by process id(PID)
  - threads
    - implemented by the kernel
    - no dedicated data structure representing thread
    - rather a process that shares certain resources
    - identified by thread id(TID), thread group id(TGID)
  - tasks
    - process, and threads are scheduled by the data structure called `task_struct`
    - have scheduling information, identifiers, signal handlers, other info like performance and security
    - not exposed outside of kernel

```bash
$ ps -j
USER   PID  PPID  PGID   SESS JOBC STAT   TT       TIME COMMAND
user 11470 11468 11470      0    1 S    s000    0:00.02 -zsh
```

#### Memory Management

- physical memory, virtual memory
  - virtual memory mapped to physical memory
  - paging
  - to effectively map virtual memory address -> physical memory \
    CPU architecture supports TLB buffer(translation lookaside buffer). small cache
  - traditionally default page size: 4KB. from kernel v2.6.3 supports huge page \
     ex) 64 bit Linux allows to use up to 128TB of virtual memory address space per process \
     with approx. 64TB of physical memory(= amount of RAM on the device)
- each process has virtual memory
  - managed by page table

#### Networking

- sockets
  - abstracting communication
- TCP(Transmission Control Protocol), UDP(User Datagram Protocol)
  - connection-oriented communication and connectionless communication
- IP(Internet Protocol)
  - addressing machine

#### FileSystem

- filesystem organize files and directories on storage device like HDD, SSD, or flash memory
- many types: ext4, btrfs, NTFS
- VFS(Virtual File System)
  - supports multiple filesystem types and instances.
  - provides a common API abstraction of functions like open, close, read, and write(highest level)
  - built on filesystem abstraction called plug-ins(bottom level)

#### Device Drivers

- driver: a code that runs in the kernel
- manages a device: actual device like keyboard, pseudo-device like pseudo-terminal
- GPU: interesting one. used for graphics previously, now hot with machine learning
- driver may be statically built in the kernel, \
  dynamically loaded as module

#### syscalls

- syscall: service interface that kernel exposes, and that the user land entity calls
- linux has about 300 syscalls. depending on CPU family
- usually don't directly invoke syscalls. \
  - but via C standard library(= wrapper functions). ex) glibc, musl
  - handles repetitive low level handling
- syscalls: software interrupts
- syscall execution steps
  - sys_call_table: array of function pointers in memory
  - C library calls sys calls -> (interrupt) \
     -> kernel saves the current context to the stack \
     -> jump to the function(number index in sys_call_table)
    -> restore context after finishing

```bash
$ strace ls
$ strace -c \
    curl -s https://mhausenblas.info > /dev/null
```

### Kernel Extensions

- how to add features to the kernel source code

#### modules

- a program that you can load into a kernel on demand
- third party may provider better module
- modern way: eBPF
  - naming
    - BPF: Berkley Packet Filter
    - eBPF: no meaning
  - feature of linux kernel(built-in)
  - safely and efficiently extend the linux kernel by `bpf`` syscall
  - example
    - CNI plug-in to enable pod networking in Kubernetes(Cilium, ...)
    - for observability: linux kernel tracing(iovisor/bpftrace)
    - for security control: container runtime scanning(CNCF Falco)
    - for network load balancing: Facebook's L4 katran library
- [intro to ebpf](https://ebpf.io/what-is-ebpf/)
  - background
    - os is an ideal place to implement observability, security and networking due to kernel's privileged ability to oversee and control the entire system
    - but hard to evolve due to central role and high requirement to stability and security
  - what does ebpf do
    - allows sandboxed program to run within the operating system
    - with the aid of JIT(Just-In-Time) compiler and verification engine
  - how it works
    - event driven. by hook
    - predefined hooks: system calls, function entry/exit, kernel tracepoints(= hook point), network events, etc.
    - process -> execv() syscall -> ePPF hook called -> then to scheduler
  - how to write eBPF programs?
    - usually via projects like cilium, bcc, bpftrace
    - should be in the form of bytecode

It allows sandboxed programs to run within the operating system, which means that application developers can run eBPF programs to add additional capabilities to the operating system at runtime. The operating system then guarantees safety and execution efficiency as if natively compiled with the aid of a

## Chapter3. Shells and Scripting
### Basics
- terminal
  - a program that privdes a textual user interface
  - environment variable `TERM` : terminal emualtor
- shell
  - a program that runs inside the terminal and acts as a command interperter
  - commonly defined in `sh`. POSIX shell
  - originally there was Bourne shell sh. -> bash: Bourne Again Shell.
- streams
  - io = input, output
  - 3 default file descriptor(FD)
    - stdin(FD 0)
    - stdout(FD 1)
    - stderr(FD 2)
  - can redirect with `$FD>`, `<$FD`
  - `1>`, `>`: stdout
  - `&>`: both stdout and stderr 
  - `/dev/null`: to get rid of a stream
- special characters
  - `&`(ampersand): run as background
  - `\`(backslash): continue command with newline
  - `|`(pipe): connect stdout of one process to stdin of next process
- variables
  - environment variable
    - global
    - list with `env`
  - shell vairaible
    - valid in the context of current execution
    - list with `set`
  - common
    - IFS: list of characters to seperate field
    - ?: exit status
    - $: current process id
    - 0: current process name
- exit status
  - $?
  - 0: successful
  - non-zero: unsuccessful
  - $PIPESTATUS: array of exit statuses of piped commands
- built-in commands
  - /usr/bin: user command
  - /usr/sbin: administrative command
- job control
  - background, foreground
  - `&`: run as background
  - `ctrl + z`: sned to background
  - `nohup`: keep running with shell closed
  - `disown`: make a process run like nohup
- modern commands
  - exa: ls
  - bat: syntax highlighting, show non-printable characters, suppors git, integrated pager
  - rg: find & grep, with file name, line number
  - jq: json parsing
- common tasks
  - `alias`: shorten
  - navigating shortcuts
  - viewing long files: head, tail, less, ba
  - dateime handling: date

### human friendly shells
- fish shell
    - autosuggestion
    - $status instead of $?
    - configuration ui
- zshell
  - oh my zshell: theming
- others
  - oil shell
  - murex
  - nushell
  - powershell

### terminal multiplexer
- screen: old, outdated

#### tmux
- glossaries
  - sesssion
     - a logical unit of a working environment for one task
     - a conatiner for other units
  - window
    - a tab in a browser
    - belongs to a sesion
  - panes
    - a single shell instance running
- trigger: `ctrl + b` (default)
- commands: https://tmuxcheatsheet.com/
- tpm: tmux plugin manager
  - tmux-resurrect: restore session
  - tmux-continuum: automatically save/restores session

### Scripting