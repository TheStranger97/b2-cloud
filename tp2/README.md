# TP2

# Part I : Programmatic approach

---

## I. Premiers pas

🌞 **Créez une VM depuis le Azure CLI**

```bash
az group list
```

```bash
az group create --location westeurope -n tp2
```

```bash
az vm create -g tp2 -n super_vm --image Ubuntu2204 --admin-username ts --ssh-key-values C:/Users/samue/.ssh/azureuser.pub
```

🌞 **Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique.**

```bash
ts@supervm:~$ systemctl status walinuxagent
● walinuxagent.service - Azure Linux Agent
     Loaded: loaded (/lib/systemd/system/walinuxagent.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2025-04-01 08:19:47 UTC; 1min 35s ago
```

```bash
ts@supervm:~$ systemctl status cloud-init
● cloud-init.service - Cloud-init: Network Stage
     Loaded: loaded (/lib/systemd/system/cloud-init.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2025-04-01 08:19:46 UTC; 1min 46s ago
```

## II. Un ptit LAN

---

🌞 **Créez deux VMs depuis le Azure CLI**

```bash
az vm create -g tp2 -n super_vm1 --image Ubuntu2204 --admin-username ts --ssh-key-values C:/Users/samue/.ssh/azureuser.pub
```

```bash
az vm create -g tp2 -n super_vm2 --image Ubuntu2204 --admin-username ts --ssh-key-values C:/Users/samue/.ssh/azureuser.pub
```

```bash
ts@supervm1:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 60:45:bd:9c:51:54 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.5/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::6245:bdff:fe9c:5154/64 scope link
       valid_lft forever preferred_lft forever
ts@supervm1:~$ ping 10.0.0.6
PING 10.0.0.6 (10.0.0.6) 56(84) bytes of data.
64 bytes from 10.0.0.6: icmp_seq=1 ttl=64 time=4.44 ms
```

```bash
ts@supervm2:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 7c:ed:8d:12:59:c2 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.6/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::7eed:8dff:fe12:59c2/64 scope link
       valid_lft forever preferred_lft forever
ts@supervm2:~$ ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=1.06 ms
```

# Part II : cloud-init

---

## 2. Gooooo

---

🌞 **Tester `cloud-init`**

```bash
az>> az vm create -g tp2 -n super_vm3 --image Ubuntu2204 --admin-username ts --ssh-key-values C:/Users/samue/.ssh/azureuser.pub --custom-data C:/le/path/est/trop/long/cloud-init.txt
```

🌞 **Vérifier que `cloud-init` a bien fonctionné**

```bash
PS C:\Users\samue> ssh -i C:/Users/samue/.ssh/azureuser ts@20.126.92.154
```

```bash
ts@supervm3:~$ whoami
ts
```

## 3. Write your own

---

🌞 **Utilisez `cloud-init` pour préconfigurer la VM :**

```yaml
#cloud-config

apt:
  sources:
    docker.list:
      source: deb [arch=amd64] https://download.docker.com/linux/ubuntu $RELEASE stable
      keyid: 9DC858229FC7DD38854AE2D88D81803C0EBFCD88

packages:
  - apt-transport-https
  - ca-certificates
  - curl
  - gnupg-agent
  - software-properties-common
  - docker-ce
  - docker-ce-cli
  - containerd.io

users:
  - default
  - name: ds
    groups: docker
    passwd: 75fdb2605438c696a4725bb78b5f8aeca444e7d54c6f3d17f5d9eb89bd2e7f63
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILWACUx/CGzlF9crpxI++wYma+qwoah06V0p9c/UC5pw samue@Wawa

runcmd:
  - docker image pull alpine:latest

```

# Part III : Terraform

---

## 2. Copy paste

---

🌞 **Constater le déploiement**

```bash
az>> az vm list -o table
Name            ResourceGroup          Location    Zones
--------------  ---------------------  ----------  -------
tp2magueule-vm  TP2MAGUEULE-RESOURCES  westeurope
az>> vm show --name tp2magueule-vm --resource-group tp2magueule-resources -o table
Name            ResourceGroup          Location    Zones
--------------  ---------------------  ----------  -------
tp2magueule-vm  tp2magueule-resources  westeurope
az>> az group list -o table
Name                   Location    Status
---------------------  ----------  ---------
tp1                    westeurope  Succeeded
NetworkWatcherRG       westeurope  Succeeded
tp2                    westeurope  Succeeded
tp2magueule-resources  westeurope  Succeeded
```

## 3. Do it yourself

---

🌞 **Créer un *plan Terraform* avec les contraintes suivantes**

