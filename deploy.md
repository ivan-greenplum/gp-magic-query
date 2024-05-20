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

### GPText Pre-Requisites:
- Install lsof on all hosts
```
sudo yum install lsof
```
- put this text into dirs.sh 
```
mkdir /usr/local/greenplum-text-3.4.0
mkdir /usr/local/greenplum-solr
chown gpadmin:gpadmin /usr/local/greenplum-text-3.4.0
chmod 775 /usr/local/greenplum-text-3.4.0
chown gpadmin:gpadmin /usr/local/greenplum-solr
chmod 775 /usr/local/greenplum-solr
````
- Run to create the needed directories
```
sudo bash ./dirs.sh
```

### Download GPText from PivNet
* NOTE: Get your api token by logging into Pivnet and going under your profile to find the token string
* Pick get new refresh token and find the token string
```
wget https://github.com/pivotal-cf/pivnet-cli/releases/download/v1.0.0/pivnet-linux-amd64-1.0.0
chmod 755 ./pivnet-linux-amd64-1.0.0
./pivnet-linux-amd64-1.0.0 login --api-token='my-api-token' 
./pivnet-linux-amd64-1.0.0 download-product-files --product-slug='pivotal-gpdb' --release-version='6.3.0' --product-file-id=579663
```

### Extract and grant execute permission to the GPText binary. For example:
```
tar xzvf greenplum-text-3.4.0-rhel7_x86_64.tar.gz
chmod +x /home/gpadmin/greenplum-text-3.4.0-rhel7_x86_64.bin
```

### Create Install Config File

* Modify the `gptext_install_config` file to set the parameters for installation. See the following link for details. [Set GPText Installation Parameters](http://gptext.docs.pivotal.io/340/topics/installing.html#topic1__edit_config)

```
# FILE NAME: gptext_install_config
GPTEXT_HOSTS="ALLSEGHOSTS"
declare -a DATA_DIRECTORY=(/data1/primary /data1/primary)
JAVA_OPTS="-Xms1024M -Xmx1024M"
GPTEXT_PORT_BASE=18983
GP_MAX_PORT_LIMIT=28983
ZOO_CLUSTER="BINDING"
declare -a ZOO_HOSTS=(mdw mdw mdw)
ZOO_DATA_DIR="/data1/master/"
ZOO_GPTXTNODE="gptext"
ZOO_PORT_BASE=2188
ZOO_MAX_PORT_LIMIT=12188
```

### Run Install Command

* Run the GPText installation binary as `gpadmin` on the master server.

```
./greenplum-text-3.4.0-rhel7_x86_64.bin -c gptext_install_config
```

* Accept the Pivotal license agreement and respond to the installer’s prompts.

### Source and install into the database being used
* Source the greenplum-text_path.sh
```
source /usr/local/greenplum-text-3.4.0/greenplum-text_path.sh
```

* Install the gptext schema into the `gpadmin` database
```
gptext-installsql gpadmin
```
