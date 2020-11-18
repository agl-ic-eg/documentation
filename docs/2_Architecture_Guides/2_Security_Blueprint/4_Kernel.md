---
title: Kernel
---

**System Hardening:** Best practices associated with the configuration of an
embedded Linux based operating system. This section includes both hardening of
the kernel itself, as well as specific configurations and patches used to
protect against known vulnerabilities within the build and configuration of the
root filesystem.

At the Kernel level, we must ensure that no console can be launched. It could be
used to change the behavior of the system or to have more information about it.
Another aspect is the protection of the memory used by the Kernel.

The next sub-sections contain information on various kernel configuration
options to enhance the security in the kernel (3.10.17) and also for
applications compiled to take advantage of these security features.
Additionally, there are also configuration options that protect from known
vulnerable configuration options. Here's a high level summary of various kernel
configurations that shall be required for deployment.

## Kernel Version

The choice of kernel version for the AGL system is essential to its security.
Depending on the type of board and eventual production system, different kernel
versions are used. For example, one of the systems under study uses the Linux
kernel version 3.10, while another uses the Linux kernel version 4.4. For the
Linux kernel version 3.10.31, there are 25 known vulnerabilities. These
vulnerabilities would allow an attacker to gain privileges, bypass access
restrictions, allow memory to be corrupted, or cause denial of service. In
contrast, the Linux kernel version of 4.4 has many fewer known vulnerabilities.
For this reason, we would in general recommend the later kernel version as a
basis for the platform.

Note that, although there are fewer known vulnerabilities in the most recent
kernel versions there may be many unknown vulnerabilities underlying. A rule of
thumb is to update the kernel as much as possible to avoid the problems you do
know, but you should not be complacent in the trust that you place in it. A
defense-in-depth approach would then apply.

If there are constraints and dependencies in upgrading to a newer kernel version
(e.g. device drivers, board support providers) and you are forced to an older
Linux kernel version, there need to be additional provisions made to reduce the
risk of kernel exploits, which can include memory monitoring, watch-dog
services, and system call hooking. In this case, further defense-in-depth
techniques may be required to mitigate the risk of attacks to known
vulnerabilities, which can also include runtime integrity verification of
components that are vulnerable to tampering.

## Kernel Build Configuration

The kernel build configuration is extremely important for determining the level
of access to services and to reduce the breadth of the attack surface. Linux
contains a great and flexible number of capabilities and this is only controlled
through the build configuration. For example, the `CONFIG_MODULES` parameter
allows kernel modules to be loaded at runtime extending the capabilities of the
kernel. This capability needs to be either inhibited or controlled at runtime
through other configuration parameters. For example, `CONFIG_MODULE_SIG_FORCE=y`
ensures that only signed modules are loaded. There is a very large number of
kernel configuration parameters, and these are discussed in detail in this
section.

# General configuration

## Mandatory Access Control

Kernel should controls access with labels and policy.

Domain               | `Config` name  | `Value`
-------------------- | -------------- | --------------------------------------
Kernel-General-MAC-1 | CONFIG_IP_NF_SECURITY   | m
Kernel-General-MAC-2 | CONFIG_IP6_NF_SECURITY  | m
Kernel-General-MAC-3 | CONFIG_EXT2_FS_SECURITY | y
Kernel-General-MAC-4 | CONFIG_EXT3_FS_SECURITY | y
Kernel-General-MAC-5 | CONFIG_EXT4_FS_SECURITY | y
Kernel-General-MAC-6 | CONFIG_SECURITY         | y
Kernel-General-MAC-7 | CONFIG_SECURITY_SMACK   | y
Kernel-General-MAC-8 | CONFIG_TMPFS_XATTR      | y

