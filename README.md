# [Website](https://nico-shock.github.io/GPU-Passthrough-to-KVM-on-Nvidia-Arch-/)


# Requirements:

- Installed Chaotic-AUR repositories  
- Already installed Windows on KVM (or any other OS)  
- 2 GPUs (dedicated and iGPU works too)  
- Virtualization and IOMMU enabled in your BIOS/UEFI  
- An HDMI cable connected to your motherboard if you have an iGPU as a second GPU  

## *MAKE SURE TO READ EVERYTHING FIRST BEFORE CONTINUING AND ENSURE YOU UNDERSTAND EVERYTHING SHOWN*

### Install KVM if not already done:
`sudo pacman -S qemu-full virt-manager bridge-utils libguestfs dnsmasq vde2 openbsd-netcat && sudo systemctl enable libvirtd && sudo systemctl start libvirtd && sudo usermod -aG kvm $USER && sudo usermod -aG libvirt $USER && sudo virsh net-start default && sudo virsh net-autostart default`

### Install the VFIO Linux Kernel: 
`sudo pacman -S linux-vfio linux-vfio-headers`

### Get Your GPU IDs: 
`lspci -nn | grep -E "NVIDIA"`

Trim the IDs and separate them with commas (`,`) only.  
Save the IDs somewhere.  
You will need both your GPU ID and your GPU audio device ID.

## Boot from the VFIO Kernel:

### Edit your bootloader configuration for GRUB:
Edit the file: `sudo vim /boot/grub/grub.cfg`

Then add the following to your "GRUB_CMDLINE" configuration: 
`amd_iommu=on iommu=pt vfio-pci.ids=your,ids`

Then change your configuration:
`GRUB_DEFAULT="0"` -> `1>2` (replace with your number)

If you're booting from the GRUB menu, you can go under "Advanced options" and see the different kernels. The first option is always `0`, the second is `1`, and so on. You want to change it to the `linux-vfio` kernel.

To boot from the "Advanced options," use `GRUB_DEFAULT="1>2"` with the extra `>` sign to select the correct kernel (replace `2` with the correct boot option).

For GRUB, run: `sudo grub-mkconfig -o /boot/grub/grub.cfg`

### Edit your bootloader configuration for Systemd:
Edit the file: `sudo vim /boot/loader/entries/arch.conf`

Add after the "rw": 
`amd_iommu=on iommu=pt vfio-pci.ids=your,ids`

Then change the bootloader configuration to (DO NOT REMOVE ANYTHING):
```
linux /vmlinuz-linux-vfio
initrd /initramfs-linux-vfio.img
```

### *You can replace `amd_iommu=on` with `intel_iommu=on` if you have an Intel CPU.*
### *Replace `your,ids` with the IDs you saved earlier.*

### Create and Edit the Modprobe Config File:
```
sudo touch /etc/modprobe.d/vfio.conf
sudo vim /etc/modprobe.d/vfio.conf
```

Paste the following into the file:
```
options vfio-pci ids=your,ids
softdep nvidia pre: vfio-pci
```

Replace `your,ids` with your GPU IDs.

### Update the Initramfs: 
`sudo mkinitcpio -P linux-vfio`

### Reboot Your System: 
`sudo reboot now`

### Load vfio-pci modules
`sudo modprobe vfio-pci`

### Add the GPU to Your VM in KVM:
### *In your VM settings, click "Add" and select your NVIDIA GPU and NVIDIA audio device.*

## **MAKE SURE TO USE THE CORRECT HDMI PORT CONNECTED TO YOUR MOTHERBOARD WHEN YOU PASSTHROUGH YOUR DEDICATED GPU TO A VM IF YOU ONLY HAVE ONE DEDICATED GPU AND AN iGPU YOU CAN USE.**

### **ALSO MAKE SURE THAT YOU INSTALLED NVIDIA DRIVERS, LOOKING GLASS AND SPICE TOOLS.**