```hcl
provider "azurerm" {
  features {}
  subscription_id = "ef8322d4-8f87-4612-98ca-fcfc18dbe74c"
}
resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}
resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}
resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}
resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}
resource "azurerm_network_interface" "main" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}
resource "azurerm_network_interface" "internal" {
  name                = "${var.prefix}-nic2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}
resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = azurerm_network_interface.main.private_ip_address
  }
}
resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}
resource "azurerm_linux_virtual_machine" "main" {
  name                            = "${var.prefix}-vm"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_F2"
  admin_username                  = "azureuser"
  network_interface_ids = [
    azurerm_network_interface.main.id,
  ]
  admin_ssh_key {
    username   = "azureuser"
    public_key = file("C:/Users/samue/.ssh/azureuser.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}
resource "azurerm_linux_virtual_machine" "internal" {
  name                            = "${var.prefix}-vm2"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_F2"
  admin_username                  = "azureuser"
  network_interface_ids = [
    azurerm_network_interface.internal.id,
  ]
  admin_ssh_key {
    username   = "azureuser"
    public_key = file("C:/Users/samue/.ssh/azureuser.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

```

```hcl
variable "prefix" {
  description = "da prefix"
  default = "tp2terraform"
}
variable "location" {
  description = "da location"
  default = "West Europe"
}
```

```bash
azureuser@tp2terraform-vm:~$ ping 10.0.2.5
PING 10.0.2.5 (10.0.2.5) 56(84) bytes of data.
64 bytes from 10.0.2.5: icmp_seq=1 ttl=64 time=2.32 ms
64 bytes from 10.0.2.5: icmp_seq=2 ttl=64 time=1.00 ms
```

```bash
azureuser@tp2terraform-vm2:~$ ping 10.0.2.4
PING 10.0.2.4 (10.0.2.4) 56(84) bytes of data.
64 bytes from 10.0.2.4: icmp_seq=1 ttl=64 time=0.863 ms
64 bytes from 10.0.2.4: icmp_seq=2 ttl=64 time=0.988 ms
```

## 4. cloud-iniiiiiiiiiiiiit

---

🌞 **Intégrer la gestion de `cloud-init`**

🌞 **Proof !**

livrez votre `main.tf` dans le compte-rendu

```hcl
provider "azurerm" {
  features {}
  subscription_id = "ef8322d4-8f87-4612-98ca-fcfc18dbe74c"
}
resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}
resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}
resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}
resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}
resource "azurerm_network_interface" "main" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}
resource "azurerm_network_interface" "internal" {
  name                = "${var.prefix}-nic2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}
resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = azurerm_network_interface.main.private_ip_address
  }
}
resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}
resource "azurerm_linux_virtual_machine" "main" {
  name                            = "${var.prefix}-vm"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_B1s"
  admin_username                  = "azureuser"
  network_interface_ids = [
    azurerm_network_interface.main.id,
    azurerm_network_interface.internal.id,
  ]
  admin_ssh_key {
    username   = "azureuser"
    public_key = file("C:/Users/samue/.ssh/azureuser.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
  custom_data = filebase64("cloud-init/cloud-init.txt")
}

```

livrez votre `cloud-init.txt` dans le compte-rendu

```yaml
#cloud-config

apt:
  sources:
    docker.list:
      source: deb [arch=amd64] https://download.docker.com/linux/ubuntu $RELEASE stable
      keyid: 9DC858229FC7DD38854AE2D88D81803C0EBFCD88

packages:
  - apt-transport-https
  - ca-certificates
  - curl
  - gnupg-agent
  - software-properties-common
  - docker-ce
  - docker-ce-cli
  - containerd.io

users:
  - default
  - name: ds
    groups: docker
    passwd: 75fdb2605438c696a4725bb78b5f8aeca444e7d54c6f3d17f5d9eb89bd2e7f63
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILWACUx/CGzlF9crpxI++wYma+qwoah06V0p9c/UC5pw samue@Wawa

runcmd:
  - docker image pull alpine:latest

```

vous pouvez vous connecter sans password en SSH sur le nouveau user (ptite commande `ssh` dans le compte-rendu)

```yaml
PS C:\Users\samue> ssh -i C:/Users/samue/.ssh/azureuser ds@52.166.34.24
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Apr  1 18:53:22 UTC 2025

  System load:  0.11              Processes:             121
  Usage of /:   7.9% of 28.89GB   Users logged in:       0
  Memory usage: 40%               IPv4 address for eth0: 10.0.2.4
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.

16 updates can be applied immediately.
16 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

New release '24.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

ds@tp2cloudinit-vm:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

## B. Go further

---

🌞 **Moar `cloud-init` and Terraform configuration**