Please also refer to the [Mandatory Access Control documentation in
Platform](5_Platform.md). You can also find useful documentation and
links on wikipedia about
[**MAC**](https://en.wikipedia.org/wiki/Mandatory_access_control) and about
[**SMACK**](https://en.wikipedia.org/wiki/Simplified_Mandatory_Access_Control_Kernel).

--------------------------------------------------------------------------------

## Disable kexec

**Kexec** is a system call that enables you to load and boot into another kernel
from the currently running kernel. This feature is not required in a production
environment.



Domain                 | `Config` name  | `Value`
---------------------- | -------------- | -------
Kernel-General-kexec-1 | `CONFIG_KEXEC` | `n`





**kexec** can load arbitrary kernels but signing of new kernel can be enforced
like it is can be enforced for new modules.

--------------------------------------------------------------------------------

## Disable kernel IP auto-configuration

It is preferable to have an IP configuration performed using a user-space tool
as these tend to have more validation. We do not want the network interface
coming up until the system has come up properly.



Domain                      | `Config` name   | `Value`
--------------------------- | --------------- | -------
Kernel-General-IPAutoConf-1 | `CONFIG_IP_PNP` | `n`



--------------------------------------------------------------------------------

## Disable Sysctl syscall support

Enabling this will result in code being included that is hard to maintain and
not well tested.



Domain                          | `Config` name           | `Value`
------------------------------- | ----------------------- | -------
Kernel-General-SysCtl_SysCall-1 | `CONFIG_SYSCTL_SYSCALL` | `n`



--------------------------------------------------------------------------------

## Disable Legacy Linux Support

There are some Kernel Configs which are present only to support legacy binaries.
See also "Consoles" part in order to disabling support for legacy binary
formats. The `uselib` system call, in particular, has no valid use in any
`libc6` or `uclibc` system in recent times. This configuration is supported in
**Linux 3.15 and greater** and thus should only be disabled for such versions.



Domain                       | `Config` name   | `Value`
---------------------------- | --------------- | -------
Kernel-General-LegacyLinux-1 | `CONFIG_USELIB` | `n`



--------------------------------------------------------------------------------

## Disable firmware auto-loading user mode helper

The firmware auto loading helper, which is a utility executed by the kernel on
`hotplug` events requiring firmware, can to be set `setuid`. As a result of
this, the helper utility is an attractive target for attackers with control of
physical ports on the device. Disabling this configuration that is supported in
**Linux 3.9 and greater**.



Domain                      | `Config` name                  | `Value`
--------------------------- | ------------------------------ | -------
Kernel-General-FirmHelper-1 | `CONFIG_FW_LOADER_USER_HELPER` | `n`





It doesn't strictly need to be `setuid`, there is an option of shipping firmware
builtin into kernel without initrd/filesystem.



--------------------------------------------------------------------------------

## Enable Kernel Panic on OOPS

When fuzzing the kernel or attempting kernel exploits attackers are likely to
trigger kernel OOPSes. Setting the behavior on OOPS to PANIC can impede their
progress.

This configuration is supported in **Linux 3.5 and greater** and thus should
only be enabled for such versions.



Domain                       | `Config` name          | `Value`
---------------------------- | ---------------------- | -------
Kernel-General-PanicOnOOPS-1 | `CONFIG_PANIC_ON_OOPS` | `y`



--------------------------------------------------------------------------------



## Disable socket monitoring interface

These monitors can be used to inspect shared file descriptors on Unix Domain
sockets or traffic on 'localhost' which is otherwise assumed to be confidential.

The `CONFIG_PACKET_DIAG` configuration is supported in **Linux 3.7 and greater**
and thus should only be disabled for such versions.

The `CONFIG_UNIX_DIAG` configuration is supported in **Linux 3.3 and greater**
and thus should only be disabled for such versions.



Domain                     | `Config` name        | `Value`
-------------------------- | -------------------- | -------
Kernel-General-SocketMon-1 | `CONFIG_PACKET_DIAG` | `n`
Kernel-General-SocketMon-2 | `CONFIG_UNIX_DIAG`   | `n`



--------------------------------------------------------------------------------

## Disable BPF JIT

The BPF JIT can be used to create kernel-payloads from firewall table rules.

This configuration for is supported in **Linux 3.16 and greater** and thus
should only be disabled for such versions.



Domain                   | `Config` name    | `Value`
------------------------ | ---------------- | -------
Kernel-General-BPF_JIT-1 | `CONFIG_BPF_JIT` | `n`



--------------------------------------------------------------------------------

## Enable Enforced Module Signing

The kernel should never allow an unprivileged user the ability to load specific
kernel modules, since that would provide a facility to unexpectedly extend the
available attack surface.

To protect against even privileged users, systems may need to either disable
module loading entirely, or provide signed modules (e.g.
`CONFIG_MODULE_SIG_FORCE`, or dm-crypt with LoadPin), to keep from having root
load arbitrary kernel code via the module loader interface.

This configuration is supported in **Linux 3.7 and greater** and thus should
only be enabled for such versions.



Domain                         | `Config` name             | `Value`
------------------------------ | ------------------------- | -------
Kernel-General-ModuleSigning-1 | `CONFIG_MODULE_SIG_FORCE` | `y`



It is also possible to block the loading of modules after startup with
"kernel.modules_disabled".



Domain                         | `Variable` name           | `Value`
------------------------------ | ------------------------- | -------
Kernel-General-ModuleSigning-2 | `kernel.modules_disabled` | `1`



--------------------------------------------------------------------------------



## Disable all USB, PCMCIA (and other `hotplug` bus) drivers that aren't needed

To reduce the attack surface, the driver enumeration, probe, and operation
happen in the kernel. The driver data is parsed by the kernel, so any logic bugs
in these drivers can become kernel exploits.



Domain                   | Object              | _State_
------------------------ | ------------------- | ----------
Kernel-General-Drivers-1 | `USB`               | _Disabled_
Kernel-General-Drivers-2 | `PCMCIA`            | _Disabled_
Kernel-General-Drivers-3 | Other `hotplug` bus | _Disabled_



--------------------------------------------------------------------------------

## Position Independent Executables



Domain                           | Improvement
-------------------------------- | -----------------------------
Kernel-General-IndependentExec-1 | Kernel or/and platform part ?





Domain                           | `compiler` and `linker` options | _State_
-------------------------------- | ------------------------------- | --------
Kernel-General-IndependentExec-1 | `-pie -fpic`                    | _Enable_



Produce a position independent executable on targets which supports it.

--------------------------------------------------------------------------------

## Prevent Overwrite Attacks

`-z,relro` linking option helps during program load, several ELF memory sections
need to be written by the linker, but can be turned read-only before turning
over control to the program. This prevents some Global Offset Table GOT
overwrite attacks, or in the dtors section of the ELF binary.



Domain                            | `compiler` and `linker` options | _State_
--------------------------------- | ------------------------------- | --------
Kernel-General-OverwriteAttacks-1 | `-z,relro`                      | _Enable_
Kernel-General-OverwriteAttacks-2 | `-z,now`                        | _Enable_



During program load, all dynamic symbols are resolved, allowing for the complete
GOT to be marked read-only (due to `-z relro` above). This prevents GOT
overwrite attacks. For very large application, this can incur some performance
loss during initial load while symbols are resolved, but this shouldn't be an
issue for daemons.

--------------------------------------------------------------------------------



## Library linking



Domain                          | Improvement
------------------------------- | ---------------
Kernel-General-LibraryLinking-1 | Keep this part?



It is recommended that dynamic linking should generally not be allowed. This
will avoid the user from replacing a library with malicious library.



Domain                          | Object          | Recommendations
------------------------------- | --------------- | --------------------------------
Kernel-General-LibraryLinking-1 | Dynamic linking | Should generally not be allowed.

Linking everything statically doesn't change anything wrt security as binaries
will live under same user:group as libraries and setuid executables ignore
`LD_PRELOAD/LD_LIBRARY_PATH`. It also increases RSS footprint and creates
problems with upgrading.

# Memory

## Restrict access to kernel memory

The /dev/kmem file in Linux systems is directly mapped to kernel virtual memory.
This can be disastrous if an attacker gains root access, as the attacker would
have direct access to kernel virtual memory.

To disable the /dev/kmem file, which is very infrequently used by applications,
the following kernel option should be set in the compile-time kernel
configuration:

Domain                         | `Config` name    | `Value`
------------------------------ | ---------------- | -------
Kernel-Memory-RestrictAccess-1 | `CONFIG_DEVKMEM` | `n`

In case applications in userspace need /dev/kmem support, it should be available
only for authenticated applications.

--------------------------------------------------------------------------------

## Disable access to a kernel core dump

This kernel configuration disables access to a kernel core dump from user space.
If enabled, it gives attackers a useful view into kernel memory.



Domain                   | `Config` name       | `Value`
------------------------ | ------------------- | -------
Kernel-Memory-CoreDump-1 | `CONFIG_PROC_KCORE` | `n`



--------------------------------------------------------------------------------

## Disable swap

If not disabled, attackers can enable swap at runtime, add pressure to the
memory subsystem and then scour the pages written to swap for useful
information.



Domain               | `Config` name | `Value`
-------------------- | ------------- | -------
Kernel-Memory-Swap-1 | `CONFIG_SWAP` | `n`





- Enabling swap at runtime require `CAP_SYS_ADMIN`.
- Swap block device is usually under root:disk.
- Linux never swaps kernel pages.
- If swap disabling is not possible, swap encryption should be enabled.



--------------------------------------------------------------------------------



## Disable "Load All Symbols"

There is a /proc/kallsyms file which exposes the kernel memory space address of
many kernel symbols (functions, variables, etc...). This information is useful
to attackers in identifying kernel versions/configurations and in preparing
payloads for the exploits of kernel space.

Both `KALLSYMS_ALL` and `KALLSYMS` shall be disabled;



Domain                         | `Config` name         | `Value`
------------------------------ | --------------------- | -------
Kernel-Memory-LoadAllSymbols-1 | `CONFIG_KALLSYMS`     | `n`
Kernel-Memory-LoadAllSymbols-2 | `CONFIG_KALLSYMS_ALL` | `n`



--------------------------------------------------------------------------------

## Stack protection

To prevent stack-smashing, similar to the stack protector used for ELF programs
in user-space, the kernel can protect its internal stacks as well.

This configuration is supported in **Linux 3.11 and greater** and thus should
only be enabled for such versions.

This configuration also requires building the kernel with the **gcc compiler 4.2
or greater**.



Domain                | `Config` name              | `Value`
--------------------- | -------------------------- | -------
Kernel-Memory-Stack-1 | `CONFIG_CC_STACKPROTECTOR` | `y`



Other defenses include things like shadow stacks.

--------------------------------------------------------------------------------

## Disable access to /dev/mem

The /dev/mem file in Linux systems is directly mapped to physical memory. This
can be disastrous if an attacker gains root access, as the attacker would have
direct access to physical memory through this convenient device file. It may not
always be possible to disable such file, as some applications might need such
support. In that case, then this device file should be available only for
authenticated applications.

This configuration is supported in **Linux 4.0 and greater** and thus should
only be disabled for such versions.



Domain                 | `Config` name   | `Value`
---------------------- | --------------- | -------
Kernel-Memory-Access-1 | `CONFIG_DEVMEM` | `n`



--------------------------------------------------------------------------------



## Disable cross-memory attach

Disable the process_vm_*v syscalls which allow one process to peek/poke the
virtual memory of another.

This configuration is supported in **Linux 3.5 and greater** and thus should
only be disabled for such versions.



Domain                         | `Config` name         | `Value`
------------------------------ | --------------------- | -------
Kernel-Memory-CrossMemAttach-1 | `CROSS_MEMORY_ATTACH` | `n`



--------------------------------------------------------------------------------

## Stack Smashing Attacks



Domain                        | `compiler` and `linker` options | _State_
----------------------------- | ------------------------------- | --------
Kernel-Memory-StackSmashing-1 | `-fstack-protector-all`         | _Enable_



Emit extra code to check for buffer overflows, such as stack smashing attacks.

--------------------------------------------------------------------------------

## Detect Buffer Overflows

Domain                          | `compiler` options and `config` name | `Value`
------------------------------- | ------------------------------------ | -------
Kernel-Memory-BufferOverflows-1 | `-D_FORTIFY_SOURCE`                  | `2`
Kernel-Memory-BufferOverflows-2 | `CONFIG_FORTIFY_SOURCE`              | `y`

Helps detect some buffer overflow errors.

# Serial

## Disable serial console

The serial console should be disabled to prevent an attacker from accessing this
powerful interface.

Domain                   | `Config` name                | `Value`
------------------------ | ---------------------------- | -------
Kernel-Consoles-Serial-1 | `CONFIG_SERIAL_8250`         | `n`
Kernel-Consoles-Serial-2 | `CONFIG_SERIAL_8250_CONSOLE` | `n`
Kernel-Consoles-Serial-3 | `CONFIG_SERIAL_CORE`         | `n`
Kernel-Consoles-Serial-4 | `CONFIG_SERIAL_CORE_CONSOLE` | `n`

--------------------------------------------------------------------------------

## Bake-in the kernel command-line

The kernel command-line is used to control many aspects of the booting kernel,
and is prone to tampering as they are passed in RAM with little to no reverse
validation on these parameters. To prevent this type of attack, the kernel shall
be configured to ignore commands line arguments, and use pre-configured (compile
time) options instead.

Set the kernel command line in the `CONFIG_CMDLINE KConfig` item and then pass
no arguments from the bootloader.



Domain                        | `Config` name             | `Value`
----------------------------- | ------------------------- | -----------------------------------
Kernel-Consoles-CommandLine-1 | `CONFIG_CMDLINE_BOOL`     | `y`
Kernel-Consoles-CommandLine-2 | `CONFIG_CMDLINE`          | `"insert kernel command line here"`
Kernel-Consoles-CommandLine-3 | `CONFIG_CMDLINE_OVERRIDE` | `y`



It is recommended that any per-device settings (e.g: MAC addresses, serial
numbers, etc.) be stored and accessed from read-only memory (or files), and that
any such parameters be verified (signature checking) prior to their use.

--------------------------------------------------------------------------------

## Disable KGDB

The Linux kernel supports KGDB over USB and console ports. These mechanisms are
controlled by the `kgdbdbgp` and `kgdboc` kernel command-line parameters. It is
important to ensure that no shipping product contains a kernel with KGDB
compiled-in.



Domain                 | `Config` name | `Value`
---------------------- | ------------- | -------
Kernel-Consoles-KDBG-1 | `CONFIG_KGDB` | `n`



--------------------------------------------------------------------------------

## Disable magic sysrq support

On a few architectures, you can access a powerful debugger interface from the
keyboard. The same powerful interface can be present on the serial console
(responding to serial break) of Linux on other architectures. Disable to avoid
potentially exposing this powerful backdoor.



Domain                  | `Config` name        | `Value`
----------------------- | -------------------- | -------
Kernel-Consoles-SysRQ-1 | `CONFIG_MAGIC_SYSRQ` | `n`



--------------------------------------------------------------------------------

## Disable support for binary formats other than ELF

This will make possible to plug wrapper-driven binary formats into the kernel.
It enables support for binary formats other than ELF. Providing the ability to
use alternate interpreters would assist an attacker in discovering attack
vectors.



Domain                         | `Config` name        | `Value`
------------------------------ | -------------------- | -------
Kernel-Consoles-BinaryFormat-1 | `CONFIG_BINFMT_MISC` | `n`

# Debug

No debuggers shall be present on the file system. This includes, but is not
limited to, the GNU Debugger client/server (commonly known in their short form
names such as the `gdb` and `gdbserver` executable binaries respectively), the
`LLDB` next generation debugger or the `TCF` (Target Communications Framework)
agnostic framework. Including these binaries as part of the file system will
facilitate an attacker's ability to reverse engineer and debug (either locally
or remotely) any process that is currently executing on the device.

## Kernel debug symbols

Debug symbols should always be removed from production kernels as they provide a
lot of information to attackers.



Domain                 | `Config` name       | `Value`
---------------------- | ------------------- | -------
Kernel-Debug-Symbols-1 | `CONFIG_DEBUG_INFO` | `n`



These kernel debug symbols are enabled by other config items in the kernel. Care
should be taken to disable those also. If `CONFIG_DEBUG_INFO` cannot be
disabled, then enabling `CONFIG_DEBUG_INFO_REDUCED` is second best.



At least `CONFIG_DEBUG_INFO_REDUCED` should be always enabled for developers to
convert addresses in oops messages to line numbers.



--------------------------------------------------------------------------------

## Disable Kprobes

Kprobes enables you to dynamically break into any kernel routine and collect
debugging and performance information non-disruptively. You can trap at almost
any kernel code address, specifying a handler routine to be invoked when the
breakpoint is hit.



Domain                 | `Config` name    | `Value`
---------------------- | ---------------- | -------
Kernel-Debug-Kprobes-1 | `CONFIG_KPROBES` | `n`



--------------------------------------------------------------------------------

## Disable Tracing

FTrace enables the kernel to trace every kernel function. Providing kernel trace
functionality would assist an attacker in discovering attack vectors.



Domain                 | `Config` name   | `Value`
---------------------- | --------------- | -------
Kernel-Debug-Tracing-1 | `CONFIG_FTRACE` | `n`



--------------------------------------------------------------------------------

## Disable Profiling

Profiling and OProfile enables profiling the whole system, include the kernel,
kernel modules, libraries, and applications. Providing profiling functionality
would assist an attacker in discovering attack vectors.



Domain                   | `Config` name      | `Value`
------------------------ | ------------------ | -------
Kernel-Debug-Profiling-1 | `CONFIG_OPROFILE`  | `n`
Kernel-Debug-Profiling-2 | `CONFIG_PROFILING` | `n`



--------------------------------------------------------------------------------

## Disable OOPS print on BUG()

The output from OOPS print can be helpful in Return Oriented Programming (ROP)
when trying to determine the effectiveness of an exploit.



Domain                   | `Config` name             | `Value`
------------------------ | ------------------------- | -------
Kernel-Debug-OOPSOnBUG-1 | `CONFIG_DEBUG_BUGVERBOSE` | `n`



--------------------------------------------------------------------------------

## Disable Kernel Debugging

There are development-only branches of code in the kernel enabled by the
`DEBUG_KERNEL` conf. This should be disabled to compile-out these branches.



Domain             | `Config` name         | `Value`
------------------ | --------------------- | -------
Kernel-Debug-Dev-1 | `CONFIG_DEBUG_KERNEL` | `n`
Kernel-Debug-Dev-2 | `CONFIG_EMBEDDED`     | `n`



In some kernel versions, disabling this requires also disabling
`CONFIG_EMBEDDED`, and `CONFIG_EXPERT`. Disabling `CONFIG_EXPERT` makes it
impossible to disable `COREDUMP`, `DEBUG_BUGVERBOSE`, `NAMESPACES`, `KALLSYMS`
and `BUG`. In which case it is better to leave this enabled than enable the
others.

--------------------------------------------------------------------------------



## Disable the kernel debug filesystem

The kernel debug filesystem presents a lot of useful information and means of
manipulation of the kernel to an attacker.



Domain                    | `Config` name     | `Value`
------------------------- | ----------------- | -------
Kernel-Debug-FileSystem-1 | `CONFIG_DEBUG_FS` | `n`



--------------------------------------------------------------------------------

## Disable BUG() support

The kernel will display backtrace and register information for BUGs and WARNs in
kernel space, making it easier for attackers to develop exploits.



Domain             | `Config` name | `Value`
------------------ | ------------- | -------
Kernel-Debug-BUG-1 | `CONFIG_BUG`  | `n`



--------------------------------------------------------------------------------

## Disable core dumps

Core dumps provide a lot of debug information for hackers. So disabling core
dumps are recommended in production builds.

This configuration is supported in **Linux 3.7 and greater** and thus should
only be disabled for such versions.



Domain                   | `Config` name     | `Value`
------------------------ | ----------------- | -------
Kernel-Debug-CoreDumps-1 | `CONFIG_COREDUMP` | `n`



--------------------------------------------------------------------------------



## Kernel Address Display Restriction

When attackers try to develop "run anywhere" exploits for kernel
vulnerabilities, they frequently need to know the location of internal kernel
structures. By treating kernel addresses as sensitive information, those
locations are not visible to regular local users.

**/proc/sys/kernel/kptr_restrict is set to "1"** to block the reporting of known
kernel address leaks.



Domain                       | `File` name                      | `Value`
---------------------------- | -------------------------------- | -------
Kernel-Debug-AdressDisplay-1 | `/proc/sys/kernel/kptr_restrict` | `1`



Additionally, various files and directories should be readable only by the root
user: `/boot/vmlinuz*`, `/boot/System.map*`, `/sys/kernel/debug/`,
`/proc/slabinfo`



Domain                       | `File` or `Directorie` name | _State_
---------------------------- | --------------------------- | -----------------------------
Kernel-Debug-AdressDisplay-1 | `/boot/vmlinuz*`            | _Readable Only for root user_
Kernel-Debug-AdressDisplay-2 | `/boot/System.map*`         | _Readable Only for root user_
Kernel-Debug-AdressDisplay-3 | `/sys/kernel/debug/`        | _Readable Only for root user_
Kernel-Debug-AdressDisplay-4 | `/proc/slabinfo`            | _Readable Only for root user_



--------------------------------------------------------------------------------

## DMESG Restrictions

When attackers try to develop "run anywhere" exploits for vulnerabilities, they
frequently will use `dmesg` output. By treating `dmesg` output as sensitive
information, this output is not available to the attacker.

**/proc/sys/kernel/dmesg_restrict can be set to "1"** to treat dmesg output as
sensitive.



Domain               | `File` name                       | `Value`
-------------------- | --------------------------------- | -------
Kernel-Debug-DMESG-1 | `/proc/sys/kernel/dmesg_restrict` | `1`



Enable the below compiler and linker options when building user-space
applications to avoid stack smashing, buffer overflow attacks.

--------------------------------------------------------------------------------



## Disable /proc/config.gz

It is extremely important to not expose the kernel configuration used on a
production device to a potential attacker. With access to the kernel config, it
could be possible for an attacker to build a custom kernel for the device that
may disable critical security features.



Domain                | `Config` name     | `Value`
--------------------- | ----------------- | -------
Kernel-Debug-Config-1 | `CONFIG_IKCONFIG` | `n`

# File System

## Disable all file systems not needed

To reduce the attack surface, file system data is parsed by the kernel, so any
logic bugs in file system drivers can become kernel exploits.

### Disable NFS file system

NFS FileSystems are useful during development phases, but this can be a very
helpful way for an attacker to get files when you are in production mode, so we
must disable them.



Domain                   | `Config` name   | `Value`
------------------------ | --------------- | -------
Kernel-FileSystems-NFS-1 | `CONFIG_NFSD`   | `n`
Kernel-FileSystems-NFS-2 | `CONFIG_NFS_FS` | `n`



--------------------------------------------------------------------------------



## Partition Mount Options

There are several security restrictions that can be set on a filesystem when it
is mounted. Some common security options include, but are not limited to:

`nosuid` - Do not allow set-user-identifier or set-group-identifier bits to take
effect.

`nodev` - Do not interpret character or block special devices on the filesystem.

`noexec` - Do not allow execution of any binaries on the mounted filesystem.

`ro` - Mount filesystem as read-only.

The following flags shall be used for mounting common filesystems:



Domain                     | `Partition`         | `Value`
-------------------------- | ------------------- | -----------------------------------------------------------------
Kernel-FileSystems-Mount-1 | `/boot`             | `nosuid`, `nodev` and `noexec`.
Kernel-FileSystems-Mount-2 | `/var` & `/tmp`     | In `/etc/fstab` or `vfstab`, add `nosuid`, `nodev` and `noexec`.
Kernel-FileSystems-Mount-3 | _Non-root local_    | If type is `ext2` or `ext3` and mount point not '/', add `nodev`.
Kernel-FileSystems-Mount-4 | _Removable storage_ | Add `nosuid`, `nodev` and `noexec`.
Kernel-FileSystems-Mount-5 | _Temporary storage_ | Add `nosuid`, `nodev` and `noexec`.
Kernel-FileSystems-Mount-6 | `/dev/shm`          | Add `nosuid`, `nodev` and `noexec`.
Kernel-FileSystems-Mount-7 | `/dev`              | Add `nosuid` and `noexec`.



If `CONFIG_DEVTMPFS_MOUNT` is set, then the kernel will mount /dev and will not
apply the `nosuid`, `noexec` options. Either disable `CONFIG_DEVTMPFS_MOUNT` or
add a remount with `noexec` and `nosuid` options to system startup.



Domain                     | `Config` name           | _State_ or `Value`
-------------------------- | ----------------------- | -----------------------------------------------------------------------
Kernel-FileSystems-Mount-1 | `CONFIG_DEVTMPFS_MOUNT` | _Disabled_ or add remount with `noexec` and `nosuid` to system startup.