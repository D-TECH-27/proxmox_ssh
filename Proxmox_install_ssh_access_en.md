To install a Proxmox server, you must have : 

  - a physical machine for the server connected to the network (don't forget to format the physical disk on this machine)
  - a formated USB drive
  - another machine (with a formating software (balenaEtcher or Rufus) and a wired connection)
  - an ISO image file of Proxmox Virtual Environment

Step 1 : Collect the ISO file of Proxmox VE via the official site : https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso

In this case, we can download the latest version (9.2).

Step 2 : Insert the formated USB drive and open the Rufus or the balenaEtcher software. You can dowload the software on the official sites :

  - balenaEtcher : https://etcher.balena.io/#download-etcher
  - Rufus : https://rufus.ie/en/#download

Step 3 : 

- On balenaEtcher : 

  - Select "Flash From File" and select the ISO image file :
<img width="799" height="512" alt="image" src="https://github.com/user-attachments/assets/6403afaa-7c79-4540-b023-e40ef993f524" />
 
  - Then, click on "Select target" to chose your USB drive
<img width="792" height="509" alt="image" src="https://github.com/user-attachments/assets/5cf1d86e-7fd6-4bb4-9d04-2bc31b0b667f" />

  - Finally, click on "Flash" and wait for it!
<img width="794" height="502" alt="image" src="https://github.com/user-attachments/assets/6ab79d2e-ec37-4efa-b1c4-19716e1fb85f" />

- On Rufus : 

 - Your USB drive has selected automatically. If not, select your USB drive on the "Device" field. Then click "SELECT" (not the arrow) to choose your ISO image file (If you can't find the file, open your File Explorer and drag and drop your file. A message appears but you can ignore it by clicking "OK"). Click "START" and wait for it! : 
<img width="467" height="539" alt="image" src="https://github.com/user-attachments/assets/1273f663-9b38-40e3-af17-15c5ffbf955e" />

Don't forget to eject the drive when it's finished!

Step 4 : On the other machine (the Proxmox server), insert the USB drive, start it and press the button below (according to your computer brand) while the machine is booting to access to the BIOS Setup : 

- ASUS : F2
- HP : F10, F2, Esc or F6
- DELL : F2
- Acer : F2
- Gigabyte : Del
- ASRock : F2
- Lenovo : F2

Search the boot order menu and change the boot priority by selecting your USB drive and sliding to the first boot drive. Save and exit!

Step 5 : Select "Proxmox VE (Graphical)" and press Enter

The installation enables to : 

- Select the disk to install Proxmox (on the SSD or HDD, not on the USB drive, please!)
- Select the language and the locale
- Insert a "root" password (root is the default user who has all permissions on the system)
- Select a network controller and insert IP address, subnet mask and gateway (static)

The system will reboot after installing. When it's rebooting, eject the USB drive

Step 6 : Go to the first machine and open a web browser and insert : 

      http://[Proxmox IP address]:8006

If you have access to your Proxmox, login on root!

Step 7 : On the Proxmox server, insert this command to install those tools : 

- vim : Text editor
- htop : Hardware info
- net-tools : Network info
- git : Github package import
- nmap : Packet analyzer
- wget : Package import

... and more

      apt install -y vim htop iotop iftop curl wget git net-tools dnsutils lsof tree tcpdump nmap iperf3 mtr smartmontools nvme-cli unattended-upgrades chrony -y

If those packages are not available, you can remove them and press Enter. It is just some tools that cannot impact your configuration.

Step 8 : SSH

To generate a SSH key (in this case, with ed25519 encryption) on Linux, insert this command : 

      ssh-keygen -o -a 256 -t ed25519 -C "$(hostname)-$(date +'%d-%m-%Y')"

Press Enter 3 times (no password to set).

Step 9 : Give all access to the SSH key to "root"

      chmod 700 /root/.ssh
      chmod 400 /root/.ssh/id_ed25519
      chown root:root /root/.ssh/id_ed25519*

Step 10 : On your first machine, open a Command Prompt and insert this command : 

- On Windows : ssh-keygen
- On Linux : go to Step 8
- On MacOS : ssh-keygen -t ed25519

Step 11 : Import the key to the Proxmox server. In this case, I use Windows PowerShell

      type "%USERPROFILE%\.ssh\id_ed25519.pub" | ssh root@[Proxmox IP address] "cat >> .ssh/authorized_keys"

You can remove the "%USERPROFILE%\" if it doesn't work properly.

Step 12 : Verify the imported SSH key on the server by accesssing to the "authorized_keys" file : 

      nano /.ssh/authorized_keys

Step 13 : Go to SSH configuration file : 

      nano /etc/ssh/sshd_config
 
And remove the comment symbol (#) by applying or editing these settings : 

- Protocol 2
- Port 22
- AddressFamily inet
- AllowUsers root@[IP address of the first machine]
- ListenAddress [Proxmox IP address]
- HostKey /etc/ssh/ssh_host_ed25519_key
- LoginGraceTime 2m
- PermitRootLogin yes
- MaxAuthTries 3
- MaxSessions 10
- PubkeyAuthentication yes
- AuthorizedKeysFile .ssh/authorized_keys
- HostBasedAuthentication no
- IgnoreRHosts yes
- PasswordAuthentication no
- PermitEmptyPasswords no
- PrintMotd no
- PrintLastLog yes
- TCPKeepAlive yes
- ClientAliveInterval 1800
- ClientAliveCountMax 3
- UseDNS no
- MaxStartups 2

Save the file by pressing Ctrl+O and exit (Ctrl+X)!

Step 14 : Testing SSH

Insert the command on Windows Command Prompt : 

      ssh root@[Proxmox IP address]

(If the default port was modified (22 by 443 for example), add "-p [port number]" to the end).