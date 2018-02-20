RHCSA/RHCE Automated Homelab Configurator - VM Edition
======================================================

### PURPOSE:
This is a linux homelab configurator based on lab scenarios in seventh edition of Michael Jang's RHCSA/RHCE book. This project is a fork of the [original jang_homelab](https://github.com/bashtheshell/jang_homelab) project, intended for VM rather than direct bare metal. With this configurator, you'd be able to build the entire lab in pure bash within a matter of approximately two hours. While there are obviously faster deployment solutions publicly available, this is intended to be used as a learning lab as one would be able to see how automation work in pieces using only bash and works with exercises in Jang's book for RHCSA/RHCE. No configuration management tool is used in this configurator. 


### DISCLAIMER:
*This homelab configurator and/or this repository is not affiliated, endorsed, or sponsored by Michael Jang, Alessandro Orsaria, or McGraw-Hill Global Education Holdings, LLC, McGraw-Hill School Education Holdings, LLC, or one of their affiliated companies ("MHE" or "us" or "we").*


### WARNING: PLEASE READ BEFORE CONTINUING
**YOU MAY LOSE ALL YOUR DATA IF YOU ATTEMPT TO USE THIS CONFIGURATOR WITHOUT READING THE PREREQUISITES AND EXPECTATIONS BELOW. ALTHOUGH A FRESH VM IS HIGHLY RECOMMENDED.**


### PREREQUISITIES:
This configurator assumes the following configurations are true. Modification to the [custom-variables-fb.var](./preboot/custom-variables-fb.var) may be needed to suit your environment.

