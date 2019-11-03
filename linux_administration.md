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

