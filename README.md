### Bootstrapping

Kernel loaded into memory and begins execution.
When computer turned on, first executes boot code that is stored in ROM => boot code figures out how to load and start kernel => kernel probes system’s HW and spawns init process (pid 1).

Shell scripts (**init scripts**) manage tasks such as checking and mounting fs (using `fsck`) and start system daemons.


Recovery boot to a shell
mode (also called **single-user/maintenance**) that runs basics in case anything prevents system boot. Need physical access to system (network not allowed in this mode).


Boot process steps:
1) Reading of boot loader from **master boot record (MBR)**
2) Loading and initialization of the kernel

    Kernel initialization:

	kernel is a program and first task is to load this program into 	memory. Path is usually:
	
    `/unix || /vmunix || /boot/vmlinuz`

	2 stage loading process:
	1) system ROM loads small boot program (called **boot loader**) to memory from 	disk. 
	2) kernel probes system to learn how much RAM available 	and is then reserved for the kernel.

3) Device detection and configuration
4) Creation of kernel processes
5) Administrator intervention (single-user mode only)
6) Execution of init scripts.


### PC boot
Initial boot code is called **BIOS (Basic IO System)**.
BIOS is configured to boot from a device (USB/disk/DVD/network).

After choosing the boot device, BIOS tries reading the first block of the device (512 byte segment known as master boot record MBR). MBR contains a program that tells the computer from which partition to load the boot loader. This boot loader is responsible for loading the kernel.

### GRUB: Grand Unified Boot Loader
Default boot loader for most UNIX/Linux systems with Intel processors. GRUB’s job is to choose a kernel from list and load that kernel. Two versions exist, GRUB Legacy (using this one) and GRUB 2.

GRUB reads default boot config from `/boot/grub/menu.lst` (ubuntu) or `/boot/grub/grub.conf`. Reads config during startup time.

Sample `grub.conf`:
```
default=0
timeout=10
splashimage=(hd0,0)/boot/grub/splash.xpm.gz
title Red Hat Enterprise Linux Server
	root (hd0,0)
	kernel /vmlinuz-26.18.92.1.10.el5 ro 	
    root=LABEL=/
```
`(hd0,0)` means first partition of first (as defined by BIOS) HD.


### Multibooting

```
default=0
timeout=5
splashimage=(hd0,2)/boot/grub/splash.xpm.gz
hiddenmenu
title Windows XP
	rootnoverify (hd0,0)
	chainloader + 1
title Red Hat
	root (hd0,1)
	kernel /vmlinuz
```

`chainloader` loads the boot loader from sector 1 on the first partition of the primary IDE drive. `rootnoverify` guarantees that GRUB will not try to mount the specified partition,


### Init Scripts
Scripts run in order and are kept in `/etc/init.d` with links to `/etc/rc0.d`, `/etc/rc1.d` ... `/etc/rcn.d`. 

Scripts do the following but not limited to:
* Setting the name of the computer
* Setting the time zone
* Checking disks with fsck
* Mounting the system’s disks
* Removing old files from `/tmp` directory
* Configuring network interfaces
* Starting up daemons and network services.

These should not be modified.

Ubuntu uses an `init` replacement known as **Upstart** and is an event driven service management system. It uses `/etc/event.d` instead of `inittab` to manage the sequential running of jobs:

```
ls /etc/event.d
control-alt-delete last-good-boot logd rc0 rc1 ... rc6 rc-default rcS rcS-sulogin tty1 ... tty6
```

`init` is the first process that runs after the system boots and is therefore the ancestor to all user processes and most system processes.

`init` has at least 7 run levels:
* Level 0, system is completely shut down.
* Level 1 and S represent single-user mode.
* Levels 2-5 include support for networking.
* Level 6 is a 'reboot' level.

Default run level is 2 or 3 but most Linux distros boot to run level 5 by default.

`/etc/inittab` file tells `init` what to do at each level. Default run level can also be found here.

