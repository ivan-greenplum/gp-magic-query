# How to install from Google Marketplace [ note this may not provide latest software, methods below are preferred ]
- Go to google console for Greenplum Deployment: https://console.cloud.google.com/marketplace/details/pivotal-public/pivotal-greenplum-byol
- Click Launch
- Choose Greenplum 6 and all optional installs
- Wait for the cluster to be deployed
- SSH to the cluster using gcloud command (More on gcloud: https://cloud.google.com/sdk/gcloud/)

```
gcloud beta compute --project "data-gpdb-ud" ssh --zone "us-east1-b" "divya-greenplum-byol-1-mdw"
$ sudo su -  gpadmin
```

# How to install on Windows
- Install Rocky Linux version 8 using Microsoft Store
- Make sure Rocky Linux is working from Windows

- Update OS:
```
sudo dnf update
sudo dnf makecache

sudo dnf install openssh-server
sudo dnf install libcap
sudo dnf install vim
```

- Setup user for Greenplum
```
sudo groupadd gpadmin
sudo useradd -g gpadmin gpadmin
sudo passwd gpadmin
visudo
# add this line
gpadmin ALL=(ALL) NOPASSWD: ALL
```

- Setup ssh
```
# Login as root
ssh-keygen -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N ''
ssh-keygen -t ecdsa -b 256 -f /etc/ssh/ssh_host_ecdsa_key -N ''
ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ''
/usr/sbin/sshd

# Login as gpadmin
ssh-keygen -t rsa -b 2048
ssh-copy-id gpadmin@127.0.0.1
```
test ssh to 127.0.0.1 with no password to ensure this is ready

- setup ping
```
sudo setcap cap_net_raw+p /usr/bin/ping
```
Test ping to 127.0.0.1

- Install greenplum software
Copy software to linux as root user
```
cp /mnt/c/Users/ivan/Downloads/greenplum-db-7.1.0-el8-x86_64.rpm .
dnf install greenplum-db-7.1.0-el8-x86_64.rpm
chown -R gpadmin:gpadmin /usr/local/greenplum-db/
```

- Add this content to .bashrc for gpadmin
```
export GPHOME=/usr/local/greenplum-db
. $GPHOME/greenplum_path.sh
export COORDINATOR_DATA_DIRECTORY=/home/gpadmin/gp/gpsne-1
```
. .bashrc

- Initialize Greenplum
```
mkdir gp
cd gp
cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_singlenode .
```
Now edit gpinitsystem_singelnode by updating these lines:
```
declare -a DATA_DIRECTORY=(/home/gpadmin/gp /home/gpadmin/gp)
COORDINATOR_HOSTNAME=127.0.0.1
COORDINATOR_DIRECTORY=/home/gpadmin/gp
```

Create hostlist_singlenode file in /home/gpadmin/gp and add only 127.0.0.1 to the file
```
echo "127.0.0.1" > /home/gpadmin/gp/hostlist_singlenode
```
- initialize the clsuter [ Note if ssh error comes up first time enter yes ]
```
gpinitsystem -c gpinitsystem_singlenode
```

add to .bashrc this variable
```
export COORDINATOR_DATA_DIRECTORY=/home/gpadmin/gp/gpsne-1
```

Validate that  you can restart Greenplum
```
gpstop
gpstart
```

# How to install GPText ontop of the Greenplum deploy

### Login to master as gpadmin
* go the deployment screen on GCP and use this section to connect: **Connect with gcloud**