- This will be installed on a VM that supports nested hypervisor.
- CentOS 7.4 HTTP install tree (for physical machine) is located at a local IP address:  `172.16.1.250/24`
- The private key (for private repository on Bitbucket.org) is located at `http://172.16.1.250/kickstart/bitbucket_key`
- The CentOS 7.4 ISO image is also located on the same server at `http://172.16.1.250/centos_image/CentOS-7-x86_64-DVD-1708.iso`
- All files in this repository are in a Bitbucket private repository
- Your public SSH key is added to [*user.authorized_keys*](./user.authorized_keys) file
- Primary hard drive is on `/dev/sda` with at least 50 GiB of space. Thin-provisioning is acceptable. Use at least 100 GiB to be safe.  
- Second hard-drive is on `/dev/sdb` with at least 75 GiB of space. Thin-provisioning is acceptable. Use at least 100 GiB to be safe.  
- You would need to supply your own `/etc/ssmtp/ssmtp.conf` file to enable IP address change notification. You can follow the [Arch's wiki here](https://wiki.archlinux.org/index.php/SSMTP) for guidance. You only need to supply the configuration file to receive IP address change notification to your email as [*ssmtp-fb.sh*](./ssmtp-fb.sh) and [*check_ip.sh*](./check_ip.sh) scripts have already done most of the heavy lifting for you. This can be completed later after the entire lab configuration is configured. Also, you'd need to add your email to the [*custom-variables-fb.var*](./preboot/custom-variables-fb.var) file to receive the IP change notification.


### WHAT TO EXPECT:
- This configurator will erase all existing partitions on `/dev/sda` and `/dev/sdb`. Two fresh disks allocated to the VM is preferred.
- The VMs are vanilla install using only **Server with GUI** package group. Nothing more or less. You may begin the lab exercises starting at Chapter 3 in Jang's book.
- In my environment, this configurator takes about 2 hours to complete. Your mileage may vary.


### SYSTEM CONFIGURATION:
**Operating system:** CentOS 7.4  
**Hostname:** hostname.domain.net  
**IP address:** 172.16.1.50/24 (static)  
**Gateway:** 172.16.1.1  
**Nameserver:** 172.16.1.1  
**Administrative user:** custom_user  
**/boot partition**: mount on `/dev/sda1` (size 1024 MiB)  
**LVM use?:** Volume Group **mainVG** on `/dev/sda2` (use entire remaining disk space)  
**/home:** mount on `/dev/mainVG/home` (size 30% of VG size)  
**/ (root):** mount on `/dev/mainVG/root` (all remaining space after /home)  
**swap space:** mount on `/dev/mainVG/swap` (size 2 GiB)


### QUICK START:
Before you can begin, do the following in order:

- You need to add your SSH public key to [user.authorized_keys](./user.authorized_keys). 
- Examine the [preboot-configurator.sh](./preboot-configurator.sh) script to make few changes to the variables at the top. 
- Add the`preboot-configurator.sh` script to the local web server so that your PXE server can point to the `physicalhost-ks.cfg` kickstart file. The `preboot-configurator.sh` would download the [custom-variables-fb.var](./preboot/custom-variables-fb.var), [physicalhost-ks.cfg](./preboot/physicalhost-ks.cfg), and [kickstart-modifier.sh](./preboot/kickstart-modifier.sh) files to the `preboot/` directory. It also optionally run the `kickstart-modifier.sh` script to modify the  `physicalhost-ks.cfg` for you with the customization made in `custom-variables-fb.var` file. 
- Please review the `preboot-configurator.sh`, `physicalhost-ks.cfg`, and `custom-variables-fb.var` files before executing the scripts.
- After making changes to `custom-variables-fb.var` and `preboot-configurator.sh`, run the `preboot-configurator.sh` script.
- Now you can change the boot sequence to boot from PXE if it is not done for you already and then boot your VM.
- PRO TIP: You can configure your PXE boot server to automatically serve the appropriate boot menu for a specific VM using the VM's MAC address. You'd also need to pass the `ifname` kernel option to complete the automation. [Look here](http://www.syslinux.org/wiki/index.php?title=PXELINUX#Configuration) for more information.


### SECURITY CONCERN: 
The security implication of this homelab configurator is by no means very secured. If you are uncomfortable having your private key sitting on a local web server, then feel free make customization to the [*physicalhost-ks.cfg*](./preboot/physicalhost-ks.cfg) kickstart file in **%post** script section. Instead, you can actually have the entire install tree and private key on a USB flash drive. However, such instruction will not be provided here. Also, two files, [custom-variables-fb.var](./preboot/custom-variables-fb.var) and [physicalhost-ks.cfg](./preboot/physicalhost-ks.cfg), would contain sensitive information such as hashed passwords.  


### FILES OVERVIEW: 
To understand how this configurator comes together in pieces, inspect each file in the following order:

*[user.authorized_keys](./user.authorized_keys)*
  > Place your SSH public keys here so you can remote in your physical host once it's configured

*[preboot-configurator.sh](./preboot-configurator.sh)*
  > Script used on local web server to prep it to kickstart the physical host for fresh install

*[kickstart-modifier.sh](./preboot/kickstart-modifier.sh)*
  > Replacing all uppercase keywords in 'physicalhost-ks.cfg' file with actual values from 'custom-variables-fb.var'

*[custom-variables-fb.var](./preboot/custom-variables-fb.var)*
  > File containing all variables for kickstart file and script files in main-script-fb.sh

*[physicalhost-ks.cfg](./preboot/physicalhost-ks.cfg)*
  > Kickstart file for physical host to set up KVM hypervisor/libvirt to host VMs

*[firstboot-fb.service](./firstboot-fb.service)*
  > Systemd Unit file to execute the main-script-fb.sh during first-boot before users can log in 

*[main-script-fb.sh](./main-script-fb.sh)*
  > Main script executes each script below in sequential order
  
*[sysctl-fb.sh](./sysctl-fb.sh)*
  > Configure kernel runtime parameter
  
*[tcpwrapper-fb.sh](tcpwrapper-fb.sh)*
  > To allow or deny services from host
  
*[firewall-fb.sh](./firewall-fb.sh)*
  > Configure firewalld rules using firewall-cmd
  
*[sshd_config_modifier-fb.sh](./sshd_config_modifier-fb.sh)*
  > Modify the `/etc/ssh/sshd_config` file for SSH server
  
*[add_sshd-fb.txt](./add_sshd-fb.txt)*
  > Wishlist file for above script to add/modify sshd_config keywords
  
*[ssh-selinux-port-fb.sh](./ssh-selinux-port-fb.sh)*
  > Change the default **SELinux** port context for SSH port 
  
*[fail2ban-fb.sh](./fail2ban-fb.sh)*
  > Set up **fail2ban** for SSH
  
*[restart-firewall-sshd-fb.sh](./restart-firewall-sshd-fb.sh)*
  > Restart **SSH**, **firewalld**, and **fail2ban** services to put new changes in effect
  
*[rebuilt4next_reboot-fb.sh](./rebuilt4next_reboot-fb.sh)*
  > Rebuild the initial RAM disk to ensure system is bootable
  
*[ssmtp-fb.sh](./ssmtp-fb.sh)*
  > Set up **SSMTP** mail transfer agent (**MTA**) to receive email notification of public IP address change
  
*[check_ip.sh](./check_ip.sh)*
  > The above script depends on this script to check IP address change.
  
*[http_ftp_install-tree-fb.sh](./http_ftp_install-tree-fb.sh)*
  > Set up **HTTP** and **FTP** install trees on the physical server for VMs
  
*[add-hosts-fb.sh](./add-hosts-fb.sh)*
  > Resolve physical machine's and VMs' hostnames in `/etc/hosts`
  
*[libvirt-prep-fb.sh](./libvirt-prep-fb.sh)*
  > Prepare networks and storage pools for VMs
  
*[kscreator-fb.sh](./kscreator-fb.sh)*
  > Create kickstart file uniquely for each VM
  
*[vm-kstemplate-fb.cfg](./vm-kstemplate-fb.cfg)*
  > Kickstart template files for above script
  
*[build-vms-fb.sh](./build-vms-fb.sh)*
  > Set up VMs and install guest OSes
  
*[qemu-hooks-fb.sh](./qemu-hooks-fb.sh)*
  > Allow bidirectional communication between VMs behind NAT-T networks

*[rebuild-vms-fb.sh](./rebuild-vms-fb.sh)*
  > Optional wrapper script to conveniently rebuild the guest VMs typically use for post-installation activities.
 
The security configurations in the following scripts and configuration files are by no means exhaustive and is not recommended for use outside of personal homelab:

- *[sysctl-fb.sh](./sysctl-fb.sh)* 
- *[tcpwrapper-fb.sh](tcpwrapper-fb.sh)* 
- *[add_sshd-fb.txt](./add_sshd-fb.txt)*
- *[fail2ban-fb.sh](./fail2ban-fb.sh)*
