# Lenovo-Legion-5

Dual GPU AMD-NVIDIA Setting on Arch Linux based Distro
Ashar A.

Ashar A.

Mar 20, 2020·4 min read

In the past, I found no succeed method to switch between AMD processor with GPU (APU) such as Ryzen, to NVIDIA GPU in notebook. I have decided to try it myself and I hope this tutorial will help everybody out there.
Disclaimer:

I have tested this steps twice with different Arch Linux based Distros (of course fresh installation, and vanilla-distro Arch Linux) and succeed.
My notebook has AMD Ryzen 3550H CPU with VEGA 8 GPU and NVIDIA GTX 1050. Its brand is ASUS but this tutorial should be applicable on any other brand, also it is safe to use between Intel CPU-GPU and NVIDIA GPU hybrid too.
I am using linux kernel, If You are willing to use linux-lts or linux-zen, the steps are similar.
Follow this tutorial if You know what You are doing and I don’t care If something happened to Your notebook. Do at Your own risk.

My Setup:
Arch Linux with Linux Kernel 5.5.9
GCC 9.3.0
KDE Plasma Desktop
AMDGPU Driver from Kernel
Propietary NVIDIA Driver

    Blacklist the NOUVEAU

The first thing to know, using NVIDIA propietary driver means You can’t use nouveau driver. Do this (text_editor You can use such as gedit, nano, vim, etc):

    sudo (text_editor) /etc/modprobe.d/blacklist.conf

write and save:

    blacklist nouveau

2. Back Up Xorg.conf, and Delete It

This to save the working Xorg config when something wrong happened You can restore it through TTY2 (Ctrl Alt F2).

    sudo cp /etc/X11/xorg.conf ~/

    sudo rm /etc/X11/xorg.conf

3. Install NVIDIA GPU Propietary Driver

I am using DKMS. Make sure You are running the latest Linux Kernel on Arch Linux, and also have linux-headers. You can skip the 3.1 step If You are on specific kernel You have and knowing what position are You in now.

Make sure You have specific gcc version same as kernel. If You aren’t sure, just do following two steps.

    3.1. sudo pacman -Syy && sudo pacman -S linux linux-headers base-devel

    3.2. sudo pacman -S nvidia-dkms

4. Edit AMDGPU and NVIDIA Xorg Settings

Do this command to look what files are in /usr/share/X11/xorg.conf.d/
ls /usr/share/X11/xorg.conf.d/

Some distro might produced different name but have NVIDIA name at one file among many files listed above. For my case, the name is 10-nvidia-drm-outputclass.conf

Edit the 10-amdgpu.conf

    sudo (text_editor) /usr/share/X11/xorg.conf.d/10-amdgpu.conf

Change driver to modesetting and save:

    Section “OutputClass”
    Identifier “AMDgpu”
    MatchDriver “amdgpu”
    Driver “modesetting”
    EndSection

Now edit the NVIDIA config file:

    sudo (text_editor) /usr/share/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf

Add Option “PrimaryGPU” “Yes” to the file and save:

    Section “OutputClass”
    Identifier “nvidia”
    MatchDriver “nvidia-drm”
    Driver “nvidia”
    Option “AllowEmptyInitialConfiguration”
    ModulePath “/usr/lib/nvidia/xorg”
    ModulePath “/usr/lib/xorg/modules”
    Option “PrimaryGPU” “Yes”
    EndSection

5. Editing the Mkinitcpio

To make sure the Kernel knows some driver is installed, You have to edit the mkinitcpio.

    sudo (text_editor) /etc/mkinitcpio.conf

Add nvidia and amdgpu with following sequence and save:

    MODULES=(nvidia amdgpu)

Run mkinitcpio

    sudo mkinitcpio -p linux

6. Run the Amazing Prime Select

This program is intentionally written for Fedora, but it tested on so many distros. Use this to switch between AMD to NVIDIA or Intel to NVIDIA.

    git clone https://github.com/wildtruc/nvidia-prime-select.git
    cd nvidia-prime-select
    sudo make install

Run this to switch to NVIDIA:

    sudo nvidia-prime-select nvidia

And run this whenever want to switch back default VGA (both AMDGPU and Intel):

    sudo nvidia-prime-select intel

Whenever You run one of those switching scripts, You have to reboot after.

7. Reboot

After reboot, You will look this amazing nvidia-smi working like a charm.
The Dual GPU is working nicely

Known Issues (from nvidia-prime-select):
The script has been test on Gnome Shell, Gnome Classic, Cinnamon, LXQT, Kodi (for previous version, lightdm only for new one).

    The only issue comes with Gnome Classic, desktop crash on final start. I’m not sure it comes from Gnome Classic itself.
    For Fedora users upgrading from Fedora 23 to 24 using the dnf tools, don’t forget to re-enable the service after the first reboot. You have to probably reset your display xrandr config too.
    Since Fedora 24, rc.nivia schedule time set is not enough to let GDM fully start. Need to extend from 5 to 10 secondes (update 10/08/16).
    Session restart on gdm (gnome3) may cause result in a blank screen. In previous nvidia-prime-select, this issues was fix by inserting a delay waiting for full gdm start before insert xrandr command line. Try to uncomment ‘sleep’ function in /etc/nvidia-prime/xinitrc.prime and different delay. If it doesn’t fix, think to change session manager to lightdm.
    In some case, xrandr display config (~.config/monitors.xml) could conflict with nvidia-prime-select xrandr auto conf function. First, remove ~.config/monitors.xml, and restart your session. If it doesn’t fix, set your display again and disable nvidia-prime.desktop autostart (menu > system > pref > personal > autostart), then restart your session.
    At session restart login has a strange behaviour and could take 30/40s to display correctly. It maybe a polkit issue, but not sure. Need debug and figure out.
    Do not hesitate to send issue reports on Github page.
