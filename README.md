# proxmox-cloudinit tutorial
Commands to create a cloud-init template for Proxmox 8.2.5

1. Prerequisite:
   Download the ISO using the GUI   

# I used the KVM image of Ubuntu 22.04 lts: https://cloud-images.ubuntu.com/daily/server/jammy/current/jammy-server-cloudimg-amd64-disk-kvm.img or you can use any of those images: https://cloud-init.io/
  https://cloud-images.ubuntu.com/daily/server/jammy/current/jammy-server-cloudimg-amd64-disk-kvm.img

2. Create the VM for cloud-init template
   Go to proxmox server and select your node -> Shell and run the first command to create a new VM that will be used as a template and give it some resources but **not below this minimum**:

# **STEP 1**

- set the number as you wish, (I picked "1000") and this number will be the ID number of the cloud-init template, next argument will add memory resources, the next argument will add a number of cpu cores, then add an argument to be able to give a name for the template, and at the end it must be specified the network interface and by default the network interface in proxmox is "net0 virtio" but don't forget to set it as a "bridge mode"  

  $ qm create 1000 --memory 2048 --core 2 --name ubuntu-cloud --net0 virtio,bridge=vmbr0
# after the command has been executed, a new VM will be displayed under your proxmox node

# **STEP 2**

- navigate in the node shell where you earlier executed the commands, and type the command to navigate into the directory where all ISOs are downloaded and stored.

  $ cd /var/lib/vz/template/iso/
# now you can check with "ls -la" command to ensure your ISO is there.

# **STEP 3** 

- the ISO downloaded earlier in "Prerequisite" must be mounted/imported as a disk to our VM created in "Step 1". It must be mounted from the node's Shell with the next command:
   
  $ qm importdisk 1000 jammy-server-cloudimg-amd64-disk-kvm.img local-lvm                                              
 # 
 as a Storage I used the default "local-lvm", if you have another storage such as a NAS, you must replace the "local-lvm" with your storage.

 # **STEP 4** - in this step, we set on our VM "1000" a disk that will be stored on local-lvm (or if you have a NAS, you can add your storage here). This step it is like plugging a harddrive into a motherboard.

  $ qm set 1000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-1000-disk-0

 # **STEP 5** - in this step, we set on our VM "1000" "ide2" the cloud-init drive to our storage. This step is like attaching a CD/DVD drive with a disk inserted.
   
  $ qm set 1000 --ide2 local-lvm:cloudinit
   
 # **STEP 6** - in this step, we set on our VM "1000" in the boot options to use the scsi0 drive
 
  $ qm set 1000 --boot c --bootdisk scsi0

 # **STEP 7** - the final step, in this step we set the serial port on the VM to have the possibility to login via proxmox VNC and interact with the VM(the equivalent of plugging in a VGA monitor to our VM) in case of any network failure.
   
  $ qm set 1000 --serial0 socket --vga serial0

**ALL DONE WITH THE CLI !!!**
-----------------------------

The NEXT STEPS must be completed under the Proxmox GUI:

# STEP 1
select the new "1000" VM under you proxmox node then go to the
- "Hardware" section.
  Memory: you can modify memory size
  Processors: here I set CPU type to "host" to have all capabilities of the host processor
  Hard Disk: SSD emulation (if your proxmox node uses an ssd drive) and at DISK ACTION: increase the disk capacity.
- "Cloud-Init" section
  User: add username
  Password: add password
  SSH Public key: add proxmox public key to be able to connect via SSH
  IP Config: set it to DHCP (to give random IP to all VM clones)

# STEP 2
Right-click on the VM with ID "1000" and click on "Convert to template" (after this step, you can not make any changes)

# STEP 3
Now you can Right-click to the VM "1000" template and click "Clone" button to clone a new VM
in this step you can chose any available VM ID number, name, Target node and the mode of your clone, Linked Clone/ Full Clone.

** Good luck! **


   
  
