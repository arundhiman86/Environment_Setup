# Environment Setup for DevOps

#### Environment VM's

**VM1** - ```ssh user@<ip_address>```

**VM2** - ```ssh user@<ip_address>```

**VM3** - ```ssh user@<ip_address>```

---

#### AWS Environment Machines

**Kops manager** - ```ssh -i "<private_key.pem>" ubuntu@<public_ip_address>```

**kubernetes master** - ```ssh -i "<private_key.pem>" ubuntu@<public_ip_address>```

**Kubernetes slave1** - ```ssh -i "<private_key.pem>" ubuntu@<public_ip_address>```

----

# Environment OS properties VM1, VM2, VM3

```sh
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.4 LTS
Release:	16.04
Codename:	xenial
```

# Setup Jenkins on VM1

```sh
$ sudo apt-get update
$ sudo apt-get install openjdk-8-jdk
$ java -version 
$ wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
$ sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
$ sudo apt-get update
$ sudo apt-get install jenkins
$ systemctl status jenkins.service
$ ps -aux | grep jenkins
```

**Note:-**

1. To check jenkins version -> cat /var/lib/jenkins/config.xml (```<version>2.107.3</version>```)
2. To find java home path -> ```jrunscript -e 'java.lang.System.out.println(java.lang.System.getProperty("java.home"));'```

```sh
$ java -version
openjdk version "1.8.0_171"
OpenJDK Runtime Environment (build 1.8.0_171-8u171-b11-0ubuntu0.16.04.1-b11)
OpenJDK 64-Bit Server VM (build 25.171-b11, mixed mode)
```

# Setup Ansible on VM2 

```sh
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```

# Setup Docker on VM1, VM2, VM3

First, add the GPG key for the official Docker repository to the system:

```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add the Docker repository to APT sources:

```sh
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Next, update the package database with the Docker packages from the newly added repo:

```sh
sudo apt-get update
```

Make sure you are about to install from the Docker repo instead of the default Ubuntu 16.04 repo:

```sh
apt-cache policy docker-ce
```

Output of apt-cache policy docker-ce

```sh
$ sudo apt-cache policy docker-ce
docker-ce:
  Installed: 18.03.0~ce-0~ubuntu
  Candidate: 18.03.1~ce-0~ubuntu
  Version table:
     18.03.1~ce-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 *** 18.03.0~ce-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
        100 /var/lib/dpkg/status
     17.12.1~ce-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     17.12.0~ce-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     17.09.1~ce-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     17.09.0~ce-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     17.06.2~ce-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     17.06.1~ce-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     17.06.0~ce-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     17.03.2~ce-0~ubuntu-xenial 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     17.03.1~ce-0~ubuntu-xenial 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     17.03.0~ce-0~ubuntu-xenial 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
```

Notice that docker-ce is installed to another version or not installed, but the candidate for installation is from the Docker repository for Ubuntu 16.04. The docker-ce version number might be different.

Finally, install Docker:

```sh
sudo apt-get install -y docker-ce
```

Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it's running:

```sh
sudo systemctl status docker
```

# Setup Artifactory on VM2

```sh
$ echo "deb https://jfrog.bintray.com/artifactory-debs {distribution} {components}" | sudo tee -a /etc/apt/sources.list
```

**Note:** If you are unsure, components should be "main." To determine your distribution, run lsb_release -c

**Example:** ```echo "deb https://jfrog.bintray.com/artifactory-debs xenial main" | sudo tee -a /etc/apt/sources.list```

```sh
$ curl https://bintray.com/user/downloadSubjectPublicKey?username=jfrog | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install jfrog-artifactory-cpp-ce
```

Output:- 

**Note:** 

To configure mysql with artfactory which is highly recomende use the command below, make sure SQL is already installed.

```sh
************ SUCCESS ****************
The Installation of Artifactory has completed successfully.

PLEASE NOTE: It is highly recommended to use Artifactory in conjunction with MySQL. You can easily configure this setup using '/opt/jfrog/artifactory/bin/configure.mysql.sh'.

You can activate artifactory with:
> systemctl start artifactory.service

Then check the status with:
> systemctl status artifactory.service
```

**JVM parameters**

Make sure to modify your JVM parameters by modifying ```JAVA_OPTIONS``` and edit correct ```JAVA_HOME``` path in ```/etc/opt/jfrog/artifactory/default``` as appropriate for your installation.

# Setup kops (kubernetes devOps) in Aws Machine - kops master

**To connect to EC2 instance:**

```sh
ssh -i "<private_key.pem>" ubuntu@<public_ip_address>
```

#### Configure kops master