`telinit` command changes `init`'s run level once system is up. `telinit 3` forces `init` to go to run level 3. `telinit -q` causes `init` to reread `/etc/inittab`.

### Shutting down the system

`shutdown` command is safest, most considerate and thorough.
Running `shutdown` sends warnings to other logged-in users, i.e.

```
sudo shutdown -h 09:30 "Going down for scheduled maintenance. Downtown ~ 1 hour"
```

Or to specify timer before shutdown:
```
sudo shutdown -h +15 "Going down for emergency repair"
```

`halt` logs shutdown, kills nonessential procsses, executes `sync` system call and waits for filesystem writes to complete then halts the kernel.

`reboot` is identical to halt but causes machine to reboot instead of halting.

### Access Control and Rootly Powers

#### File ownership
Files can have both user and group ownership. To check a file ownership, we can run `ls -l $filename`:

```bash
kobbigal⚡️$ ls -l README.md
-rw-r--r--@ 1 kobbigal  staff  4730 Nov  6 17:30 README.md
```

UIDs are mapped in `/etc/passwd` while GIDs are mapped in `/etc/group`. `/etc/shadow` stores the passwords.

#### Process ownership

Process owner can send process signals and reduce the process' scheduling priority.
Processes have a few different identities associated to them:
* Real, effective, saved UID
* Real, effective, saved GID
* Filesystem UID that is used to determine file access permissions.

#### root

UID 0.
Can perform any valid operation on any file or process.

#### Role-based access control (RBAC)

In modern UNIX systems, users are assigned to roles that have certain defined permissions.

#### `su` - subtitute user identity

The `su` command, if run without arguments, will create a shell run as root. You can also log in as another user (if you have their password) by running `su - $username`. The dash indicates that the shell will be run in login mode. `su` is located in `/usr/bin/su` or `/bin/su`.

#### `sudo` - limited `su`

Appending `sudo` to any command will attempt to run the command as the superuser. `sudo` consults the file `/etc/sudoers` lists the users who are authorized to use the `sudo` command and the list of commands they're allowed to run on the host. After inserting the correct `sudo` password, a 5-minute timer will open where the user will not need to insert a password.

`sudo` keeps a log of the commands that were executed. The following information is saved:

* Commands executed.
* The hosts which the command were run on.
* The directory from which they were run.
* Time which the command was run.

Use `syslog` to check this information.

To modify `/etc/sudoers`, you use `visudo` command.


### Processes

Processes in UNIX consist of:

1) Address space (set of memory pages - units in which memory is managed.) It contains among other things:

	* Code and libraries that the process is executing.
	* Variables.
	* Stack.

2) A set of data structures within the kernel. These structures record:

	* Prcoess address space map.
	* Current status of process (sleeping, stopped, etc.)
	* Execution priority of process.
	* Information about the resources the process has used.
	* Information about the files and network ports the process has opened.
	* Process' signal mask (a record of which signals are blocked.)
	* Owner of the process.

#### Process Attributes

* **PID** - Kernel assigns a unique ID to every process. In newer versions of the kernel (2.6.24), processes with the same PID can run concurrently in different namespaces (to support containerization).

* **PPID** (Parent PID) - An existing process clones itself to create a new process. The clone can then exchange the program it's running for a different one. 
When a process is cloned, the original process is referred to as the parent. PPID of a process is the PID of the parent from which it was cloned. If the parent dies, `init` (PID 1) becomes the parent.

* **UUID** and **EUID** - Process' UID is the copy of the UID of the parent process. The EUID is the 'effective' user ID which is an extra UID used to determine what resources and files a process has permission to access. For most processes, UUID = EUID.

* **GID** and **EGID** - Process' GID is the group identification number of the process. The only time which GID is relevant is when a process creates new files where new files may adopt the GID of the creating process.

* **Niceness** - How 'nice' you are to the other users of the system. Range is -20 to +19. High value means you're very nice to other users of the system therefore you receive lower scheduling priority. To set the niceness value:

