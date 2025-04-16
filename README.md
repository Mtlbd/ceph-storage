# Install Ceph Cluster by Ceph-deploy on Ubuntu 18.04

---

## 💼 VM Setup

Add **two hard disks** to each VM **before installing Ubuntu**:
1. **50 GB** for OS (Ubuntu installation)
2. **1 TB** unallocated disk for OSD

---

## ⚖️ Hostname Configuration

Add the following lines to `/etc/hosts` on **all nodes**:

```
192.168.1.31	ceph-A
192.168.1.32	ceph-N1
192.168.1.33	ceph-N2
192.168.1.34	ceph-N3
```

---

## ⏱️ Install NTP

Install and configure `ntp` on all nodes to ensure time synchronization.

---

## 👽 Install Python

Install Python on all nodes:
```bash
sudo apt install python -y
```

---

## 👤 Create `ceph-admin` User

On **all nodes**:
```bash
sudo adduser ceph-admin
sudo usermod -aG sudo ceph-admin
sudo visudo
```
Add the following line:
```
ceph-admin ALL=(ALL:ALL) NOPASSWD:ALL
```

---

## 🔐 SSH Access Setup from Admin Node

Login as `ceph-admin` to the admin node (e.g., `192.168.1.31`) and run:

```bash
ssh-keygen
ssh-copy-id ceph-admin@ceph-A
ssh-copy-id ceph-admin@ceph-N1
ssh-copy-id ceph-admin@ceph-N2
ssh-copy-id ceph-admin@ceph-N3
```

---

## 🚀 Install Ceph-deploy (Admin Node)

```bash
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
echo deb https://download.ceph.com/debian-nautilus/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
sudo apt update
sudo apt -y install ceph-deploy
```

---

## 🔹 Deploy Initial Ceph Configuration

```bash
ceph-deploy new ceph-A ceph-N1 ceph-N2 ceph-N3
ceph-deploy install --release nautilus ceph-A ceph-N1 ceph-N2 ceph-N3
ceph-deploy mon create-initial
ceph-deploy admin ceph-A ceph-N1 ceph-N2 ceph-N3
```

---

## ⚡ OSD Node Config (All OSD Nodes)

```bash
sudo ceph config set mon auth_allow_insecure_global_id_reclaim false
sudo reboot
```

---

## 📊 Check Cluster Status (Any Node)

```bash
sudo ceph status
```

Expected output:
```
cluster:
  id:     <UUID>
  health: HEALTH_OK
services:
  mon: 3 daemons, quorum ceph1,ceph2,ceph3
  mgr: no daemons active
  osd: 0 osds: 0 up, 0 in
```

---

## ➕ Add OSDs (Admin Node)

```bash
ceph-deploy osd create --data /dev/sdb ceph-A
ceph-deploy osd create --data /dev/sdb ceph-N1
ceph-deploy osd create --data /dev/sdb ceph-N2
ceph-deploy osd create --data /dev/sdb ceph-N3
```

---

## 💾 Create Manager Daemon (Admin Node)

```bash
ceph-deploy mgr create ceph-A
```

---

## 🔍 Enable Dashboard (on ceph-A)

```bash
sudo apt install -y ceph-mgr-dashboard
sudo ceph config set mgr mgr/dashboard/ssl false
sudo ceph mgr module enable dashboard
sudo ceph dashboard ac-user-create admin -i <password-file> administrator
```
> `<password-file>` is a file containing the password for the dashboard user.

---

## 🌐 Create RGW (Admin Node)

```bash
ceph-deploy rgw create ceph-A
```

Add to `/etc/ceph/ceph.conf` on `ceph-A`:
```
[client]
rgw_frontends = civetweb port=80
```

---

## 📂 Configure RGW User (on ceph-A)

```bash
sudo radosgw-admin user create --uid=admin --display-name='Ceph Admin' --system
sudo ceph dashboard set-rgw-api-access-key -i <api-access-key-file>
sudo ceph dashboard set-rgw-api-secret-key -i <api-secret-key-file>
sudo ceph dashboard set-rgw-api-ssl-verify False
sudo ceph mgr module disable dashboard && sudo ceph mgr module enable dashboard
```

---

## 🎉 Done!

Your Ceph Cluster is now installed and configured using Ceph-deploy on Ubuntu 18.04.

---

**Author:** Mohammad Talebi

