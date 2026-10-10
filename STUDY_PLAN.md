# Study Plan: From the Transistor to the Web Browser

A week-by-week way to work through [the outline in the README](README.md) using existing
books, courses and code. Nobody has published a finished version of this course, so each
week pairs a project from the outline with the best material for it.

**Pace.** 12 weeks at roughly 12–15 hours a week, matching the outline's own estimates.
Expect to overrun on the CPU, the compiler and the OS; that is normal. Stretch it to 24
weeks at half pace rather than skipping a project.

**Software only.** Everything runs in simulation (Verilator, QEMU). Section 7, real
hardware, is an optional extra at the end.

**Rule from the outline:** every project builds on the last. Your assembler targets your
CPU, your compiler feeds your assembler, your OS runs on your CPU, your browser runs on
your OS. When you can't keep the chain (say, your CPU is too slow to boot the OS), fall
back to QEMU for that stage and come back later.

---

## Week 0 (optional): Warm-up

If you have never built a CPU or written a compiler, do this first. It is the same journey
at toy scale and makes every later week easier.

- **Do:** [Nand2Tetris](https://www.nand2tetris.org/) Part 1, projects 1–6 (gates → CPU → assembler).
  Part 2 (VM, compiler, OS) can wait or be skipped.
- **Done when:** your Hack assembler's output runs on your Hack CPU.

---

## Section 1–2: Transistors and bringup (Week 1)

### Week 1 — Transistors, Verilog, LED, UART
- **Read:**
  - *Digital Design and Computer Architecture: ARM Edition* (Harris & Harris), chapters 1–4
    (transistors, combinational and sequential logic, HDL). This book is the spine of
    Sections 1–3.
  - [Verilator](https://www.veripool.org/verilator/) docs: install it and run the example.
  - [nandland](https://nandland.com/) UART tutorial.
- **Practise:** [HDLBits](https://hdlbits.01xz.net/) — work through the Verilog language and
  circuits sections until they feel easy.
- **Build:**
  1. Blink an LED (Verilog, ~10 lines) in Verilator; print the LED state from the C++ testbench.
  2. A UART transmitter and receiver (Verilog, ~100 lines), memory-mapped (MMIO).
  3. An echo testbench: bytes sent in come back out, and a byte toggles the LED.
- **Done when:** you type into a terminal, the simulated UART echoes it back.

---

## Section 3: Processor (Weeks 2–4)

### Week 2 — ARM assembly and the assembler
- **Read:**
  - Harris & Harris, chapter 6 (ARM architecture and assembly).
  - The ARM7TDMI Technical Reference Manual / ARMv4 instruction encodings — keep it open all week.
- **Build:** an assembler in Python (~500 lines): ARMv4 data processing, load/store,
  branches, labels. Output a flat binary.
- **Check:** assemble small programs and compare bytes against `arm-none-eabi-as` +
  `objcopy`, and run them in QEMU (`qemu-arm` or `qemu-system-arm`).
- **Done when:** your binaries are byte-identical to GNU's for a test suite of programs.

### Week 3 — ARM7 CPU, single-cycle then pipelined
- **Read:** Harris & Harris, chapter 7 (microarchitecture: single-cycle, multicycle, pipelined ARM).
- **Build:** an ARM7 CPU in Verilog (~1500 lines). Start single-cycle; then a 3-stage
  fetch/decode/execute pipeline. At least 1 MB of RAM in simulation.
- **Done when:** programs from your own assembler run on it and print over your UART.

### Week 4 — Finish the CPU, write the bootrom
- **Build:**
  - Missing instructions, condition codes, multiply, exceptions/interrupt entry.
  - A bootrom (ARM assembly, ~40 lines) baked into ROM: receives a program over the UART,
    copies it to RAM, jumps to it.
- **Check:** run the same test programs in QEMU and on your CPU and compare register dumps.
- **Done when:** you send a program over the simulated serial port and it runs.

---

## Section 4: Compiler (Weeks 5–7)

### Week 5 — A C compiler: lexer, parser, expressions
- **Read / follow:**
  - [chibicc](https://github.com/rui314/chibicc) by Rui Ueyama — a small C compiler built
    up one commit at a time. Read the commit history in order and redo each step yourself.
  - *Writing a C Compiler* (Nora Sandler) — chapter-by-chapter tests; targets x86 but the
    front end carries over.
  - Optional: *Crafting Interpreters* (free online) for parsing fundamentals.
- **Language:** the outline says Haskell. Use it if you want the challenge; C or Python is fine.
- **Build:** lexer, recursive-descent parser, AST; integer expressions, locals, `if`, `while`,
  functions. Emit ARM assembly for **your** assembler.
- **Done when:** recursive `fib` compiles and runs on your CPU.

### Week 6 — Finish the compiler, write the linker
- **Build:**
  - Compiler: pointers, arrays, structs, `char`, strings, globals — enough C to write a libc.
  - Linker in Python (~300 lines): change the assembler to emit object files with relocations,
    then link them into ELF.
- **Read:** *Linkers and Loaders* (John Levine), chapters on object files and relocation;
  the ELF spec.
- **Done when:** a program split across files links into an ELF that runs in QEMU with
  semihosting.

### Week 7 — libc, malloc, ethernet, bootloader
- **Build:**
  - libc (C, ~500 lines): `memcpy`, `memset`, `strlen`, `printf`, `malloc`/`free`
    (start with a free-list allocator). No syscall wrappers yet.
  - Ethernet MAC (Verilog, ~200 lines), modelled in simulation.
  - Bootloader in C (~300 lines): loads a kernel over UDP instead of serial.
- **Read:** K&R appendix B for libc behaviour; Ethernet frame and UDP header formats.
- **Done when:** your bootloader, compiled by **your** compiler, fetches a program over
  simulated ethernet and runs it.

---

## Section 5: Operating system (Weeks 8–10)

### Week 8 — MMU and the kernel's skeleton
- **Read:**
  - [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/) (free): virtualisation chapters —
    processes, scheduling, address translation, paging, TLBs.
  - The ARMv5 (ARM9) MMU chapter of the ARM Architecture Reference Manual.
- **Build:** an MMU in Verilog (~1000 lines): page tables, TLB, faults. Kernel entry,
  exception vectors, a page allocator.
- **Done when:** the kernel turns on paging and keeps running.

### Week 9 — Processes and syscalls
- **Follow:** [xv6](https://pdos.csail.mit.edu/6.1810/) and the MIT 6.1810 labs. xv6 is a
  small Unix you can read in a weekend; use it as the reference design for your own.
- **Build (C, ~2500 lines across weeks 9–10):** `fork`, `execve`, `wait`, `exit`, `sleep`;
  `mmap`, `munmap`, `mprotect`; a simple round-robin scheduler. Pick a debug path early —
  `printf` over UART at first, a gdb remote stub if you get stuck often.
- **Done when:** two user processes run at once and print interleaved output.

### Week 10 — SD card, FAT, userland
- **Read:** OSTEP, persistence chapters (files, directories, file system implementation);
  the FAT specification.
- **Build:**
  - SD card controller (Verilog, ~150 lines) and its driver.
  - FAT filesystem (C, ~300 lines): `open`, `read`, `write`, `close`.
  - Userland (C, ~250 lines): `init`, a shell, `cat`, `ls`, `rm`, `download`.
- **Done when:** you boot to a shell and `cat` a file from the simulated SD card.

---

## Section 6: Browser (Week 11)

### Week 11 — TCP, telnetd, dynamic linking, the browser
- **Read / follow:**
  - [Let's code a TCP/IP stack](https://www.saminiir.com/lets-code-tcp-ip-stack-1-ethernet-arp/)
    (Sami Niiranen) and its [level-ip](https://github.com/saminiir/level-ip) code — ARP, IPv4,
    ICMP, TCP, step by step.
  - [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/) for the sockets API
    your syscalls should provide.
  - [Web Browser Engineering](https://browser.engineering) (free) — the first chapters
    (downloading pages, drawing text) map directly onto a text browser.
  - Optional: *TCP/IP Illustrated, Vol. 1* (Stevens) when the details matter.
- **Build:**
  - TCP stack in the kernel (C, ~500 lines) and `socket`/`bind`/`connect`/`send`/`recv`.
  - `telnetd` (C, ~50 lines): a multi-connection bind shell.
  - Dynamic linking (C, ~300 lines): a user-space dynamic linker and linker changes.
  - A text web browser (C, 500+ lines): HTTP/1.0 GET, strip and lay out HTML, ANSI output.
- **Done when:** your browser, on your OS, on your CPU, loads a page from a server on your
  host machine.

---

## Section 7: Real hardware (Week 12, optional)

### Week 12 — Onto an FPGA
The outline's version (design and reflow your own board) is a project of its own. A
cheaper route that keeps the spirit:

- Buy an off-the-shelf FPGA board with UART, ethernet and an SD slot (any board supported
  by an open toolchain such as yosys + nextpnr is easiest).
- Synthesize your CPU, UART, ethernet and SD controllers for it; fix whatever simulation
  forgave (timing, reset, clock domains).
- Optional: write the JTAG bit-banging code (C, ~200 lines) from the outline.
- **Done when:** the whole stack from weeks 1–11 runs on a real chip.

---

## Whole-stack companions

Read alongside, whenever a section feels thin:

- **Project Oberon** (Niklaus Wirth, free) — one book that designs an FPGA CPU, compiler,
  OS and GUI. The closest existing thing to this whole course.
- **Computer Systems: A Programmer's Perspective** (Bryant & O'Hallaron) — the bridge from
  hardware to C, linking and memory.
- **Ben Eater's** breadboard computer videos — if you want to see the hardware side physically.

## Other people's attempts

Useful for comparing notes, though none are complete:
[techmexdev](https://github.com/techmexdev/fromthetransistor),
[dxlnr](https://github.com/dxlnr/fromthetransistor),
[davidmeadejr](https://github.com/davidmeadejr/fromthetransistors),
[fromthetransistor.com](https://www.fromthetransistor.com/) (a RISC-V write-up, partly unfinished).
