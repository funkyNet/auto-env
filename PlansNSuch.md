# How to setup terraform to interact with Proxmox
https://learn.hashicorp.com/tutorials/terraform/install-cli  
```
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform
```

## Create a template that I can just clone
https://austinsnerdythings.com/2021/08/30/how-to-create-a-proxmox-ubuntu-cloud-init-image/  
`wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img -P cd /var/lib/vz/vz/template/iso` 
## Prep for image
sudo apt update -y && sudo apt install libguestfs-tools -y
## insert qemu into image
sudo virt-customize -a focal-server-cloudimg-amd64.img ##install qemu-guest-agent
## Build VM via cli
sudo qm create 9000 ##name "ubuntu-2004-cloudinit-template" ##memory 2048 ##cores 2 ##net0 virtio,bridge=vmbr0
sudo qm importdisk 9000 focal-server-cloudimg-amd64.img local-lvm
sudo qm set 9000 ##scsihw virtio-scsi-pci ##scsi0 local-lvm:vm-9000-disk-0
sudo qm set 9000 ##boot c ##bootdisk scsi0
sudo qm set 9000 ##ide2 local-lvm:cloudinit
sudo qm set 9000 ##serial0 socket ##vga serial0
sudo qm set 9000 ##agent enabled=1

## create template out of VM
sudo qm template 9000

## Follow guide to use terraform against proxmox
https://austinsnerdythings.com/2021/09/01/how-to-deploy-vms-in-proxmox-with-terraform/

## Setup terraform
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=$(dpkg ##print-architecture)] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update
sudo apt install terraform

## Setup user for API key
SKIPPED and just used the root pwd

## Terraform has 3 stages, plan, init, apply
## Plan
cd ~
mkdir terraform-demo && cd terraform-demo
touch main.tf vars.tf

## edit main.tf
terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
      version = "2.7.4"
    }
  }
}

## init terraform bare bones
terraform init

## update main.tf
see main.tf file

## GAP create SSH keys
https://peter-whyte.com/setup-ssh-keys-in-wsl/
ssh-keygen -t rsa -b 4096 -C "mattfunk20@email.com"
pub key saved in /home/funky/.ssh/id_rsa.pub

## update vars.tf
variable "ssh_key" {
  default = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCrCPmO9EwSE6w71iwiTXmnJXbHbdT3q5/qK7otgqPucsqGimdb9jxHkfEwsT40BbVXq/Eu3Du+cRblGAgOVMWDmHfJ4G/myUt/0hCByu1B2v/OVtDXmtas/zIdoSVS2xe37iVg4qxiZSqHo7y5eNHsTsBKYpwWRyo6FODYJVqTbn9zQum7MhEQTs1IKt9Mj2JloACuRuQk78YoWBjghCl7QkhEdsdViNx+zxOmVdyxKUZKmQ2ScwE8aTjun95jQTDEtQ8S/BQPjA3e5QqxrJvEmT8Iczcf1TiOzbyTobLvxDU+Pw08MKegjzCVIFGO8HNYHgfyW8PcjYqh+O+XLuQ+HvCjiEMPKRtEzn75fGE9w91yn1J5Jof6I7QV6UlhLltYdh5k92xzoX60JLH0eglSy7KXNtzePODAueLGEXrWyIRVZ286K7Lmh9AeWzekXWZID8OK8jLu7vX3D8jZCiOnnhZIiA0Rl7QGsAsKDxe0bPmi/Id3lvXSnpFveuwFKL0zqEgdxQeSePmeFvGk2i3X7HYyDYhYzEefZJPeuxrlGyrr6zPQayw2Ea7n0luaegRiI4/uaOfVIA7cuD1K5YNUWR6+EJGqJ/QhAaWZhOL2sufD3cyyJYKJuFCFH3hf3nWxfYBuVkhCr8gRLTEi6+fV4DWV9TAqrFZOF21KqiWWjw== mattfunk20@email.com"
}
variable "proxmox_host" {
    default = "pve"
}
variable "template_name" {
    default = "ubuntu-2004-cloudinit-template"
}





############ REFERENCES
https://medium.com/@archanabalasundaram/packer-with-terraform-8c45f895cddb

