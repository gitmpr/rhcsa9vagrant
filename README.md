# RHCSA 9 Vagrant Practise lab

![screenshot](https://github.com/gitmpr/rhcsa9vagrant/blob/main/screenshot/Screenshot%20from%202023-04-06%2022-24-00.png?raw=true)

Vagrant lab for practising for the rhcsa 9. It was mentioned on the Red Hat Slack that RHEL9.0 is used on the latest RHCSA exam, 
not the 9.1 release. So the vagrant box is pinned to the older version
https://join.slack.com/t/redhat-certs/shared_invite/zt-1ss7ia1sk-Hn3mMGPRaJIfKbiYP89ZoA

# How to use this lab
- Clone this repo
- Download the rhel 9 dvd iso into the same directory (!this is not the latest version!) https://developers.redhat.com/content-gateway/file/rhel-baseos-9.0-x86_64-dvd.iso

- install virtualbox
- install vagrant
- run `vagrant up` inside this directory
    
The vagrant script will set up two RHEL9.0 VMs: anode and cathode. 
anode will mount the dvd iso and use this for its own repo to use.
It will also run an Apache web server serving the BaseOS and AppStream repos which are used by the cathode VM.

In two separate terminals you want to execute the following commands to ssh into the boxes

`vagrant ssh anode`

`vagrant ssh cathode`


or use the following multiline command to create a tmux session:

    tmux new-session -s rhcsa9vagrant -n rhcsa9vagrant \
       'tmux split-window -h "vagrant up; bash"; \
        tmux split-window -v -p 75; \
        tmux split-window -h -p 50; \
        tmux send-keys -t vagrant:0.2 "vagrant ssh anode"; \
        tmux send-keys -t vagrant:0.3 "vagrant ssh cathode"; \
        tmux select-pane -t 0'


or use the tmuxp file:
`tmuxp load tmuxp/rhcsa9vagrant.yaml`

TODO
- This has only been tested on Arch linux. For windows/mac you might need to use different IDE controller names for virtualbox

- Figure out a way to, through a modification in the Vagrantfile, disable internet access through the host NAT while still having vagrant ssh access.

- tmuxp config

- UEFI install

- ftp yum repo

- lvm config disable device file
https://www.burakkurc.com.tr/en/2023/01/29/rhel-9-lvm-devices-file-sys_wwid-nvme-pvid-last-seen-on-not-found/
`sudo sed -i 's/use_devicesfile = 1/use_devicesfile = 0/g' /etc/lvm/lvm.conf`
