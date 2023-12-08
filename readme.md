<img src="https://upload.wikimedia.org/wikipedia/commons/2/24/Ansible_logo.svg" width="250" height="156" alt="Ansible Logo" />   <img src="https://upload.wikimedia.org/wikipedia/commons/4/46/Hewlett_Packard_Enterprise_logo.svg" width="250" height="156" alt="HPE Logo" />              <img src="https://upload.wikimedia.org/wikipedia/commons/9/9a/Vmware.svg" width="250" height="156" alt="VMware Logo" />

# ESXi Deployment to HPE Synergy servers Ansible Playbook

This playbook deploying VMware ESXi server installation to multiple HPE Synergy servers at the same time, using Customized Installation and HPE ILO Virtual Media.
All steps in playbook are: 
- Creating kickstart file for customized ESXi installation
- Creating custom ESXi install ISO image. Only customization is path to customized kickstart file (USB)
- Creating IMG file with kickstart file in it
- Creating server profile from profile template in HPE Oneview
- Mounting ISO image to HPE ILO Virtual CD and IMG file to HPE ILO Virtual USB
- Booting server from attached ISO Image and Waiting for server installation
- Unmounting IMG file from Virtual USB (ISO Image is unmounted by installer)
- Creating DNS records in FreeIPA DNS server

## Requirements

  1. At first you need Ansible of course: `pip3 install ansible`
  2. Than Ansible Galaxy Collection with requirements: 
     1. `ansible-galaxy collection install hpe.oneview`
     2. `pip3 install -r ~/.ansible/collections/ansible_collections/hpe/oneview/requirements.txt`
     3. `ansible-galaxy collection install community.general`
  3. Python module for HPE ILO: `pip3 install python-hpilo`
  4. ISO image is created using `mkisofs`, so install for your OS.
  5. IMG file with FAT32 Filesystem is created with this little script: https://github.com/Othernet-Project/dir2fat32
     It is based on: 
        - util-linux (fdisk)
        - dosfstoos (mkfs.vfat)
        - mtools (mmd, mcopy)


### Usage
All variables are in vars/vars.yaml file.
All passwords are in vars/creds.yaml file. In production environment i'm using ansible-vault to encrypt this file. 

  1. Download VMware ESXi installation ISO image and unzip it into image folder
  2. Create directory with name of build number under files/ directory
  3. Copy BOOT.CFG file from image/EFI/BOOT/ to files/{{ esxi_build }}/EFI/BOOT/ directory
  4. Edit BOOT.CFG file in the files/{{ esxi_build }}/EFI/BOOT/ directory and change location of kickstart file to USB. Location ks=usb is enough as this is searching for file ks.cfg in root folder of all USB devices
  5. Run ansible playbook: `ansible-playbook deploy.yaml`

### Customization in kickstartfile
- Install ESXi on first available disk
- Set root password
- Create portgroup for management with vmnic0 and set static IP address, netmask, gateway, nameserver and hostname

After first reboot some other things are done: 
- Add vmnic1 to the management portgroup
- Set HPE Smart Update Utility to AutoDeploy mode
- Enable and start SSH
- Enable and start esx shell

In order to work kickstart file properly, secureboot must be disabled. Otherwise whole section  %firstboot will not run.

## Future additions
- Add host to vCenter
- Add host to Distributed switches
- Create vmkernel adapters for vMotion
- Turn on SecureBoot
