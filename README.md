ByteOS
A tiny UEFI operating system experiment

ByteOS is an experimental x86-64 UEFI project built around one simple idea:

How small can a bootable operating system be?

The current BOOTX64.EFI is only 576 bytes.

It is a hand-built PE32+ UEFI application written entirely in NASM assembly. When executed, ByteOS intentionally does almost nothing: it disables interrupts and halts the CPU in an infinite loop.

No graphics.
No filesystem driver.
No kernel framework.
No standard library.
No operating-system services.

Just a tiny UEFI executable.

✨ Features
🧩 x86-64 UEFI application
⚡ Written in pure NASM assembly
📦 Hand-built PE32+ executable
🪶 Only 576 bytes
🖥️ Boots without displaying anything
🚫 No C runtime
🚫 No operating-system dependencies
🚫 No emulator required
🔧 Designed as a learning/size-optimization experiment
📁 Project structure
ByteOS/
├── src/
│   ├── boot.asm
│   └── linker.ld
│
├── build/
│   ├── BOOTX64.EFI
│   └── ByteOS.img
│
└── README.md


linker.ld is retained from the earlier linker-based version of the project. The current tiny EFI build is generated directly from boot.asm.

🔬 How it works

The entire program eventually reaches this:

cli

.hang:
    hlt
    jmp .hang


That's it.

CLI disables maskable interrupts.

HLT stops the processor until an interrupt occurs.

Because interrupts are disabled and execution immediately loops back to HLT, ByteOS effectively sits there doing nothing forever.

The result is intentional:

UEFI
 │
 ▼
ByteOS
 │
 ▼
blank screen
 │
 ▼
CPU halted

🧱 Building
Requirements

On Arch Linux:

sudo pacman -S nasm


Build the EFI executable:

nasm -f bin src/boot.asm -o build/BOOTX64.EFI


Check its size:

stat -c '%s bytes' build/BOOTX64.EFI


Current target:

576 bytes


Check the executable:

file build/BOOTX64.EFI


Expected:

PE32+ executable for EFI (application), x86-64

💾 FAT32 UEFI image

A UEFI-compatible FAT32 image can contain the standard fallback boot path:

EFI/
└── BOOT/
    └── BOOTX64.EFI


Create an image:

truncate -s 33M build/ByteOS.img
mkfs.fat -F 32 -n BYTEOS build/ByteOS.img


Mount it:

mkdir -p build/mnt
mount -o loop build/ByteOS.img build/mnt


Install ByteOS:

mkdir -p build/mnt/EFI/BOOT
cp build/BOOTX64.EFI build/mnt/EFI/BOOT/BOOTX64.EFI
sync
umount build/mnt


The FAT32 image is intentionally larger than the EFI executable because FAT32 requires filesystem metadata and allocation structures.

🧪 Testing

Testing with a UEFI virtual machine is recommended before writing anything to physical media.

For QEMU + OVMF:

qemu-system-x86_64 \
  -drive if=pflash,format=raw,readonly=on,file=/path/to/OVMF_CODE.4m.fd \
  -drive if=pflash,format=raw,file=build/OVMF_VARS.fd \
  -drive format=raw,file=build/ByteOS.img

Expected behavior

If ByteOS successfully loads:

Nothing happens.

That is expected.

There is deliberately no text output, graphics, keyboard handling, shell, or user interface.

⚠️ Current status

ByteOS is an experimental project, not a general-purpose operating system.

The current implementation is intended primarily to explore:

UEFI boot processes
PE/COFF structure
x86-64 assembly
extremely small executables
firmware compatibility
filesystem boot layouts
binary size optimization

The tiny hand-crafted PE structure may not be accepted by every UEFI implementation.

Always test in QEMU/OVMF before using physical hardware.

🗺️ Roadmap

The project is intentionally starting from almost nothing.

Possible future milestones:

 Reliable OVMF boot
 Boot on real UEFI hardware
 Reduce executable size further
 Display a single character
 Read the UEFI memory map
 Initialize basic graphics
 Add a tiny kernel
 Keyboard input
 Simple command shell
 Custom filesystem
 Multiboot experiments
 See how small a useful OS can become
📏 The size challenge

The fun part of ByteOS is treating executable size as a constraint.

Current:

BOOTX64.EFI
     │
     └── 576 bytes


The long-term goal is not simply to make a tiny file.

The goal is to discover:

What is the smallest useful x86-64 UEFI operating system we can build?

🤝 Contributing

Ideas, experiments, size optimizations, firmware compatibility reports, and pull requests are welcome.

If you discover a way to remove even a few bytes, that's a worthwhile contribution.

📜 License

Choose a license appropriate for your project. For example, MIT:

MIT License

Copyright (c) 2026 ByteOS contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files, to deal in the Software
without restriction, including without limitation the rights to use, copy,
modify, merge, publish, distribute, sublicense, and/or sell copies of the
Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

⭐ Why?

Because sometimes the best way to understand an operating system is to start with one that does almost nothing.

576 bytes.

One EFI executable.

One infinite loop.

ByteOS.
