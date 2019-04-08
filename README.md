# CDP_LLDP_Auto_Port_Description
This Ansible script will take a list of devices and get CDP/LLDP neighborship on them and change port description using neighbor information. This will override manual port descriptions configured. 
 - Note: As of now this scripts supports IOS,NXOS,EOS devices. Infact this may not work on all the firmware versions in these models e.g NXOS lower than 7.X is not supported. 
# Requirements:
1. Ansible installed. This should already set path to major ansible scripts e.g ansible-playbook ansible-connection  ansible-config 
   - If these binaries/scripts are not in your path then either you need to put them in path or create symbolic links. In my setup I am using symbolic links as below. Change your path accordingly.
      - `ln -s /opt/python3.6/bin/ansible-config ansible-config` 
      - `ln -s /opt/python3.6/bin/ansible-connection ansible-connection` 
      - `ln -s /opt/python3.6/bin/ansible-playbook ansible-playbook`



# Major components of this project
1. List of network hosts (Filename hosts )
2. An Ansible playbook for gathering LLDP/CDP information and changing port description (Filename: NeighborDescription.yml)
3. Jump server and encryptions (all_backup.yml file folder group_vars)
   - Using jump server as Proxy 
   - putting credentials in file and encrypting them
   - putting vault password
4. Ansible configuration file to set few ansible parameters as per our requirement.  (Filename: ansible.cfg)



# Detailed explanation of components
## 1. Hosts (also called inventory file)

Our current project supports Cisco ios,Cisco nxos (v7.x...) and Arista eos hence we need inventory file which has list of IP addresses/hostnames categorized as below
```
[ios]
a.b.c.d
w.x.y.z
[nxos]
a.b.c.d
w.x.y.z
[eos]
a.b.c.d
w.x.y.z
```


This script doesn't cover how to gather this IP list. There need to be separate script/procedure to have list of devices categorized as shown above. 

## 2. Ansible playbook ( NeighborDescription.yml)
Note: For detailed comments check NeighborDescription.yml file.
Ansible performs certain tasks on remote devices. These tasks are combined to achieve something and it is called a play. You can have multiple plays in one playbook. 

Ansible Playbook
- Play for IOS
  - Task to gather current LLDP information
  - Task to gather current port description
  - Task to change port description if it is not same as LLDP information.
- Play for NXOS
  - Task to gather current CDP/LLDP information
  - Task to gather current port description
  - Task to change port description if it is not same as LLDP information.
- Play for EOS
  - Task to gather current LLDP information
  - Task to gather current port description
  - Task to change port description if it is not same as LLDP information.



## 3. Using jump server as Proxy
We are developing script on a server which have no access to network devices directly hence our server will have to jump via proxy/bastion/jump server in order to reach network devices. For this we need to do following
 - Public key based ssh login to proxy/bastion/jump server.
   https://www.cyberciti.biz/tips/ssh-public-key-based-authentication-how-to.html 
 - using special ssh commands in /group_vars/all_backup.yml file and also we need to change credentials for network devices. 
   ```
   ansible_ssh_common_args: '-o ProxyCommand="ssh -W %h:%p -q userName@jumpServer"'
   ansible_ssh_extra_args: "-o UserKnownHostsFile=/dev/null -o ControlMaster=no"
   ansible_user: cisco
   ansible_ssh_pass: cisco
   ```
 - Moving /group_vars/all_backup.yml to /group_vars/all.yml
    Use shell command to move file
    ```
    mv /group_vars/all_backup.yml /group_vars/all.yml
    ```
 - 


## 4. Ansible config parameters.
Some config parameters. Refer ansible documentation for details.

## 5. (Optional but most important). Encrypting file with passwords
 As we have file group_vars/all.yml which has login credentials in plaintext format, our goal is to encrypt it. This will also allow you to push code to github without worrying about plaintext credentials. 
 How to do it? https://docs.ansible.com/ansible/latest/user_guide/vault.html
 - Use following command to encrypt a file which already have credentials. This will ask you for vault key to encrypt this file with. Use any key you like. Its like new vault password.  
      `ansible-vault encrypt group_vars/all.yml`
 - Once you have vault password, you need to tell Ansible so that Ansible scripts know how to decrypt vault password. For this you need to create a file .vault_pass and put your vault password in it.
 - Modify ansible.cfg so that Ansible knows about .vault_pass file. Following line in ansible.cfg file s for this purpose 
      `vault_password_file=.vault_pass`

