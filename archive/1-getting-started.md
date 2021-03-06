# Lab 1: Getting Started


In this lab, we will begin to teach you the practical skills to cover the very fundamentals of Ansible, starting with how to install Ansible. We'll then look at node requirements and how to validate your Ansible installation.

In this lab, we will cover the following topics:
 * Installing and configuring Ansible 
 * Understanding your Ansible installation


## Login

SSH to the machine that your instructor has provided you. You login name is `ubuntu` and password will be provided by your instructor.


# Getting the Labs

You can download the labs as follows:

```bash
wget https://elephantscale-public.s3.amazonaws.com/labs/ansible-course.zip
```

You can proceed this lab as follows:

```bash
cd ~/ansible-course/Lab_1
```


## Confirm passwordless ssh


(This step should not be necessary on the lab machines, but will be required if you have a vanilla ubuntu install)

You should be able to `ssh` to localhost, as follows:

```bash
ssh localhost  # you may have to type in yes when asking for a key
exit
```

If this does not work, you need to configure the key for passwordless ssh to localhost

```bash
ssh-keygen -t rsa # Press enter for each line 
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod og-wx ~/.ssh/authorized_keys 
```



## Installing Ansible

Confirm that you have ansible installed as follows

```bash
ansible --version
```

And you should get the following response:

```console

ansible 2.9.24
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/ubuntu/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12 (default, Mar  1 2021, 11:38:31) [GCC 5.4.0 20160609]


```

If you do not have installed already,

```bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt-get install ansible
```


## Configuring Hosts

We are going to be creating some hosts in `/etc/hosts` file to test with ansible in our labs

First, edit (as sudo) the `/etc/hosts` file with a commmand line editor such as `vim` or `nano`.

```bash
sudo vi /etc/hosts/
```

Add the following lines to the file `/etc/hosts`

```text
127.0.0.1 web1.example.com
127.0.0.1 web2.example.com
127.0.0.1 ap1.example.com
127.0.0.1 ap2.example.com
127.0.0.1 frt01.example.com
127.0.0.1 frt02.example.com
```

Now, confirm this works by attempting to perform an ssh to each of the four machiens

```bash
ssh web1.example.com   # You may have to say "yes" when asked
exit
ssh web2.example.com   # You may have to say "yes" when asked
exit
ssh ap1.example.com   # You may have to say "yes" when asked
exit
ssh ap2.example.com   # You may have to say "yes" when asked
exit
ssh frt01.example.com   # You may have to say "yes" when asked
exit
ssh frt02.example.com   # You may have to say "yes" when asked
exit
```

## Configure ansible hosts files

We need to add our new hosts to our `/etc/ansible/hosts` file

```bash
sudo vim /etc/ansible/hosts
```

```text
[webservers]
web1.example.com
web2.example.com

[apservers]
ap1.example.com
ap2.example.com
```



## Perform a ping command

```bash
ansible webservers -m ping
```

Ignore any deprecation warnings about python 2 vs 3

You should get the following

```console
ap1.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
ap2.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

```

## Verifying the Ansible installation
In this section, you will learn how you can verify your Ansible installation with simple ad hoc commands.

As we discussed in the previous section, we must also define an inventory for Ansible to run against. Another simple example is shown here:

These should be added to your `/etc/ansible/hosts`

```text
[frontends] 
frt01.example.com 
frt02.example.com
```
 Following are three simple examples that demonstrate ad hoc commands---they are also valuable for verifying both the installation of Ansible on your control machine and the configuration of your target hosts, and they will return an error if there is an issue with any part of the configuration:


### Ping Hosts
Ping hosts: You can perform an Ansible "ping" on your inventory hosts using the following command:
```bash
ansible frontends  -m ping
```

### Display Gathered Facts

```bash
ansible frontends  -m setup | less
```

### Filtered Gathered Facts

```bash
ansible frontends  -m setup -a "filter=ansible_distribution*"
```

Ad hoc commands are incredibly powerful, both for verifying your Ansible installation and for learning Ansible and how to work with modules as you don't need to write a whole playbook---you can just run a module with an ad hoc
command and learn how it responds. Here are some more ad hoc examples for you to consider:

# Copy a file
Copy a file from the Ansible control host to all hosts in the [frontends] group with the following command:
```bash
ansible frontends -m copy -a "src=/etc/hosts dest=/home/ubuntu/hosts"
```

## Create A new directory
Create a new directory on all hosts in the [frontends] inventory group, and create it with specific ownership and permissions:

