# iris_lab

    1  yum update -y
    2  wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
    3  cat /etc/yum.repos.d/jenkins.repo
    5  rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
    8  amazon-linux-extras install java-openjdk11 -y
    9  java --version
   10  yum install jenkins -y
   11  systemctl enable jenkins
   12  systemctl start jenkins
   13  systemctl status jenkins


Clean up
========
   31  systemctl stop jenkins
   32  yum remove jenkins
   33  rm -rf /var/lib/jenkins

   36  rpm -qa | grep java
   37  yum remove java-11-openjdk
   40  yum remove java-11-openjdk*
   41  java --version
   42  rpm -qa | grep java
   43  yum remove java-17* javapackages*

Reinstall
=========

   44  yum install java-1.8.0-openjdk
   46  java -version
   48  yum install https://archives.jenkins-ci.org/redhat-stable/jenkins-2.346.3-1.1.noarch.rpm -y
   49  systemctl enable jenkins
   50  systemctl start jenkins
   51  systemctl status jenkins
   52  yum install maven -y
   53  yum install git -y


kubeadm join k8s-controller:6443 --token zt79ic.lg6daovi4cuyymso --discovery-token-ca-cert-hash sha256:2a1a72f9a3e3f3d4d0ca634dcf29b7eba55d9bcee42efe0b5ce95eb306a68f94




Setup Jenkins Node:
-------------------

Create a ec2 instance and login

    0 sudo -i
    1  vim /etc/ssh/sshd_config
		find Pubkekey and uncomment the line
		find Password and replace no with yes

    2  systemctl restart sshd
    3  useradd jenkins
    4  passwd jenkins
    5  yum install -y git maven
    6  yum install java-1.8.0-openjdk -y
    7  which git
    8  mvn --version
    9  java --version
   10  rpm -qa | grep java
   11  yum remove java-17* -y
   12  java -version
   13  su - jenkins

Setup Master
-------------
login and become root (
   su - jenkins -s /bin/bash
    4  ssh-keygen
    5  ssh-copy-id 35.153.66.65
    6  ssh 35.153.66.65
    7  cat .ssh/id_rsa
    8  ssh 35.153.66.65



login to ec2 instance (worker node in our case)

yum install 





ssh <new ip> of your node
exit
change the IP in ansible hosts file
	vim /etc/ansible/hosts

vim roles/apache/defaults/main.yml
	add apache_user: apache and apache_group: apache

vim roles/apache/vars/main.yml
	add apache_port: 8080

cd roles/apache/templates
scp 54.86.123.251:/etc/httpd/conf/httpd.conf .
mv httpd.conf httpd.conf.j2
vim httpd.conf.j2
	update Listen {{ apache_port }}
		 user {{ apache_user }}
		 group {{ apache_group }}
	save and exit

cd ../../..

vim roles/apache/tasks/main.yml

- name: reconfigure apache port to 8080
  template:
    src: httpd.conf.j2
    dest: /etc/httpd/conf/httpd.conf

- name: restart apache
  service:
    name: httpd
    state: restarted


vim roles/apache/tasks/main.yml

- name: reconfigure apache port to 8080
  template:
    src: httpd.conf.j2
    dest: /etc/httpd/conf/httpd.conf
  notify:
    - restart apache

move the restart apache task to handlers/main.yml

vim roles/apache/handlers/main.yml

- name: restart apache
  service:
    name: httpd
    state: restarted




vim roles/apache/tasks/main.yml

- name: install httpd
  yum:
    name: httpd
    state: present
  when: ansible_os_family == "RedHat"

- name: create index.html
  copy:
    src: index.html
    dest: /var/www/html/index.html
  when: ansible_os_family == "RedHat"


create an ubuntu instance

login as root

vim /etc/ssh/sshd_config

	PermitRootLogin yes
	PubkeyAuthentication yes
	PasswordAuthentication yes
save and exit
systemctl restart sshd

ssh-copy-id 54.165.44.193

vim /etc/ansible/hosts
	add the ip of ubuntu instance
save and exit

ansible-playbook install_apache.yml -v
ansible webservers -m setup | grep -i os_family


cd roles/apache/tasks/
cp main.yml redhat.yml
vim redhat.yml (remove all when conditions)


cp main.yml ubuntu.yml
vim ubuntu.yml

- name: install httpd
  apt:
    name: apache2
    state: present

- name: start httpd service
  service:
    name: apache2
    state: started

 vim main.yml
- include: redhat.yml
  when: ansible_os_family == "RedHat"

- include: ubuntu.yml
  when: ansible_os_family == "Debian"


Integrating Ansible with Jenkins
================================

login to your node as root and ensure jenkins user is present in your target
	useradd jenkins
	passwd jenkins

add jenkins user as a sudo user
	usermod -aG wheel jenkins

Login to your Jenkins/Ansible master as root

	systemctl restart

	su - jenkins -s /bin/bash
	ssh-copy-id <IP of your node>

On Jenkins Dashboard
	Install ansible plugin
	create a freestyle job and use below repo to configure your playbook

https://github.com/DevOpsHubIN/iris_lab.git

# Docker Lab Day 1 #

Install Docker

  221  yum update
  222  yum install docker
  223  systemctl status docker
  224  systemctl start docker
  225  systemctl status docker
  226  docker images

List images and Run hello-world, ubuntu container

  228  docker images
  229  docker run hello-world
  230  docker images
  231  docker ps
  232  docker run hello-world
  233  docker ps
  234  docker ps -a
  235  docker run ubuntu
  236  docker ps
  237  docker images
  238  docker ps -a

Create and connect to a container

  241  cat /etc/os-release
  242  uname -r
  243  docker run -ti ubuntu /bin/bash
		  cat /etc/os-release
  		  uname -r
		  exit
  244  docker ps
  245  docker ps -a
  246  docker run -tid ubuntu /bin/sh
		  exit
  247  docker ps
  248  docker ps -a

Creating a image from a running container

  255  docker run -ti ubuntu bash
			apt update
			apt upgrade -y
			echo "This image is updated on `date`" > update_time
			exit

  256  docker ps -a | grep <your container ID>
  257  docker commit <your container ID> ubuntu_iris:updated
  258  docker images
  259  docker run -ti ubuntu_iris:updated bash
		exit
  260  docker ps
  261  docker ps -a | grep <your container ID>


creating a name for a container

docker run --name iris_ubuntu_v3 -tid ubuntu_iris:updated bash

Managing container state

  285  docker ps -a
  286  docker start iris_ubuntu_v2
  287  docker ps
  288  docker pause iris_ubuntu_v3
  289  docker ps
  290  docker start iris_ubuntu_v3
  291  docker unpause iris_ubuntu_v3
  292  docker kill sleepy_haslett
  293  docker ps
  294  docker ps -a
  295  docker rm 2d8351f8de95
  296  docker container prune
  297  docker ps

Looking at the logs

  302  docker run --name devops -d ubuntu bash -c "lose /etc/password"
  303  docker log devops
  304  docker logs devops


Limiting resources on docker container

  307  docker stop devops
  308  docker rm devops
  309  docker run --name devops -m 512m -d ubuntu bash
  310  docker describe devops
  311  docker inspect devops
  312  docker stop devops
  313  docker rm devops
  314  docker run --name devops -m 512m --cpus=1 -d ubuntu bash



