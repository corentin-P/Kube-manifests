# Creating a HomeLab with K8s

## 1. Creating the bootable USB key to install Ubuntu Server (without GUI)
Ubuntu 24.02: https://ubuntu.com/download/server

## 2. Boot the PC (raspberry) and select the SSH server option

## 3. Configure WIFI
Find the name of the Wi-Fi interface: run 
```sh
ip a
``` 
and look for the interface starting with `wl`.

Edit (or create) the file:
`/etc/netplan/01-network-manager-all.yaml`

```yaml
network:
  ethernets:
    eth0:
      dhcp4: true
      optional: true
  version: 2
  wifis:
    wlp6s0: # Wi-Fi interface name
      optional: true
      access-points:
        "<wifi name>": # Wi-Fi name
          password: "<password>" # Wi-Fi password
      dhcp4: true
```

Then run: 
```sh
sudo netplan apply
```

For more information, see this [tutorial](https://linuxconfig.org/ubuntu-20-04-connect-to-wifi-from-command-line)

## 4. Configure and start the SSH server

If it’s not installed:
```sh
sudo apt install openssh-server
```

Check the SSH service status:
```sh
sudo systemctl status ssh
```

Edit the SSH configuration:
```sh
sudo vi /etc/ssh/sshd_config
```

Enable and start the SSH server:
```sh
sudo systemctl enable ssh
sudo systemctl start ssh
```

## 5. Connect to the raspberry from another PC via SSH 

Find the PC’s IP address and take the Wi-Fi IP if connected via Wi-Fi.
```sh
ip a 
```

Open a Linux terminal (or WSL) and enter: (if you use port 22, no need to specify it)
```sh
ssh -p <port> <user>@IP
```

**You can now access the raspberry from your PC and you don't need to be on the raspberry for the next steps, just keep your terminal with ssh connections.**

## 6. Install and configure Git

Install Git:
```sh
sudo apt install git-all
```

[Create](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux#adding-your-ssh-key-to-the-ssh-agent) an SSH key

[Add](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) the SSH key to GitHub

Create a repository in the project directory:
```sh
git init -b main
```

```sh
Add files and commit:
git add . && git commit -m "init"
```

Create the repository on GitHub, copy the SSH URL and link it:
```sh
git remote add origin <url>
```

Push the project:
```sh
git push origin main
```
## 7. Configure the IP address for the switch

The switch uses the IP address: 192.168.0.1.

### On Windows

Go to the network settings and configure the network as shown in the example image
![Config example](assets/Ip_config_windows.png)

### On Linux

Check that “eth0” has no configured IP:
```sh
ip a
```

Activate the ethernet interface:
```sh
ip link set eth0 up
```

Assign a manual IP:
```sh
ip addr add <IPv4_address/mask> dev eth0
```


## 8. See all computers connected to the switch

From Linux or WSL:

Install nmap:
```sh
sudo apt install nmap
```

Find the network IPv4 and mask (example: 192.168.0.0/24)

Scan the network:
```sh
nmap -sn <network/mask>
```

Example:
```sh
nmap -sn 192.168.0.0/24
```

## 9. Share the Wi-Fi internet connection to the ethernet network

On Windows:
Go to:
Control Panel → Network and Internet → Network and Sharing Center → Change adapter settings
Then double-click the Wi-Fi network → Properties → Sharing tab.

⚠️ The PC will automatically take IP 192.168.137.1.
Configure the switch and the other PC on the same network before enabling sharing:

Switch: 192.168.137.2

Other PC: 192.168.137.3

## 10. Install k3s (lightweight Kubernetes) and Helm

[Documentation](https://docs.k3s.io/quick-start)

```sh
curl -sfL https://get.k3s.io | sh -
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
export KUBECONFIG=~/.kube/config
sudo apt-get update
sudo apt-get install helm
```

## 11. Install k9s

K9s is a great tool but I didn't manage to use it on arm64 architecture (raspberry)

## 12. Install PostgreSQL and pgAdmin

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install postgres bitnami/postgresql -f postgres/postgres-values.yml --version 18.1.10
helm repo add runix https://helm.runix.net
helm install pgadmin4 runix/pgadmin4 -f pgadmin/pgadmin-values.yml --version 1.50.0
```
