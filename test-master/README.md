# Tasks:
Implement deployment of 3 tier application, which would run on Ubuntu server 18.04 LTS, would use Nginx, Postgres and Python code in consitent and repeatable way. You would need to automate deployment of:
1. Setup use of 10.0.0.2/18 static IP address, Netmask 255.255.0.0, gateway 10.0.0.1/18.    
2. Install Nginx, configure it to serve static pages and dynamic pages via FCGI (python application)
3. Install PostgreSQL DBMS and create DB, user for DB, set users password.
4. Install simple Python application which would serve "Hello World!" via FCGI.
5. Make sure all your changes are persistent after reboot.

## Network configuration on the target machine:
```
ubuntu@ec2:~$ cat /etc/cloud/cloud-init.yaml 

## As amazon reserved ip adresses 10.0.0.2 for DNS service we will use 100.0.0.2/18 for static IP adress and 100.0.0.1/18 for gateway

# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        ens160:
            addresses:
            - 100.0.0.2/18
            gateway4: 100.0.0.1
            nameservers:
                addresses:
                - 100.0.0.1
    version: 2
ubuntu@ec2:~$ sudo netplan apply
```

## Configuration of the target machine using Ansible playbook:
```
$ ansible --version
ansible 2.0.0.2
```

1. Clone git repository:
```
git clone https://github.com/ybil/testAPP.git
```
2. Generate SSH key-pair and copy public key to the target machine:
```
ssh-keygen -f ~/.ssh/ansible
ssh -i ~/.ssh/ansible.pub <username>@100.0.0.2
```

3. Replace `ansible_user parameter` in `hosts` inventory file with name of the user on the target machine.
4. Run playbook:
```
ybil@ansible:~/testAPP/ansible$ ansible-playbook test_deploy.yaml --ask-sudo-pass -i hosts
SUDO password: 

PLAY ***************************************************************************

TASK [Install base packages]*************************************************
ok: [host1]

TASK [Check that the /usr/local/bin/python3.6 exists]**********************
ok: [host1]

TASK [Update apt-cache]********************************************************
ok: [host1]

TASK [Install packages needed for installing Python]***************************
ok: [host1]

TASK [Extract python 3.6.4 into /tmp]**********************************
ok: [host1]

TASK [Configure python 3.6.4]***************************************************
ok: [host1]

TASK [Make]***********************************************************************
ok: [host1]

TASK [Install Python 3.6.4]*****************************************************
ok: [host1]

TASK [Remove tmp files used for Python 3.6.4 installation]**************************
ok: [host1]

PLAY ***************************************************************************

TASK [Check that database is created] **********************************************
ok: [host1]

TASK [Check that user has access to database] **************************************
ok: [host1]

PLAY ***************************************************************************

TASK [Copy the nginx config file] **********************************************
ok: [host1]

TASK [Create symlink] **********************************************************
ok: [host1]

TASK [Copy default html page] **************************************************
ok: [host1]

TASK [Create /var/www/hello-app dir] *******************************************
ok: [host1]

TASK [Clone git repo] **********************************************************
changed: [host1]

TASK [Add exec permmisions to index.py] ****************************************
changed: [host1]

TASK [Copy and add exec permissions to fcgi init-script] ***********************
ok: [host1]

TASK [Add fcgi to autoboot] ****************************************************
ok: [host1]

TASK [Restart fcgi] ************************************************************
changed: [host1]

TASK [Restart nginx] ***********************************************************
changed: [host1]

PLAY RECAP *********************************************************************
host1                      : ok=18   changed=2    unreachable=0    failed=0   
```


