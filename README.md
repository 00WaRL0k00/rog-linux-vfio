# Ensure the following are installed
* asusctl
* cpupower (for setting cpu governor to performance before starting VM)
* vfio-isolate (to isolate your CPU's for the VM)
* looking-glass
* libvirt (and all QEMU things required for typical GPU passthrough)

Optional: scream (only needed if an audio device isn't being passed through)

# Prepare files for extraction
Simply extract the tarball to somewhere that works for you. The open a terminal and `cd` to the folder. Your folder should contain the following:
* Folders: `etc`, `home`, `usr`
* `gamingvm.desktop`
* This readme file

A tar is used instead of repo files so that the executable permissions are already set for you. All you need to do is copy as indicated in the rest of this guide! :)


# Why use libvirt hooks?
Using libvirt hooks allows you to automatically run scripts at different stages of the startup/shutdown phase of your VM. My collection of sripts does the following:
1) Before the VM is started:
    * sets asusctl mode to vfio
    * allocates hugepages
    * sets CPU governor to performance and isolates cares
2) When VM shutdown signal is sent:
    * Kill Scream and Looking Glass
3) After the VM has stopped:
    * deallocates hugepages
    * sets CPU governor back to schedutil and remove CPU core isolation
    * sets asusctl mode back to integrated

# Install VM hooks for libvirt
```
#  mkdir -p /etc/libvirt/hooks
#  wget 'https://raw.githubusercontent.com/PassthroughPOST/VFIO-Tools/master/libvirt_hooks/qemu' -O /etc/libvirt/hooks/qemu
#  chmod +x /etc/libvirt/hooks/qemu
```
From your working folder, copy the contents of the etc folder:
```
$  cp etc/libvirt/hooks /etc/libvirt/hooks
```

The copied folder should contain a `kvm.conf` file, along with a `qemu.d` folder.
1) Edit the kvm.conf values:
    * VM_NAME needs to match the name of your VM (probably `win10` in most cases)
    * MEMORY should equal the amount of RAM allocated for your VM.
        * The preset value is 8388608 Mib (or 8 GiB)
        * The value must be in MiB, the rest of the hook scripts will reference these values.
        * This value is used to allocate hugepages for your VM
2) Rename the `GamingBox` folder inside of `qemu.d` to match your VM name like you did for the kvm.conf file
3) If you are using other methods for isolating CPU's or setting the governor, edit the scripts in the `qemu.d` folder to use your preferred methods

# XML Files and Startup Script
First thing to do here is copy the home folder to yours:
```
$  cp home/kyle/vfio ~/vfio
```

Inside the `vfio` folder you'll find 3 files:
* acpitable.bin
    * This file is the fake battery which you'll need to add in your VM so that the Nvidia Windows driver loads. Refer to the Win10.xml for details
* start_vm.sh
    * This executable script is what you need to run in order to start the VM.
    * Notes about this file:
        * You will need to replace the text `GamingBox` with the name of your VM
        * You will need to replace the text `kyle` with the name of your Linux user
* Win10.xml
    * This is an example of my VM's XML file. You can refer to it and use it to modify your VM's XML

# Set up an easy to use launcher entry
For ease of use, I've also included steps on how you can create an entry in your application launcher to start your VM. You will first need to copy the `gamingvm.desktop` file to your distribution's applications folder:

```
#  cp gamingvm.desktop /usr/share/applications/
```

Once you've copied the file to the applications folder, you'll need to edit this line: `Exec=/home/kyle/vfio/start_vm.sh` . Like in the previous step, you'll need to replace the text `kyle` with the name of your Linux user.

I've also created a little Windows 10 icon for the launcher:
```
$  cp -r home/kyle/.local/share/icons/ ~/.local/share/
```
You should now have an entry in your app launcher with a nice fancy Windows 10 logo

# Readme To-Do List!
* Add sections on how to:
    * Set up Looking Glass and Scream
    * Set up groups so no authentication is needed in order to connect to libvirt/qemu
    * Add evdev devices (This allows you to share your keyboard/mouse with your VM using a hotkey, no Spice server required - very cool stuff)
    * Create your windows gaming VM from start to finish
