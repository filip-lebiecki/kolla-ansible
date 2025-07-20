# Kolla-Ansible Single Node Deployment Guide

This repository provides a concise summary of all the essential commands and steps required to deploy a single-node OpenStack cloud using Kolla-Ansible. The guide is based on a practical walkthrough and is suitable for anyone looking to set up a robust, production-ready OpenStack environment.

## What is Kolla-Ansible?
Kolla-Ansible is a tool for deploying OpenStack services in Docker containers, using Ansible for automation. It simplifies the complexity of OpenStack deployments, making upgrades and management easier, and is widely used in production environments.

## Quick Reference: Command Summary
All commands used in the deployment process are summarized below. This includes:
- System preparation
- Networking and storage setup
- Installation and configuration
- Deployment and post-deployment steps
- Verification and initial environment setup
- Instance creation and connectivity

## How to Use This Repo
1. Review the steps for a step-by-step command reference.
2. Follow the commands in order for a successful single-node OpenStack deployment.
3. Customize configuration files as needed for your environment.

## Additional Resources
- [Kolla-Ansible Documentation](https://docs.openstack.org/kolla-ansible/latest/)
- [OpenStack Documentation](https://docs.openstack.org/)

# Kolla-Ansible Command Summary

This file summarizes all the key commands used in a single-node OpenStack deployment with Kolla-Ansible. Each command is grouped by context and includes a brief description.

---

## System Preparation
- `lsb_release -a` — Show OS version info
- `id` — Show current user info
- `sudo visudo` — Edit sudoers file for passwordless sudo (add `ubuntu   ALL=(ALL:ALL) NOPASSWD:ALL`)
- `nproc` — Show number of CPU cores
- `free -mh` — Show memory info
- `show BIOS` — Check BIOS settings (generic)
- `Set-VMProcessor -VMName stack -ExposeVirtualizationExtensions $true` — Enable nested virtualization in Hyper-V
- `sudo kvm-ok` — Check if KVM acceleration is available
- `sudo modprobe kvm-amd` — Load AMD KVM kernel module
- `sudo sh -c "echo 'options kvm-amd nested=1' >> /etc/modprobe.d/kvm-amd.conf"` — Enable nested virtualization for AMD CPUs
- `sudo modprobe -r kvm-amd` — Unload AMD KVM module
- `sudo modprobe kvm-amd` — Reload AMD KVM module
- `cat /sys/module/kvm_amd/parameters/nested` — Check if nested virtualization is enabled

## Networking
- `ip a` — Show network interfaces
- `sudo cat /etc/netplan/network.yaml` — Show network configuration

## Storage
- `sudo fdisk -l` — List disks
- `sudo pvcreate /dev/sdb` — Initialize physical volume for LVM
- `sudo vgcreate cinder-volumes /dev/sdb` — Create LVM volume group for Cinder
- `sudo vgs` — Show LVM volume groups

## Installation
- `sudo apt update` — Update package lists
- `sudo apt upgrade -y` — Upgrade system packages
- `sudo apt install git python3-dev libffi-dev gcc libssl-dev libdbus-glib-1-dev python3-venv -y` — Install Kolla-Ansible dependencies
- `python3 -m venv ~/kolla-venv` — Create Python virtual environment
- `source ~/kolla-venv/bin/activate` — Activate Python virtual environment
- `pip install -U pip` — Upgrade pip
- `pip install docker pkgconfig dbus-python` — Install required Python packages
- `pip install git+https://opendev.org/openstack/kolla-ansible@master` — Install Kolla-Ansible from source
- `sudo mkdir -p /etc/kolla` — Create Kolla config directory
- `sudo chown $USER:$USER /etc/kolla` — Change ownership of Kolla config directory
- `cp -r ~/kolla-venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla` — Copy example config files
- `ls ~/kolla-venv/share/kolla-ansible/ansible/inventory/` — List inventory files
- `head -20 ~/kolla-venv/share/kolla-ansible/ansible/inventory/all-in-one` — Show first 20 lines of all-in-one inventory
- `cp ~/kolla-venv/share/kolla-ansible/ansible/inventory/all-in-one .` — Copy inventory file to home directory
- `kolla-ansible install-deps` — Install Kolla-Ansible dependencies
- `kolla-genpwd` — Generate passwords for OpenStack services
- `cat /etc/kolla/passwords.yml` — Show generated passwords

## Configuration
- `sudo vi /etc/kolla/globals.yml` — Edit main Kolla-Ansible config
- `kolla_base_distro: "ubuntu"` — Set base OS for containers
- `ip -4 -br a` — Show IP addresses (brief)
- `ping -O 192.168.80.50` — Check if VIP is available
- `kolla_internal_vip_address: "192.168.80.50"` — Set internal VIP address
- `network_interface: "eth0"` — Set management network interface
- `neutron_external_interface: "eth1"` — Set external network interface
- `enable_cinder: "yes"` — Enable Cinder block storage
- `enable_cinder_backend_lvm: "yes"` — Enable Cinder LVM backend

## Deployment
- `kolla-ansible bootstrap-servers -i ./all-in-one` — Prepare host for OpenStack deployment
- `kolla-ansible prechecks -i ./all-in-one` — Run pre-deployment checks
- `kolla-ansible deploy -i ./all-in-one` — Deploy OpenStack services

## Post-Deployment
- `pip install python-openstackclient -c https://releases.openstack.org/constraints/upper/master` — Install OpenStack CLI
- `kolla-ansible post-deploy -i ./all-in-one` — Run post-deployment tasks
- `mkdir -p ~/.config/openstack` — Create OpenStack CLI config directory
- `cp /etc/kolla/clouds.yaml ~/.config/openstack/clouds.yaml` — Copy clouds.yaml for CLI
- `vi .bashrc` — Edit bashrc to set environment variables
- `export OS_CLOUD=kolla-admin` — Set default OpenStack cloud
- `source ~/kolla-venv/bin/activate` — Activate Python virtual environment
- `sudo usermod -aG docker $USER` — Add user to docker group
- `Ctrl+D` — Log out
- `ssh ubuntu@192.168.80.33` — Log back in

## Verification
- `openstack service list` — List OpenStack services
- `openstack compute service list` — List Nova compute services
- `openstack network agent list` — List Neutron network agents
- `openstack volume service list` — List Cinder volume services
- `docker ps` — List running Docker containers
- `grep keystone_admin_password /etc/kolla/passwords.yml` — Get Keystone admin password

## Initial Environment Setup
- `cp kolla-venv/share/kolla-ansible/init-runonce .` — Copy init-runonce script
- `vi init-runonce` — Edit init-runonce script
- `./init-runonce` — Run init-runonce script

## Instance Creation
- `openstack server create --image cirros --flavor m1.tiny --key-name mykey --network demo-net demo1` — Create a new VM instance
- `openstack server list` — List VM instances
- `openstack network list --external` — List external networks
- `openstack floating ip create public1` — Create a floating IP
- `openstack server add floating ip demo1 192.168.12.112` — Associate floating IP with instance
- `openstack server show demo1 -c addresses` — Show instance addresses
- `ssh cirros@192.168.12.112` — SSH into instance
- `gocubsgo` — Default Cirros password
- `ping dns.google` — Test outbound connectivity