```sh
$ wget https://github.com/kubernetes/kops/releases/download/1.9.0/kops-linux-amd64
$ chmod +x kops-linux-amd64
$ mv kops-linux-amd64 kops
$ sudo mv kops /usr/local/bin/
$ sudo apt-get install python-pip
$ sudo pip install awscli
```

Create a ```kops``` user through AWS console in IAM (Identity & Access Management) so that kops can deploy kubernetes.

```sh
$ aws configure
```

#### Install kubectl

```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
$ chmod +x kubectl
$ sudo mv kubectl /usr/local/bin/ 
```

# Setup kubenetes on AWS 

Follow this link. - https://github.com/arundhiman86/Kubernetes_cluster_setup/blob/master/README.md

#### On VM1

Note:- With swapon  it is not able to add the node to the kubernetes cluster.

```sh
$ swapoff -a
```

 - To disable permanently, edit the ```vim /etc/fstab``` file and line that contains swap, this will presist after reboot.

# Setup Nagios Core in VM3

**#Important:-**

Switch to root user to perform the installation

#### Prerequisites:

For Ubuntu users, run all steps from this document with root permissions. The following command can be run to switch to a root shell.
sudo -i

**For Ubuntu (15.10 and below) users:**

```sh
$ sudo apt-get install wget build-essential apache2 php5 php5-gd libgd-dev unzip
```

**For Ubuntu (16.04 and above) users:**

```sh
$ sudo apt-get install wget build-essential apache2 php apache2-mod-php7.0 php-gd libgd-dev unzip
$ apt-get update
```

#### Download Nagios Core and Nagios Plugins Tarballs

For all systems, run the following commands in your terminal:

```sh
$ cd /tmp
$ wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.3.4.tar.gz
$ wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
```

This will download Nagios Core, and its required plugins.

Next add the appropriate user and groups for the Nagios process to run:

```sh
$ useradd nagios
$ groupadd nagcmd
$ usermod -a -G nagcmd nagios
$ usermod -a -G nagios,nagcmd www-data
```

#### Nagios Core Installation

Extract the package contents:

```sh
$ tar zxvf nagios-4.3.4.tar.gz
$ tar zxvf nagios-plugins-2.2.1.tar.gz
```

Change to the new Nagios directory and install the packages:

```sh
$ cd nagios-4.3.4
$ ./configure --with-command-group=nagcmd --with-mail=/usr/bin/sendmail --with-httpd-conf=/etc/apache2/
$ make all
$ make install
$ make install-init
$ make install-config
$ make install-commandmode
$ make install-webconf
$ cp -R contrib/eventhandlers/ /usr/local/nagios/libexec/
$ chown -R nagios:nagios /usr/local/nagios/libexec/eventhandlers
$ /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

```sh
$ /usr/bin/install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-available/nagios.conf
$ sudo a2ensite nagios
$ sudo a2enmod rewrite cgi
```
For Ubuntu users using systemd:

```sh
$ sudo cp /etc/init.d/skeleton /etc/init.d/nagios
```

```sh
$ sudo vi /etc/init.d/nagios #(and add the following lines at the end of this file)

DESC="Nagios"
NAME=nagios
DAEMON=/usr/local/nagios/bin/$NAME
DAEMON_ARGS="-d /usr/local/nagios/etc/nagios.cfg"
PIDFILE=/usr/local/nagios/var/$NAME.lock
```

```sh
$ systemctl restart apache2
```

**Before starting nagios service you have to create this file:**

```sh
$ sudo vi /etc/systemd/system/nagios.service #(add the following lines)

[Unit]
Description=Nagios
BindTo=network.target

[Install]
WantedBy=multi-user.target

[Service]
User=nagios
Group=nagios
Type=simple
ExecStart=/usr/local/nagios/bin/nagios /usr/local/nagios/etc/nagios.cfg
```
Then run the following commands:-

```sh
$ sudo systemctl enable /etc/systemd/system/nagios.service
$ sudo systemctl start nagios
$ sudo systemctl restart nagios
```

Create a default user and set a password for Web Interface Access:

```sh
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

#### Nagios Plugin Installation

```sh
$ cd /tmp/nagios-plugins-2.2.1
$ ./configure --with-nagios-user=nagios --with-nagios-group=nagios
$ make
$ make install
```

**Nagios Service Setup**

The following command will register the Nagios daemon to be run upon system startup.

```sh
$ sudo update-rc.d nagios defaults
```

#### Nagios Web Interface

After correctly following the procedures you should nownbe able to access your Nagios Core installation from a web browser.

**You can access the Nagios Core web interface at:**
 
http://\<nagios_server_ip_address\>/nagios
 
**Note:-** 

Log in with the credentials you chose when adding the nagiosadmin user to the htpasswd.users file.

#### Nagios Core Installation Complete!
---
