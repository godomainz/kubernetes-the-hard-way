# Provisioning Compute Resources

Note: You must have VirtualBox and Vagrant configured at this point

Download this github repository and cd into the vagrant folder

`git clone https://github.com/mmumshad/kubernetes-the-hard-way.git`

CD into vagrant directory

`cd kubernetes-the-hard-way\vagrant`

Run Vagrant up

`vagrant up`


This does the below:

- Deploys 5 VMs - 2 Master, 2 Worker and 1 Loadbalancer with the name 'kubernetes-ha-* '
    > This is the default settings. This can be changed at the top of the Vagrant file

- Set's IP addresses in the range 192.168.5

    | VM            |  VM Name               | Purpose       | IP           | Forwarded Port   |
    | ------------  | ---------------------- |:-------------:| ------------:| ----------------:|
    | master-1      | kubernetes-ha-master-1 | Master        | 192.168.5.11 |     2711         |
    | master-2      | kubernetes-ha-master-2 | Master        | 192.168.5.12 |     2712         |
    | worker-1      | kubernetes-ha-worker-1 | Worker        | 192.168.5.21 |     2721         |
    | worker-2      | kubernetes-ha-worker-2 | Worker        | 192.168.5.22 |     2722         |
    | loadbalancer  | kubernetes-ha-lb       | LoadBalancer  | 192.168.5.30 |     2730         |

    > These are the default settings. These can be changed in the Vagrant file

- Add's a DNS entry to each of the nodes to access internet
    > DNS: 8.8.8.8

- install vim
sudo apt-get install vim -y

- Change the hostname
 sudo hostnamectl set-hostname loadbalancer
- Edit the /etc/hosts file with hostname
sudo vim /etc/hosts


- sudo apt update && sudo apt install -y jq && sudo apt install net-tools -y && sudo apt install htop -y && sudo apt install git -y && sudo apt install openssh-server -y

- add your public key to .ssh/authorized_keys
cat > ~/.ssh/authorized_keys <<EOF
blah blah
EOF


- ssh/config
cat > ~/.ssh/config <<EOF
Host *
    ServerAliveInterval 60

Host jenkins
 User root
 Hostname jenkins.oneit.com.au
 IdentityFile ~/.ssh/id_rsa
 Port 40022

Host prod-oneit-files.oneit.com.au
 User jenkins
 Hostname prod-oneit-files.oneit.com.au
 Port 40022
 ProxyCommand ssh -o 'ForwardAgent yes' jenkins 'ssh-add && nc %h %p'
EOF

- add your authorized_keys,  id_rsa,  id_rsa.pub
cat > ~/.ssh/authorized_keys <<EOF
blah blah akila@ubuntu
EOF

cat > ~/.ssh/id_rsa <<EOF
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA4JCrDKhnU93VfzjYaSE8MQtDm5cRYXfNV6k/ioHxkIvlYErw
EOUC39qtK28/rzY3ejmWVW0w2Uhp0wwczZaOM8bxJ2bIsL8d8DR5XlLrkKHhnbzr
JjJsUe/MYk2+MLJvsgI/temY5qTTjZ+3bckWrmidszeU+EchtPhuw6gYwPnxmIZr
BpH5J6c9fVmt2INYoRjVeRoRFlbF1EGuIYvV5extbwxeNQnL1JVX3iFlLwDchRIZ
Hsz8d+K3tNRo/jhV9IfHPpyQHQYUWBJ1lIqQrWHKi7mGXvVqwrETEQvPdgSnVp0t
sBcsc7Nhseln+g2C9s8Bk12HaZoF/RjcUhz4IQIDAQABAoIBAAvtgzhb5Ykd2k40
ncIPwtu0BnZIMuMjcuO6GKbpugP8ekWAFXpAP8PWIKaS9SYAUjgKwQJul06jOwO7
u/frjEgRxBNcsUI6FIQCtYOeEecPwiUXuMHBoeFERG3gRT7e63HgDrRB4R43GQmH
tz18ldjTs7SmOiJp3M949qEr14zAYGuKlLcyRqFzeLqr6xK+6rDvUGvJxRL/Lb5U
IbUMzFayin3WqywUSVAKpXDxcRDxts+5ylP/c9uI1JqDrcLYiTtpKu+DL0NagTqP
vEe0kjcgCR+cFhJTOjhkhpMpEd1q55KiZuK8s53qTQR+bjx9NKzxa3OTYYi8t8fs
pa1niAECgYEA9Tr2llTKoaLdUkmKcVmdWNicXBmha47wk7qHGnEKwfkD7Mpsb6+t
SSqvodSzUfWovtWOEb1+QVWDD8/2FqnXWZvQyJY/mpQ6zrTSzItVOty4PzQdJpmW
Bz24pY7wYblQB2Af/gpItu+/kROePdHg2VMwoSKnzJUzlycS8HgLCsECgYEA6m1f
kX5MworinZreLyLAPWwd8kTC7ylk0UgknzjjOln638dTJyq5SqyMP5c6TP/sirdX
Kgi0mPAq16vHbgQyNQ0oXwV1ZD/Ip7zq4L+aiPL87DkmJFGYatP5KmMsDBxmmO/M
bR5fT5iM3GZxNK3qPfZrh7ZLAxzk4yHjF3VgJWECgYEAjb3e+VVZKcPxGLbZBls9
zzSka7eEzZ54/2o43Nep2CQOWLdHpeZsynWZvngqjZzoRCU7UJWufCTo9CLHoqHY
jzq4mrf9W2OB+igaD5AZW0RoWl/M2Zq8VMMgDtFnr5Rk5V5yH2viS5qXp0snk6PT
ysmCuiBFzMIQZ7V2BPfdqgECgYEAjsqGLsoemVUdieBeO5nQPNmROBOYJTMyfKOT
4wQ0rENIo2v3A2Frscd+OfG0ilhMzYW1ax4YWxvXDL1OYX3e0x+rmo1pnuGXKEzT
SIiM6aQQWRbKW87zpwZsu9viZZIbEEboXwLkDUifbFRd2jeg+ZMSlnx8Hm5IIO1w
NMbDBKECgYAi8F3Us98inbTRAmgL4K28y5PYVOdyoCVSVGrtQhoZ48QYSHLCCn2W
6trAgDQTJ6oMmijCOlh6d7xqPcZt6oP5J8uAv2CPPuF+JN5vIidiwa2NzZlLfGxn
GeeC4B3tipji966mF6i9Aalhiyp8aT/2YJqDT5/HTu/IYelqDqMp3A==
-----END RSA PRIVATE KEY-----
EOF