`nice -n 5 ~/bin/longtask` - lowers priority by 5.
`renice -5 8829` - sets nice value to -5

`nice` is **not** built into `bash`.

* **Control Terminal** - Control terminal determines default linkages for the `stdin`, `stdout` and `stderr`.

#### Lifecycle of a Process

To create a new process, a process copies itself with the `fork` system call. `fork` creates a copy of the original process (almost identical). The new process has a distinct PID. `fork` returns 2 different values. The child returns 0 and the parent receives the PID of the newly-created child. Each copy of the process needs to examine the return value to verify their role.

After the `fork`, the child process will use one of the `exec` commands to execute the new program.

As we discussed, `init` is the parent process with PID 1 which installed automatically by the kernel when the system boots. But it also plays a role in process management.
When a process completes execution, it calls `_exit` to notify the kernel it is ready to die and supplies an exit code. The kernel requires that its death be acknowledged by the process' parent which the parent does with a call to `wait` - wait for the process to change state. The parent receives a copy of the child's exit code, the summary of the process resource consumption. If the parent dies before the child, no call to `wait` is necessary and the orphaned child's parent becomes `init`.

![](http://3.bp.blogspot.com/-DHWNQNvLWsc/U_1RD60lDBI/AAAAAAAAAhA/1YaNoo5re8Q/s1600/process-life-cycle.gif)


#### Signals

Signals are process-level interrupt requests. Signals can be sent to processes, between processes and from different origins (kernel, user, terminal).

When a signal is received, either the process receiving the signal has a specific handler ('catching' the signal) for a specific signal or the kernel takes a default action on behalf of the process.

For a list of different signals, see `/usr/include/signal.h` or `man signal`.


#### Process States

* Runnable - the process can be executed whenever CPU time is available. It has all the resources it needs and just waiting for the CPU to process its data.
* Sleeping - The process is waiting for some resource. When a process is awaiting a system call that cannot be executed immediately, the process state changes to this status.
* Zombie   - The process is trying to die.
* Stopped - The process is suspended (not allowed to execute)

#### Monitor Processes

`ps` is a tool that returns some useful information:

* PID
* UID
* Control terminal of processes
* How much memory a process is using
* How much CPU time it consumed
* Current status (zombies show up as `exiting` or `defunct`)

Popular way to run `ps aux` or `ps lax`.
Commands with brackets (`[kjournald]`) are kernel threads scheduled as processes.

`top|htop` is a tool that provides regularly updated summary of active processes and resources.

To get the full memory consumption (both virtual and resident set) of a process, when `top` is running, press `f` and choose the `DATA` column.


#### `/proc` Filesystem

`ps|top` read the process status information from the `/proc` directory. It includes information that is not available when using `vmstat|top`. For more information run `man proc`.


#### Trace Signals and System Calls

We can attach to a running process to see its output by using `strace`. It will show the process system calls and the return code for each. We can use the `-f` flag to look for forked processes and `-e file` to display only file operations.


#### Runaway Processes

Processes that consume too many resources and usually cause performance issues.
Runaway processes that produce output can fill up an entire filesystem. To check for this, run `df -k` and check for filesystems with 100% utilization. Then use the `du` command on that filesystem to find which directory is using the most space. `fuser` and `lsof` commands can also be useful to debug.


### Filesystem

In UNIX, everything is a file. Examples include processes, audio devices, kernel data structures and more.

The filesystem is comprised by:
* A namespace - a way to name things and organize them in a hierarchy.
* An API - set of system calls for navigating/manipulating objects.
* A security model - a scheme for protecting, hiding, and sharing things.
* An implementation - software to bind the logical model to the hardware.

There are many different filesystem implementations. Some are disk-based, some network based (e.g. NFS), some are foreign filesystems from other OS (NTFS, FAT for Windows).

#### Paths

`/` is the root directory. 

Pathnames can be absolute (`/tmp/foo`) or relative (`some_path/some_path2`). Relative paths are interpreted starting at the current directory (run `pwd` to check the current directory.)