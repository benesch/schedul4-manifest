# schedul4-manifest

schedul4 is a highly-experimental user-level scheduler for the [seL4
microkernel], developed as a final project for [CS260r] at Harvard
University.

The author admits his familiarity with microkernels, scheduling, and
operating system development is minimal. Everything about this
repository should be regarded as broken until further notice.

[View the project proposal [PDF].][project-proposal]

## Installation

### From scratch

Follow the official seL4 instructions for
[getting started with CAmkEs][camkes-getting-started].

In short, once you have the prerequisites:

```bash
$ mkdir schedul4-project
$ cd schedul4-project
$ repo init --manifest-url=https://github.com/benesch/schedul4-manifest.git
$ repo sync
```

### With Vagrant

A VirtualBox VM image with a working environment is also available. It's
most easily installed with [Vagrant], a command-line wrapper around
VirtualBox. From this directory, run:

```bash
$ vagrant up --no-provision
$ vagrant ssh
# hack away
```

Upon login, `cd` into the project root at `/vagrant/project`. This
directory is automatically synced with `THIS_REPO/project/` on your host
machine, so you can edit files with your text editor of choice, and run
only `make` via SSH.

On OS X, the fastest way to install Vagrant is through [Homebrew's][homebrew]
[Cask plugin][homebrew-cask]:

```bash
$ brew install caskroom/cask/brew-cask
$ brew cask install virtualbox
$ brew cask install vagrant
```

On Windows and Linux, download the official installers for both:

* [VirtualBox]
* [Vagrant]

## Usage

To build an application into a runnable kernel image:

```bash
$ make APPLICATION_platform_defconfig
$ make
```

To build the ipcbench application in the Vagrant box, for example:

```bash
$ make ipcbench_debian_defconfig
$ make
```

A list of applications is available at
[schedul4/docs/README.md][schedul4-readme].

To run a kernel image:

```bash
$ qemu-system-i386 -nographic -m 512 \
    -kernel images/kernel-ia32-pc99 \
    -initrd images/capdl-loader-experimental-image-ia32-pc99
```

To escape from QEMU, press Ctrl-A, X.

## The challenge

Ping and Pong are trying to have a conversation, but Guzzle keeps
stealing their CPU! The scheduler currently likes Ping, Pong, and Guzzle
equally.

Modify this round-robin scheduler in
`apps/capdl-loader-experimental/src/main.c` to minimize Ping and Pong's
interprocess communication (IPC) latency. You'll probably find the
following two system calls helpful:

```c
seL4_TCB_Activate(seL4_CPtr tcb_capability);
seL4_TCB_GetPreemptionCount(seL4_CPtr tcb_capability);
```

`seL4_TCB_Activate` will immediately switch to the thread referred to by
`tcb_capability`. A list of all running threads at system bootup is
contained within the global `threads` array.

`seL4_TCB_GetPreemptionCount` will return the number of times that the
given thread has been forcibly removed from the CPU due to a timer
interrupt. This is a rough measure of how naughty a process is. But
calling this once per process per scheduling request will incur substantial
overhead.

A stellar implementation would minimize the latency of well-behaved
processes like Ping and Pong while allowing naughty processes like
Guzzle to complete eventually.

[camkes-getting-started]: https://sel4.systems/CAmkES/GettingStarted.pml
[cs260r]: http://read-new.seas.harvard.edu/cs260r/2015/w/Main_Page
[homebrew]: http://brew.sh
[homebrew-cask]: https://github.com/caskroom/homebrew-cask
[project-proposal]: http://read-new.seas.harvard.edu/cs260r/2015/images/5/51/Cs260r-proposal-b.pdf
[seL4 microkernel]: https://sel4.systems
[schedul4-readme]: https://github.com/benesch/schedul4/blob/master/docs/README.md
[vagrant]: http://vagrantup.com
[virtualbox]: http://virtualbox.org
