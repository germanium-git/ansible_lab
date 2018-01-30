## Install testing environment

Build docker images, run containers end test connectivity between the ansible server and the targets,

### Build a docker image of the ansible target
- cd target
- docker build -t ansible-target:16.04 .

#### Create target containers
- docker run --name target1 -it -d ansible-target:16.04
- docker run --name target2 -it -d ansible-target:16.04
- docker run --name target3 -it -d ansible-target:16.04

#### Identify the IP addresses assigned to target containers
- docker inspect target1 | grep IPAddress

### Build a docker image of the ansible server
- cd server
- docker build -t ansible-server:16.04 .
 
#### Create a container of the ansible server
- docker run --name ansible -it -d ansible-server:16.04
- docker inspect ansible | grep IPAddress

#### Configure ansible
- ssh root@172.17.0.x
password: root

#### Create the inventory file inventory.txt
target1 ansible_host=172.17.0.2 ansible_ssh_pass=root
target2 ansible_host=172.17.0.3 ansible_ssh_pass=root

#### Test the SSH connection to the target servers
Connect to the ansible server
- ssh root@172.17.0.2

### Test if ansible works

- It can be tested by using ad-hoc commands:
```sh
root@909a6a674fc8:~# ansible target* -m ping -i inventory.txt
target2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
target3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
target1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

- Or by executing the ansible playbook:
```sh
root@b957560d9746:~# ansible-playbook playbook_ping.yml -i inventory.txt

PLAY [Deploy a web application] ******************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************
ok: [db_and_web_server1]

TASK [Try Ping] **********************************************************************************************************************************************************************************
ok: [db_and_web_server1]

PLAY RECAP ***************************************************************************************************************************************************************************************
db_and_web_server1         : ok=2    changed=0    unreachable=0    failed=0
```