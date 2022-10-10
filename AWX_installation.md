Hello everyone !

In this topic, I am going to talk about how to install AWX with docker. I am going to install 17.1.0 version of AWX. If you want to install the lastest version, then you need to install AWX operator which you will need to kubernetes cluster.

# Pre-requirements:
- 4vcpu,8GB memory,15GB (for awx)+20GB (for docker) disks rhel8/centos8 linux server
- python3, ansible, docker, docker-compose --> I added detailed information about the version of the packages below.
- python3 packages 
  - ansible (4.10.0)
  - ansible-core (2.11.12)
  - docker (5.0.0)
  - setuptools (59.6.0)
  - docker-compose (1.29.2)


# 1) Installing ansible
You need to install ansible-core package of RHEL8.
```sh
[root@linuxserver ~]#yum install ansible-core -y
```
```sh
[root@linuxserver ~]# ansible --version
ansible [core 2.12.2]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.8/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /bin/ansible
  python version = 3.8.12
  jinja version = 2.10.3
  libyaml = True
```

# 2) Installing docker and docker-compose
You need to create /var/lib/docker directory and mount the filesytem. Please be careful not to install under root.
```sh
I created a new volume group named vg_data and a new logical volume named lv_docker.
[root@linuxserver ~]# vgs
  VG        #PV #LV #SN Attr   VSize   VFree  
  vg_data     1   2   0 wz--n- <80.00g <15.00g

[root@linuxserver ~]# lvs
  LV        VG        Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert                                                  
  lv_docker vg_data   -wi-ao---- 20.00g  

[root@linuxserver ~]# mkfs.xfs /dev/mapper/vg_data-lv_docker
[root@linuxserver ~]#vi /etc/fstab
/dev/mapper/vg_data-lv_docker   /var/lib/docker   xfs     defaults        0 0
[root@linuxserver ~]#mkdir /var/lib/docker
[root@linuxserver ~]#mount -a
[root@linuxserver ~]#vi /etc/yum.repos.d/docker.repo
	[docker]
	name= Docker repo for rhel8
	baseurl= <dokcer package path>
	enabled= 1
	gpg_check= 0
[root@linuxserver ~]# yum install docker-ce docker-ce-cli --nobest --allowerasing -y
[root@linuxserver ~]#systemctl enable docker
[root@linuxserver ~]#systemctl status docker
[root@linuxserver ~]#systemctl start docker

To install docker-compose, you need to install the package from https://github.com/docker/compose/releases/ and copy that under /usr/bin.
[root@linuxserver ~]#chmod +x /usr/bin/docker-compose
[root@linuxserver ~]#chmod 755 /usr/bin/docker-compose
[root@linuxserver ~]#chown root:root /usr/bin/docker-compose

[root@linuxserver ~]# docker version
Client: Docker Engine - Community
 Version:           20.10.18
 API version:       1.41
 Go version:        go1.18.6
 Git commit:        b40c2f6
 Built:             Thu Sep  8 23:11:56 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.18
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.18.6
  Git commit:       e42327a
  Built:            Thu Sep  8 23:10:04 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.8
  GitCommit:        9cd3357b7fd7218e4aec3eae239db1f68a5a6ec6
 runc:
  Version:          1.1.4
  GitCommit:        v1.1.4-0-g5fd4c4d
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
  
  
[root@linuxserver ~]# docker-compose version
Docker Compose version v2.11.0
```

# 3) Installing required python3 packages

```sh
[root@linuxserver ~]#pip3 install ansible
[root@linuxserver ~]#pip3 install ansible-core
[root@linuxserver ~]#pip3 install docker==5.0.0
[root@linuxserver ~]#pip3 install requests==2.22.0
[root@linuxserver ~]#pip3 install setuptools-rust
[root@linuxserver ~]#pip3 install docker-compose==1.29.2

[root@linuxserver ~]#yum install gcc
[root@linuxserver ~]#yum install gcc-c++

```

# 4) Create AWX directory
I created a new lv named lv_awx and awx directory under /var/lib.
```sh
[root@linuxserver ~]# lvs
  LV        VG        Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert                                                  
  lv_awx vg_data   -wi-ao---- 15.00g  

[root@linuxserver ~]# mkfs.xfs /dev/mapper/vg_data-lv_awx
[root@linuxserver ~]#vi /etc/fstab
/dev/mapper/vg_data-lv_awx   /var/lib/awx   xfs     defaults        0 0
[root@linuxserver ~]#mkdir /var/lib/awx
[root@linuxserver ~]#mount -a
```
Also you need to create the following directory under /var/lib/awx
```sh
[root@linuxserver ~]#mkdir /var/lib/awx/awxcompose
[root@linuxserver ~]#mkdir /var/lib/awx/pgdocker
[root@linuxserver ~]#mkdir /var/lib/awx/projects
```

# 5) Editing AWX inventory

First of all, you need to clone awx git repository to local
```sh
[root@linuxserver ~]#git clone -b "17.1.0" https://github.com/ansible/awx.git
[root@linuxserver ~]#cd awx-17.1.0/installer
[root@linuxserver ~]#vi inventory
```
You need to edit the following parameters
```sh
dockerhub_base=ansible

awx_task_hostname=awx
awx_web_hostname=awxweb
postgres_data_dir="/var/lib/awx/pgdocker"
host_port=80
host_port_ssl=443
docker_compose_dir="/var/lib/awx/awxcompose"

pg_username=awx
pg_password=awxpassword
pg_database=awx
pg_port=5432

admin_user=admin
admin_password=adminpassword12

create_preload_data=True

secret_key=k9ON4KqaPFPC7U783452  #you can create the secret with this command "openssl rand -base64 30"

awx_official=false

project_data_dir=/var/lib/awx/projects
```

# 6) Installing community.docker
To use the docker modules, it is necessary to install community.docker with ansible-galaxy
```sh
[root@linuxserver ~]#ansible-galaxy collection install community.docker
```

# 7) Editing compose.yml
You need to add ansible_python_interpreter variable to the playbook.
```sh
[root@linuxserver ~]#vi awx-17.1.0/installer/roles/local_docker/tasks/compose.yml

    - name: Start the containers
      docker_compose:
        project_src: "{{ docker_compose_dir }}"
        restarted: "{{ awx_compose_config is changed or awx_secret_key is changed }}"
      register: awx_compose_start
      vars:
        ansible_python_interpreter: /bin/python3
```

# 8) Running installation playbook
This is final task. It will be ready in 5 minutes :)
```sh
[root@linuxserver installer]#ansible-playbook -i inventory install.yml -vvv
```
```sh
[root@linuxserver installer]## docker ps
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS          PORTS                                   NAMES
58a930767548   ansible/awx:17.1.0   "/usr/bin/tini -- /u…"   45 seconds ago   Up 43 seconds   8052/tcp                                awx_task
85d764dd0c6b   ansible/awx:17.1.0   "/usr/bin/tini -- /b…"   46 seconds ago   Up 43 seconds   0.0.0.0:80->8052/tcp, :::80->8052/tcp   awx_web
37932e7a3db8   postgres:12          "docker-entrypoint.s…"   47 seconds ago   Up 42 seconds   5432/tcp                                awx_postgres
3a3aa25e388f   redis                "docker-entrypoint.s…"   47 seconds ago   Up 42 seconds   6379/tcp                                awx_redis
```
<img width="745" alt="Capture" src="https://user-images.githubusercontent.com/28459280/194848366-c0906923-102b-4d52-b053-5a499be952c8.PNG">


