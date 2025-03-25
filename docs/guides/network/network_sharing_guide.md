
Network Sharing on Rocky Linux

This guide will help you set up network sharing on Rocky Linux using Network File System (NFS) and Samba. These methods allow easy file sharing between systems on the same network.

## Stuff ya need:
- A computer running Rocky Linux 8.6 or 9.0
- Administrator access (sudo)
- Another device to access the shared files


## **Option 1: Network File System (NFS)**

### Step 1: Install NFS Server
First, install the necessary software and start the services:
```bash
sudo dnf install nfs-utils
sudo systemctl enable --now nfs-server rpcbind
```

### Step 2: Create a Shared Folder
Make a directory for sharing files:
```bash
sudo mkdir /srv/nfs_share
sudo chown nobody:nobody /srv/nfs_share
sudo chmod 777 /srv/nfs_share
```

### Step 3: Configure the NFS Share
Add this to the `/etc/exports` file to share the folder with others:
```bash
sudo nano /etc/exports
/srv/nfs_share *(rw,sync,no_subtree_check)
```

Apply the changes:
```bash
sudo exportfs -a
sudo systemctl restart nfs-server
```

### Step 4: Adjust the Firewall
Ensure the firewall allows NFS connections:
```bash
sudo firewall-cmd --add-service=nfs --permanent
sudo firewall-cmd --reload
```

### Step 5: Access the NFS Share
On another device:
```bash
sudo mount -t nfs <server_ip>:/srv/nfs_share /mnt
```

## **Option 2: Samba**

### Step 1: Install Samba
Install Samba to share files with devices using Windows or Linux:
```bash
sudo dnf install samba samba-client
sudo systemctl enable --now smb
```

### Step 2: Create a Shared Folder
Make the directory:
```bash
sudo mkdir /srv/samba_share
sudo chown nobody:nobody /srv/samba_share
sudo chmod 777 /srv/samba_share
```

### Step 3: Configure Samba
Add the following to the `/etc/samba/smb.conf` file:
```bash
sudo nano /etc/samba/smb.conf

[share]
    path = /srv/samba_share
    browsable = yes
    writable = yes
    guest ok = yes
    read only = no
```

### Step 4: Adjust the Firewall
Allow Samba through the firewall:
```bash
sudo firewall-cmd --add-service=samba --permanent
sudo firewall-cmd --reload
```

### Step 5: Access the Samba Share
On a client system (Windows or Linux):
```bash
smbclient \\<server_ip>\share
```


## Conclusion