cat > ~/.ssh/id_rsa.pub <<EOF
blah blah akila@ubuntu
EOF


- Runs the below command on all nodes to allow for network forwarding in IP Tables.
  This is required for kubernetes networking to function correctly.
    > sudo sysctl net.bridge.bridge-nf-call-iptables=1
sudo modprobe br_netfilter && sudo sysctl -p /etc/sysctl.conf && sudo sysctl net.bridge.bridge-nf-call-iptables=1

- trun off swap
cat > ~/swapoff.sh <<EOF
#!/bin/bash
swapoff -a
if [[ "${?}" -ne 0 ]]
then
  echo 'The command swapoff -a did not execute successfully.'
  exit 1
fi
echo 'The command swapoff -a did execute successfully.'
exit 0
EOF
chmod +x ~/swapoff.sh


cat <<EOF | sudo tee /etc/systemd/system/swapoff.service
[Unit]
Description=swapoff script

[Service]
ExecStart=/home/akila/swapoff.sh

[Install]
WantedBy=multi-user.target
EOF


{
  sudo systemctl enable swapoff --now
  sudo service swapoff start
}

sudo swapon --show

- Install's Docker on Worker nodes

- Enable ip forwarding
{
    sudo ufw allow 443/tcp
    sudo ufw allow 6783/tcp
    sudo ufw allow 6783/udp
    sudo ufw allow 6784/udp
}

{
  sudo sysctl net.ipv4.ip_forward=1
  echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
  sudo sysctl -p
}




## SSH to the nodes

There are two ways to SSH into the nodes:

### 1. SSH using Vagrant

  From the directory you ran the `vagrant up` command, run `vagrant ssh <vm>` for example `vagrant ssh master-1`.
  > Note: Use VM field from the above table and not the vm name itself.

### 2. SSH Using SSH Client Tools

Use your favourite SSH Terminal tool (putty).

Use the above IP addresses. Username and password based SSH is disabled by default.
Vagrant generates a private key for each of these VMs. It is placed under the .vagrant folder (in the directory you ran the `vagrant up` command from) at the below path for each VM:

**Private Key Path:** `.vagrant/machines/<machine name>/virtualbox/private_key`

**Username:** `vagrant`


## Verify Environment

- Ensure all VMs are up
- Ensure VMs are assigned the above IP addresses
- Ensure you can SSH into these VMs using the IP and private keys
- Ensure the VMs can ping each other
- Ensure the worker nodes have Docker installed on them. Version: 18.06
  > command `sudo docker version`

## Troubleshooting Tips

If any of the VMs failed to provision, or is not configured correct, delete the vm using the command:

`vagrant destroy <vm>`

Then reprovision. Only the missing VMs will be re-provisioned

`vagrant up`


Sometimes the delete does not delete the folder created for the vm and throws the below error.

VirtualBox error:

    VBoxManage.exe: error: Could not rename the directory 'D:\VirtualBox VMs\ubuntu-bionic-18.04-cloudimg-20190122_1552891552601_76806' to 'D:\VirtualBox VMs\kubernetes-ha-worker-2' to save the settings file (VERR_ALREADY_EXISTS)
    VBoxManage.exe: error: Details: code E_FAIL (0x80004005), component SessionMachine, interface IMachine, callee IUnknown
    VBoxManage.exe: error: Context: "SaveSettings()" at line 3105 of file VBoxManageModifyVM.cpp

In such cases delete the VM, then delete the VM folder and then re-provision

`vagrant destroy <vm>`

`rmdir "<path-to-vm-folder>\kubernetes-ha-worker-2"`

`vagrant up`
