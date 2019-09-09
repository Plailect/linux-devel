# linux-devel

## What

A modern Linux kernel kernel development environment featuring integrated debugging based on vscode and libvirtd.

![scrsht_header](/images/scrsht_header.png)

## Why

Kernel development can be an extremely difficult environment to create an efficient workflow for. If you do not already have a preferred Linux kernel development workflow (based on your favorite editor such as `vim` or `emacs`), these instructions are for you. If you already have a preferred workflow based on another tool, you may still find value in the libvirtd-related content presented here.

Configuring vscode to use libvirtd (using UEFI direct kernel boot) allows for building the kernel as one would build any other vscode project, and simple debugging by pressing `F5` as one would debug any other C/C++ vscode project. Stack frames, threads, variable introspection, and breakpoints are all presented to the user with an easy-to-use GUI. Hardware breakpoints are used by default as the debugging infrastructure uses GDB under the hood.

## TODO

- Add initramfs instructions
- Debug edge-cases where breakpoints fail

## How

1. Create a root filesystem VM image and place it somewhere owned by your user (any location / distribution will do, and more efficient solutions than this one likely exist)

```bash
cd ~/.local/share/gnome-boxes/images/
virt-builder ubuntu-18.04 --root-password password:toor -o ub1804.qcow2 --format qcow2 --update
```

2. Clone the linux source tree and place it somewhere

```bash
cd ~/documents/projects/
git clone --depth=1 git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
```

3. Configure the kernel source using your favorite method. An example .config is provided which will work with the above virt-builder image, but the most important things which need to be enabled are the virtio block device / network driver, and EFISTUB output

```bash
cd ~/documents/projects/linux/
wget https://raw.githubusercontent.com/Plailect/linux-devel/master/resources/.config
```

4. (Optional) Setup [ccache](https://ccache.dev/)

5. Create a libvirtd session VM (session VMs are owned by your user, system VMs are owned by root; see [this post](https://blog.wikichoon.com/2016/01/qemusystem-vs-qemusession.html) for details) which uses UEFI direct kernel boot pointed at the built kernel bzImage. An example configuration is presented here:

```
cd /tmp/
wget https://raw.githubusercontent.com/Plailect/linux-devel/master/resources/example.xml

# edit as necessary
# may need to install UEFI libvirt OVMF support
vim example.xml

env LIBVIRT_DEFAULT_URI=qemu:///session virsh define /tmp/example.xml
```

6. (Optional) Ensure that libvirtd has a system network you can bridge to, then set the session VM to use the virbr0 bridged adapter for networking (`virt-manager` has an excellent GUI that can do this)

```bash
sudo virsh net-define /usr/share/libvirt/networks/default.xml
sudo virsh net-autostart default
sudo virsh net-start default
```

7. Ensure you have the vscode C/C++ extension, then create the vscode configuration in your kernel source tree

```
cd ~/documents/projects/linux/
mkdir .vscode
wget https://raw.githubusercontent.com/Plailect/linux-devel/master/resources/.vscode/c_cpp_properties.json

wget https://raw.githubusercontent.com/Plailect/linux-devel/master/resources/.vscode/generate_compdb.py # from amezin/vscode-linux-kernel
python .vscode/generate_compdb.py

wget https://raw.githubusercontent.com/Plailect/linux-devel/master/resources/.vscode/launch.json
wget https://raw.githubusercontent.com/Plailect/linux-devel/master/resources/.vscode/tasks.json
```

8. Open vscode to your kernel development directory to test that everything works properly

- `ctrl+shift+b` should trigger a `make` of the kernel
- `F5` should boot the VM, wait a moment for QEMU's gdbserver to connect, then begin displaying the console in a pane
- At any point, pressing `F6` should pause and allow introspection of state in the debugger tab
