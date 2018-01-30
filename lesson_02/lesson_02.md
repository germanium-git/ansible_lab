### Simple-webapp 
Follow the instructions from 
https://github.com/mmumshad/simple-webapp


#### Create the inventory file inventory.txt
target1 ansible_host=172.17.0.x ansible_ssh_pass=root
target2 ansible_host=172.17.0.y ansible_ssh_pass=root

#### Spin up the target servers 
Don't forget to expose the web application to external network
docker run --name target1 -it -p 5000:5000 -d ansible-target:16.04
docker run --name target2 -it -p 5000:5000 -d ansible-target:16.04

All three servers should look like this output
```sh
user@server:~⟫ docker ps
CONTAINER ID        IMAGE                  COMMAND               CREATED              STATUS              PORTS                            NAMES
cee8718a0d84        ansible-target:16.04   "/usr/sbin/sshd -D"   About a minute ago   Up About a minute   22/tcp, 0.0.0.0:5001->5000/tcp   target2
38af5ebf4176        ansible-target:16.04   "/usr/sbin/sshd -D"   About a minute ago   Up About a minute   22/tcp, 0.0.0.0:5000->5000/tcp   target1
b957560d9746        ansible-server:16.04   "/usr/sbin/sshd -D"   4 days ago           Up 4 days           22/tcp                           ansible
root@909a6a674fc8:~# ansible target* -m ping -i inventory.txt
```

#### Test the SSH connection to the target servers from ansible server
Connect to the ansible server
- ssh root@172.17.0.2

### Test if ansible works
- ansible-playbook playbook_web.yml -i inventory.txt

The expected output should look like this:
```sh
root@b957560d9746:~# ansible-playbook playbook_ping.yml -i inventory.txt

PLAY [Deploy a web application] ******************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************
ok: [db_and_web_server1]

TASK [Try Ping] **********************************************************************************************************************************************************************************
ok: [db_and_web_server1]

PLAY RECAP ***************************************************************************************************************************************************************************************
db_and_web_server1         : ok=2    changed=0    unreachable=0    failed=0

root@b957560d9746:~#
root@b957560d9746:~#
root@b957560d9746:~#
root@b957560d9746:~# ansible-playbook playbook_web.yml -i inventory.txt

PLAY [Deploy a web application] ******************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************
ok: [db_and_web_server1]
ok: [db_and_web_server2]

TASK [Install all required dependencies] *********************************************************************************************************************************************************
changed: [db_and_web_server1] => (item=[u'python', u'python-setuptools', u'python-dev', u'build-essential', u'python-pip', u'python-mysqldb'])
changed: [db_and_web_server2] => (item=[u'python', u'python-setuptools', u'python-dev', u'build-essential', u'python-pip', u'python-mysqldb'])

TASK [Install MySQL database] ********************************************************************************************************************************************************************
changed: [db_and_web_server2] => (item=[u'mysql-server', u'mysql-client'])
changed: [db_and_web_server1] => (item=[u'mysql-server', u'mysql-client'])

TASK [Start MySQL Service] ***********************************************************************************************************************************************************************
 [WARNING]: Consider using service module rather than running service

changed: [db_and_web_server1]
changed: [db_and_web_server2]

TASK [Create Application Database] ***************************************************************************************************************************************************************
changed: [db_and_web_server1]
changed: [db_and_web_server2]

TASK [Copy SQL commands] *************************************************************************************************************************************************************************
changed: [db_and_web_server1]
changed: [db_and_web_server2]

TASK [Insert some test data] *********************************************************************************************************************************************************************
changed: [db_and_web_server1]
changed: [db_and_web_server2]

TASK [Create Database user] **********************************************************************************************************************************************************************
changed: [db_and_web_server2]
changed: [db_and_web_server1]

TASK [Install Python Flask dependency] ***********************************************************************************************************************************************************
changed: [db_and_web_server2] => (item=flask)
changed: [db_and_web_server1] => (item=flask)
changed: [db_and_web_server2] => (item=flask-mysql)
changed: [db_and_web_server1] => (item=flask-mysql)

TASK [Copy source code] **************************************************************************************************************************************************************************
changed: [db_and_web_server1]
changed: [db_and_web_server2]

TASK [Start Web Server] **************************************************************************************************************************************************************************
changed: [db_and_web_server2]
changed: [db_and_web_server1]

PLAY RECAP ***************************************************************************************************************************************************************************************
db_and_web_server1         : ok=11   changed=10   unreachable=0    failed=0
db_and_web_server2         : ok=11   changed=10   unreachable=0    failed=0
```

### Test web app
Open a browser and go to URL

    http://<IP>:5000                            => Welcome
    http://<IP>:5000/how%20are%20you            => I am good, how about you?
    http://<IP>:5000/read%20from%20database     => JOHN


### Remove all containers
docker stop $(docker ps -aq) # Stop all containers
docker rm $(docker ps -aq) # Remove all containers
