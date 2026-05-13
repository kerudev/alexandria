# manjaro/fixes/updates

Although the updates on the Stable branch are tested by the Manjaro team. that
doesn't mean that updates are 100% free of errors. Sometimes, more than 600 or
800 updates are pushed and there may be incompatibilities.

At the time of writing this, I've experienced two major issues with updates:

## 2025-09-26

The Linux kernel got nuked :D

My theory is that this happened because the system rebooted while the kernel
was getting updated by `pamac`, and it got deleted. I had to reinstall it
manually with a boot USB and `mhwd-kernel`.

This was a long time ago and I do not remember specific commands.

## 2026-05-02

Couldn't change main screen's resolution, nor detect the second monitor.

The issue seemed to be related to the Linux kernel getting bumped from `6.18`
to `7.0.3`, as the Nvidia drivers were not aligned.

To fix the issue:

1. `uname -r`: displays the kernel's version. Used it to verify I had `7.0.3`.
2. `sudo pacman -S linux70-headers`: they contain the definitions needed to
    compile NVIDIA drivers. Maybe not needed every time as Manjaro usually
    provides them as update dependencies.
3. `sudo pacman -S nvidia-dkms`: DKMS means `Dynamic Kernel Module Support`.
    This rebuilds kernel modules and avoids manually reinstalling drivers after
    kernel updates.
4. `sudo pacman -S nvidia-utils lib32-nvidia-utils`: this installs:
    - `nvidia-utils`: `OpenGL/Vulkan` libraries.
    - `nvidia-smi`: SMI stands for `System Management Interface`. It manages
      and monitors NVIDIA GPUs.
    - `lib32-*`: 32 bit versions needed by Steam, Wine, etc.
5. `sudo mkinitcpio -P`: rebuilds `initramfs` for all kernels.
6. `dkms status`: shows `DKMS` modules and the kernels they were built for.

After that, I noticed my Bluetooth headphones were not getting used as output
when connected and I had to switch them manually. To fix this:

1. `systemctl --user restart pipewire pipewire-pulse wireplumber`:
    - `pipewire`: multimedia server that handles audio and video streams.
    - `pipewire-pulse`: compatibility layer that allows `PulseAudio`
      applications to run on `PipeWire`.
    - `wireplumber`: `PipeWire` session and policy manager.

## References

<!-- TODO: add more references -->
- [initramfs](https://wiki.archlinux.org/title/Arch_boot_process#initramfs)