```bash
ansible frontends -m file -a "dest=/home/ubuntu/new mode=777 owner=ubuntu group=ubuntu state=directory"
```

## Delete a Directory
Delete a specific directory from all hosts in the [frontends] group with the following command:

```bash
ansible frontends -m file -a "dest=/home/ubuntu/new state=absent"
```

### Install apache2 package
Install the [apache2] package with [apt] if it is not already present---if it is present, do not update it. Again, this applies to all hosts in the [frontends] inventory group:

```bash
ansible frontends -m apt -a "name=apache2 state=present"
```

## Get Latest package
The following command is similar to the previous one, except that changing [state=present] to [state=latest] causes Ansible to install the (latest version of the) package if it is not present, and update it to the latest version if it is present:
```bash
ansible frontends -m apt -a "name=apache2 state=latest"
```

## Display Facts
Display all facts about all the hosts in your inventory (warning---this will produce a lot of JSON!):

```bash
ansible all -m setup
```


Now that you have learned more about verifying your Ansible installation
and about how to run ad hoc commands, let\'s proceed to look in a bit
more detail at the requirements of the nodes that are to be managed by
Ansible.



Running from source versus pre-built RPMs
=========================================


Let\'s get started by checking out the very latest version of the source code from GitHub:

1.  You must clone the sources from the [git] repository first,
    and then change to the directory containing the checked-out code:

```
$ git clone https://github.com/ansible/ansible.git --recursive
$ cd ./ansible
```



2.  Before you can proceed with any development work, or indeed to run
    Ansible from the source code, you must set up your shell
    environment. Several scripts are provided for just that purpose,
    each being suitable for different shell environments. For example,
    if you are running the venerable Bash shell, you would set up your
    environment with the following command:

```
$ source ./hacking/env-setup
```




3.  Once you have set up your environment, you must install the
    [pip] Python package manager, and then use this to install all
    of the required Python packages (note: you can skip the first
    command if you already have [pip] on your system):

```
$ sudo pip3 install -r ./requirements.txt
```



Note that, when you have run the [env-setup] script, you\'ll be
running from your source code checkout, and the default inventory file
will be [/etc/ansible/hosts]. You can optionally specify an
inventory file other than [/etc/ansible/hosts].

4.  When you run the [env-setup] script, Ansible runs from the
    source code checkout, and the default inventory file is
    [/etc/ansible/hosts]; however, you can optionally specify an
    inventory file wherever you want on your machine. The following command provides an
    example of how you might do this, but obviously, your filename and
    contents are almost certainly going to vary:

```
$ echo "ap1.example.com" > ~/my_ansible_inventory
$ export ANSIBLE_INVENTORY=~/my_ansible_inventory
```


Once you have completed these steps, you can run Ansible exactly as we
have discussed throughout this lab, with the exception that you must
specify the absolute path to it. For example, if you set up your
inventory as in the preceding code and clone the Ansible source into
your home directory, you could run the ad hoc [ping] command that
we are now familiar with, as follows:

```
$ ~/ansible/bin/ansible all -m ping

ap1.example.com | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```



Of course, the Ansible source tree is constantly changing and it is
unlikely you would just want to stick with the copy you cloned. When the
time comes to update it, you don\'t need to clone a new copy; you can
simply update your existing working copy using the following commands
(again, assuming that you initially cloned the source tree into your
home directory):

```
$ git pull --rebase
$ git submodule update --init --recursive
```

That concludes our introduction to setting up both your Ansible control
machine and managed nodes. It is hoped that the knowledge you have
gained in this lab will help you to get your own Ansible
installation up and running and set the groundwork for the rest of this
course.


Summary
=======

Ansible is a powerful and versatile yet simple automation tool, of which
the key benefits are its agentless architecture and its simple
installation process. Ansible was designed to get you from zero to
automation rapidly and with minimal effort, and we have demonstrated the
simplicity with which you can get up and running with Ansible in this
lab.

In the next lab, we will learn Ansible language fundamentals to
enable you to write your first playbooks and to help you to create
templated configurations and start to build up complex automation
workflows.


Questions
=========

1.  On which operating systems can you install Ansible? (Multiple
    correct answers)

A\) Ubuntu

B\) Fedora

C\) Windows 2019 server

D\) HP-UX

E\) Mainframe

2.  Which protocol does Ansible use to connect the remote machine for
    running tasks?

A\) HTTP

B\) HTTPS

C\) SSH

D\) TCP

E\) UDP

3.  To execute a specific module in the Ansible ad hoc command line, you
    need to use the [-m] option.

A\) True

B\) False

